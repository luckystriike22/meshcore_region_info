# Regions Guide: Managing Message Scopes in MeshCore

## Flowchart: Region-based Packet Forwarding

```mermaid
flowchart TD
  A[Packet Received by Repeater] --> B{Packet has transport codes?}
  B -- Yes --> C[Find matching region in region map]
  C -- Match found --> D{Region allows flooding? (allowf)}
  D -- Yes --> E[Forward packet]
  D -- No --> F[Drop packet]
  C -- No match --> G{Wildcard region allows flooding?}
  G -- Yes --> E
  G -- No --> F
  B -- No --> H{Wildcard region allows flooding?}
  H -- Yes --> E
  H -- No --> F
    
  style E fill:#b2f2b2,stroke:#333,stroke-width:2px
  style F fill:#f2b2b2,stroke:#333,stroke-width:2px
```

## Overview

Regions are a powerful feature in MeshCore that enables network administrators to organize their mesh networks into logical geographic or functional zones. They provide control over message flooding and repeating behavior, allowing fine-grained control over which parts of the network forward (repeat) messages to their neighbors.

This guide explains:
- What regions are and how they work
- The concept of region scopes and transport codes
- The `allowf` (allow flood) flag and its behavior
- How repeaters decide whether to forward messages
- Practical examples and best practices

---

## Regions: Fundamental Concepts

### What is a Region?

A **region** is a named organizational unit that represents a logical area or group in your mesh network. Regions are typically named after geographic locations or functional areas, such as:
- `#Europe`, `#NorthAmerica`, `#Asia`
- `#London`, `#Paris`, `#NewYork`
- `#Building1`, `#Floor2`, `#OfficeA`
- `#Warehouse`, `#StorageFacility`

### Region Structure

Regions are organized in a **hierarchical tree structure** with parent-child relationships:

```
*  (wildcard - root)
├── #Europe
│   ├── #UK
│   │   ├── #London
│   │   └── #Manchester
│   └── #France
│       ├── #Paris
│       └── #Lyon
└── #NorthAmerica
    ├── #USA
    │   ├── #NewYork
    │   └── #California
    └── #Canada
        ├── #Ontario
        └── #Quebec
```

### Key Region Properties

Each region has the following properties:

| Property | Description |
|----------|-------------|
| **ID** | A unique 16-bit identifier (auto-assigned) |
| **Name** | A human-readable name (max 30 characters) |
| **Parent ID** | The ID of the parent region in the hierarchy |
| **Flags** | Bit flags controlling behavior (e.g., ALLOW/DENY flooding) |

### The Wildcard Region (`*`)

The wildcard region is the **root of the hierarchical tree**. It represents:
- A catch-all default zone
- The fallback region for packets without explicit transport codes
- The top-level parent to which other regions attach

By default, the wildcard region **allows flooding** (the `F` flag is set).

---

## Region Scopes and Transport Codes

### What is a Region Scope?

A **region scope** defines which region(s) a message is intended for. It's the mechanism by which senders indicate to repeaters: _"This message is for the Europe region" or "This message is for the London subregion."_

### Understanding Transport Codes

Transport codes are the technical mechanism used to encode region scope information in packets. Each packet can carry **two 16-bit transport codes**:

```
uint16_t transport_codes[2];
```

- **transport_codes[0]**: Encodes the **sender's scope** (which region the sender is in)
- **transport_codes[1]**: Encodes the **intended reply/response scope** (where replies should go)

Each transport code is a **cryptographic hash** computed from:
1. The region's unique ID
2. The region's name (with automatic "#" prefix)
3. The packet data

### How Transport Codes Work

When a sender wants to send a message scoped to a specific region:

1. The sender selects a region (e.g., `#Europe`)
2. The sender generates a transport code by hashing the region ID/name with the packet data
3. The sender includes this transport code in the packet's `transport_codes[0]` field
4. Repeaters in the mesh receive the packet
5. Each repeater checks if this transport code matches any of its **configured regions**
6. If there's a match AND the region allows flooding (`allowf` is set), the repeater forwards the packet
7. If no match is found or flooding is denied, the repeater drops the packet

### Transport Code Matching

A repeater matches a transport code by:

1. Checking each region in its region map
2. Computing what the transport code **should be** for each region
3. Comparing it against the packet's `transport_codes[0]`
4. If any region matches, that region becomes the packet's **region scope**

**Example:**
```
Packet arrives with transport_codes[0] = 0x1A2B
Repeater's region map contains:
  - #Europe (ID: 2) -> computes to 0x1A2B  ✓ MATCH!
  - #Asia (ID: 3) -> computes to 0x3C4D
  - #Africa (ID: 4) -> computes to 0x5E6F
Result: Packet is scoped to #Europe
```

---

## The `allowf` Flag: Allow Flooding

### What is `allowf`?

The **`allowf`** (allow flood) flag determines whether a region permits **flooding** of messages to neighbor nodes. This is the primary control mechanism for repeater behavior.

| Flag State | Behavior | Meaning |
|-----------|----------|---------|
| **Set (Enabled)** | Messages CAN be repeated/flooded | Repeaters forward messages scoped to this region |
| **Not Set (Disabled)** | Messages CANNOT be repeated | Repeaters drop messages scoped to this region |

### The Flag Internally

Internally, this is represented as a bit flag in each `RegionEntry`:

```cpp
#define REGION_DENY_FLOOD   0x01    // flag value

// In RegionEntry:
uint8_t flags;  // contains REGION_DENY_FLOOD bit

// When flags & REGION_DENY_FLOOD == 0: Flooding ALLOWED (F flag set)
// When flags & REGION_DENY_FLOOD == 1: Flooding DENIED (F flag not set)
```

Note the **inverted logic**: The `REGION_DENY_FLOOD` flag being SET means flooding is **DENIED**.

### Setting and Clearing `allowf`

**Via CLI Commands:**

```bash
# Allow flooding for a region
region allowf <region_name>

# Deny flooding for a region
region denyf <region_name>
```

**Programmatically:**

```cpp
// Allow flooding
region->flags &= ~REGION_DENY_FLOOD;

// Deny flooding
region->flags |= REGION_DENY_FLOOD;
```

---

## How Repeaters Use Regions and Transport Codes

### The Packet Forwarding Decision

When a repeater receives a flooded packet, it makes a forwarding decision using this logic:

```
1. Extract transport_codes[0] from the incoming packet
2. Try to find a region that matches this transport code
   - Use region_map.findMatch(packet, REGION_DENY_FLOOD)
   - This returns the first region whose transport code matches
3. If a matching region is found:
   - Store it as recv_pkt_region
   - Check if the region has the DENY_FLOOD flag set
   - If DENY_FLOOD is SET: Block the packet (don't forward)
   - If DENY_FLOOD is NOT SET: Allow forwarding
4. If NO matching region is found:
   - Check the wildcard region's flags
   - If wildcard allows flooding: Forward the packet
   - If wildcard denies flooding: Block the packet
```

### Code Reference

From `simple_repeater/MyMesh.cpp`:

```cpp
bool MyMesh::filterRecvFloodPacket(mesh::Packet* pkt) {
  // Determine which region this packet belongs to
  if (pkt->getRouteType() == ROUTE_TYPE_TRANSPORT_FLOOD) {
    // Packet has explicit transport codes
    recv_pkt_region = region_map.findMatch(pkt, REGION_DENY_FLOOD);
  } else if (pkt->getRouteType() == ROUTE_TYPE_FLOOD) {
    // Regular flood packet (no transport codes)
    if (region_map.getWildcard().flags & REGION_DENY_FLOOD) {
      recv_pkt_region = NULL;  // Wildcard denies flooding
    } else {
      recv_pkt_region = &region_map.getWildcard();  // Wildcard allows
    }
  } else {
    recv_pkt_region = NULL;  // Direct routes don't use regions
  }
  return false;  // Continue with normal processing
}

bool MyMesh::allowPacketForward(const mesh::Packet *packet) {
  if (_prefs.disable_fwd) return false;
  if (packet->isRouteFlood() && packet->path_len >= _prefs.flood_max) 
    return false;
  
  // The critical check: if packet has transport codes but no matching region
  // (or wildcard doesn't allow), block it
  if (packet->isRouteFlood() && recv_pkt_region == NULL) {
    MESH_DEBUG_PRINTLN("allowPacketForward: unknown transport code, "
                       "or wildcard not allowed for FLOOD packet");
    return false;
  }
  return true;
}
```

---

## Packet Forwarding Scenarios

### Scenario 1: Packet WITH Transport Code + Region Allows Flooding

**Setup:**
- Repeater has region `#Europe` (allowf enabled)
- Packet arrives with transport_codes[0] matching `#Europe`

**Result:** ✅ **FORWARDED**
- `recv_pkt_region` = `#Europe` region object
- `#Europe` region has allowf flag set (DENY_FLOOD is NOT set)
- Repeater forwards packet to neighbors

---

### Scenario 2: Packet WITH Transport Code + Region Denies Flooding

**Setup:**
- Repeater has region `#SecureData` (allowf disabled / denyf enabled)
- Packet arrives with transport_codes[0] matching `#SecureData`

**Result:** ❌ **DROPPED**
- `recv_pkt_region` = `#SecureData` region object
- `#SecureData` region has denyf flag set (DENY_FLOOD is set)
- `allowPacketForward()` checks the flag and returns false
- Packet is NOT forwarded

---

### Scenario 3: Packet WITH Transport Code + No Matching Region

**Setup:**
- Repeater has regions: `#Europe`, `#Asia`
- Packet arrives with a transport code for `#Africa` (unknown to this repeater)

**Result:** ❌ **DROPPED**
- `recv_pkt_region` = `NULL` (no match found)
- `allowPacketForward()` detects this and blocks the packet
- Repeater doesn't forward, because it can't identify the region

---

### Scenario 4: Regular Flood Packet (No Transport Codes) + Wildcard Allows

**Setup:**
- Repeater has wildcard region with allowf enabled
- Packet arrives with no transport codes (generic flood packet)

**Result:** ✅ **FORWARDED**
- `recv_pkt_region` = `&region_map.getWildcard()`
- Wildcard has allowf enabled
- Repeater forwards the packet

---

### Scenario 5: Regular Flood Packet (No Transport Codes) + Wildcard Denies

**Setup:**
- Repeater has wildcard region with denyf enabled
- Packet arrives with no transport codes

**Result:** ❌ **DROPPED**
- `recv_pkt_region` = `NULL`
- Wildcard has denyf set (denies flooding)
- `allowPacketForward()` blocks the packet

---

### Scenario 6: Direct Route Packets

**Setup:**
- Any repeater configuration
- Packet arrives as a direct route (ROUTE_TYPE_DIRECT)

**Result:** ✅ **ALWAYS FORWARDED** (assuming general forwarding is enabled)
- Direct routes don't use regions
- `recv_pkt_region` = `NULL`
- `allowPacketForward()` doesn't block based on regions
- These packets are forwarded based on other criteria (hop limit, general settings)

---

## Practical Configuration Examples

### Example 1: Simple European Network

Allow messages to flood only within the Europe region:

```bash
region load
#Europe F
region save
```

**Effect:**
- Only packets with transport_codes[0] = Europe are forwarded
- Wildcard (packets without transport codes) are blocked
- All other regions' packets are dropped

---

### Example 2: Hierarchical Regional Structure

Create a multi-level geographic hierarchy:

```bash
region load
* F
  #Europe F
    #UK F
      #London F
      #Manchester F
    #France F
      #Paris F
  #NorthAmerica F
    #USA F
    #Canada F
region save
```

**Effect:**
- All regions allow flooding (F flag on each)
- Packets scoped to any region are forwarded
- Wildcard also allows, so unscoped packets pass through
- Perfect for a global mesh with local organization

---

### Example 3: Restricted Headquarters

Create a configuration where HQ messages don't leak to regional networks:

```bash
region load
#Regions F
  #Europe F
  #NorthAmerica F
  #Asia F
#HQ
region save
```

Then set:
```bash
region denyf #HQ
```

**Effect:**
- Messages scoped to Regions (and their children) are flooded
- Messages scoped to HQ are NOT repeated (blocked)
- HQ messages stay local to the HQ node

---

### Example 4: Open Mesh with Restrictions

Allow most traffic but block specific sensitive regions:

```bash
region load
* F
  #Sensitive
region save
```

Then set:
```bash
region denyf #Sensitive
```

**Effect:**
- By default, all unscoped packets and most regions allow flooding
- Packets explicitly scoped to #Sensitive are blocked by repeaters
- Ensures sensitive data doesn't leak across neighbors

---

## Region Management CLI Commands

### View Current Regions

```bash
# Dump all regions and their flood status
region

# Example output:
# * F
#  #Europe F
#   #UK F
#    #London F
#    #Manchester F
```

### Load and Save Regions

```bash
# Start interactive region loading
region load
#Europe F
  #UK F
    #London F
    #Manchester F
  #France F
    #Paris F

# End with blank line
region save
```

### Modify Region Flood Settings

```bash
# Allow flooding for a region
region allowf #Europe

# Deny flooding for a region
region denyf #Europe

# Check status
region get #Europe
# Output: #Europe F    (means allowf is enabled)
# or:     #Europe      (means allowf is disabled)
```

### Create/Remove Regions

```bash
# Create a new region under a parent
region put #Tokyo #Japan

# Create a top-level region
region put #Africa

# Remove a region (must have no children)
region remove #Tokyo
```

### Set Home Region

```bash
# Set the "home" region (your local region)
region home #London

# View current home
region home
# Output: home is #London
```

### List Regions by Status

```bash
# List regions that allow flooding
region list allowed

# List regions that deny flooding
region list denied
```

---

## Best Practices

### 1. **Design Hierarchies by Intent**
   - Don't just copy geographic boundaries; consider actual mesh topology
   - Group nodes that can directly reach each other at the same level
   - Use parent-child relationships to reflect repeater chains

### 2. **Test Before Deploying**
   - Start with wildcard region allowing (`* F`)
   - Gradually introduce restricted regions
   - Monitor which packets are being forwarded/blocked

### 3. **Use Consistent Naming**
   - Use geographic or functional names consistently
   - Prefix with `#` for public regions (auto-hashed keys)
   - Prefix with `$` for private regions (manually managed keys)

### 4. **Balance Control vs. Openness**
   - Too many restrictions can fragment the network
   - Too few restrictions lose the benefit of regional scoping
   - Default: Start permissive, restrict only where necessary

### 5. **Document Your Configuration**
   - Keep notes on why each region exists
   - Record which nodes belong to which regions
   - Update documentation when topology changes

### 6. **Monitor Repeater Behavior**
   - Check repeater logs to see blocked packets
   - Adjust regions and allowf settings if legitimate traffic is being blocked
   - Use debug output to verify transport code matching

---

## Technical Details for Developers

### Transport Code Calculation

Transport codes are calculated using SHA256-based hashing:

```cpp
uint16_t TransportKey::calcTransportCode(mesh::Packet* packet) {
  // Combines:
  // - Region ID/name
  // - Packet header and payload
  // Result is a 16-bit hash used for region matching
}
```

### Region Matching Algorithm

```cpp
RegionEntry* RegionMap::findMatch(mesh::Packet* packet, uint8_t mask) {
  for (int i = 0; i < num_regions; i++) {
    auto region = &regions[i];
    
    // Check if region allows the operation (based on mask)
    if ((region->flags & mask) == 0) {
      // Get transport keys for this region
      TransportKey keys[4];
      int num = _store->loadKeysFor(region->id, keys, 4);
      
      // Try each key to compute transport code
      for (int j = 0; j < num; j++) {
        uint16_t code = keys[j].calcTransportCode(packet);
        
        // Compare against packet's code
        if (packet->transport_codes[0] == code) {
          return region;  // Match found!
        }
      }
    }
  }
  return NULL;  // No match
}
```

### Storage

Regions are persisted to flash storage as a binary file (`/regions2`):

```
[5 bytes header]
[home_id: 2 bytes]
[wildcard.flags: 1 byte]
[next_id: 2 bytes]
[For each region:
  [id: 2 bytes]
  [parent: 2 bytes]
  [name: 31 bytes]
  [flags: 1 byte]
  [reserved: 128 bytes]
]
```

---

## Troubleshooting

### Issue: Messages Not Reaching Expected Nodes

**Possible Causes:**
1. Transport code doesn't match any region
2. Matching region has `denyf` set
3. Wildcard region has `denyf` set

**Solution:**
1. Verify the regions are configured: `region`
2. Check that relevant regions have `allowf` enabled: `region list allowed`
3. Examine repeater logs for "unknown transport code" errors

### Issue: All Messages Blocked

**Cause:** Wildcard region has `denyf` enabled

**Solution:**
```bash
region allowf *
```

### Issue: Too Much Traffic Passing Through

**Cause:** Wildcard or many regions have `allowf` enabled

**Solution:**
1. Use more specific regions
2. Set `denyf` on regions that shouldn't relay traffic
3. Configure repeater filtering at the application level

### Issue: Nodes in Different Regions Can't Communicate

**Design Issue:** Regions are meant to restrict flooding, not block communication entirely. If you need nodes in different regions to communicate:

1. Use direct routing (doesn't use regions)
2. Set common parent region with `allowf`
3. Create gateway nodes that bridge regions

---

## Summary

| Concept | Definition |
|---------|-----------|
| **Region** | A named organizational zone in the mesh network |
| **Region Scope** | Which region a message is intended for, encoded in transport codes |
| **Transport Code** | A 16-bit hash computed from region ID and packet data |
| **allowf Flag** | Determines if a region allows flooding/repeating (opposite of REGION_DENY_FLOOD) |
| **Repeater** | A node that forwards messages to neighbors based on region scope and allowf settings |
| **Wildcard Region `*`** | The root/default region, used for unscoped packets |

**Core Rule:** A repeater forwards a packet if and only if:
1. The packet's transport code matches one of the repeater's configured regions, OR
2. The packet is unscoped and the wildcard region allows flooding

Both conditions must have the corresponding region's `allowf` flag enabled.

