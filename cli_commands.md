# CLI Commands

This document provides an overview of CLI commands that can be sent to MeshCore Repeaters, Room Servers and Sensors.

## Navigation

- [Operational](#operational)
- [Neighbors](#neighbors-repeater-only)
- [Statistics](#statistics)
- [Logging](#logging)
- [Information](#info)
- [Configuration](#configuration)
  - [Radio](#radio)
  - [System](#system)
  - [Routing](#routing)
  - [ACL](#acl)
  - [Region Management](#region-management-v110)
    - [Region Examples](#region-examples)
  - [GPS](#gps-when-gps-support-is-compiled-in)
  - [Sensors](#sensors-when-sensor-support-is-compiled-in)
  - [Bridge](#bridge-when-bridge-support-is-compiled-in)

---

## Operational


### Reboot the node
**Usage:**
- `reboot`

**Description:**
Restarts the node immediately. Use this command after changing settings that require a reboot to take effect, or to recover from an unexpected state.

**Example:**
```
reboot
```
Use after updating radio parameters or firmware.

---


### Reset the clock and reboot
**Usage:**
- `clkreboot`

**Description:**
Resets the node's internal clock to zero and then reboots. Useful for troubleshooting time synchronization issues or starting a new session with a clean clock.

**Example:**
```
clkreboot
```
Use if the node's time is out of sync and you want to reset it completely.

---


### Sync the clock with the remote device
**Usage:**
- `clock sync`

**Description:**
Synchronizes the node's clock with a remote device. Use this to ensure all nodes in the network have the same time, which is important for time-based operations and logs.

**Example:**
```
clock sync
```
Use after connecting to a new network or if you notice time drift between nodes.

---


### Display current time in UTC
**Usage:**
- `clock`

**Description:**
Shows the current UTC time on the node. Useful for verifying time synchronization and debugging time-based features.

**Example:**
```
clock
```
Use to check the node's current time.

---


### Set the time to a specific timestamp
**Usage:**
- `time <epoch_seconds>`

**Parameters:**
- `epoch_seconds`: Unix epoch time (seconds since 1970-01-01 00:00:00 UTC)

**Description:**
Manually sets the node's clock to a specific Unix timestamp. Use this if you need to correct the time without a remote sync or GPS.

**Example:**
```
time 1700000000
```
Set the time to a known value if the node is isolated or after a battery replacement.

---


### Send a flood advert
**Usage:**
- `advert`

**Description:**
Broadcasts a flood advert to the network, announcing the node's presence and status. Use to manually trigger network discovery or update neighbors after a configuration change.

**Example:**
```
advert
```
Use after changing node name, location, or radio settings to update the network.

---


### Start an Over-The-Air (OTA) firmware update
**Usage:**
- `start ota`

**Description:**
Initiates a firmware update process over the air. Use this command to upgrade the node's firmware remotely. Ensure the new firmware is available and compatible.

**Example:**
```
start ota
```
Use when a new firmware version is released and you want to update without physical access.

---


### Erase/Factory Reset
**Usage:**
- `erase`

**Description:**
Restores the node to factory settings, erasing all configuration and data. **This is destructive and cannot be undone.** Use only if you want to start over or repurpose the device.

**Serial Only:** Yes

**Warning:** _**This is destructive!**_

**Example:**
```
erase
```
Use before selling or transferring a device, or to recover from persistent misconfiguration.

---

## Neighbors (Repeater Only)


### List nearby neighbors
**Usage:**
- `neighbors`

**Description:**
Displays a list of the 8 most recently heard neighbor nodes, showing their public key prefix, last seen timestamp, and signal quality (SNR). Use to check network connectivity and discover nearby repeaters.

**Example:**
```
neighbors
```
Use to verify which nodes are within range and active.

---


### Remove a neighbor
**Usage:**
- `neighbor.remove <pubkey_prefix>`

**Parameters:**
- `pubkey_prefix`: The public key prefix of the node to remove

**Description:**
Removes a specific neighbor from the neighbor list. Use if a node is no longer present or you want to clear stale entries.

**Example:**
```
neighbor.remove 1a2b3c4d
```
Use if a neighbor is no longer valid or has been replaced.

---

## Statistics


### Clear Stats
**Usage:** `clear stats`

**Description:**
Resets all system, radio, and packet statistics to zero. Use before starting a new test or after troubleshooting to clear old data.

**Example:**
```
clear stats
```
Use before measuring performance or after a reboot.

---


### System Stats - Battery, Uptime, Queue Length and Debug Flags
**Usage:**
- `stats-core`

**Description:**
Displays core system statistics, including battery voltage, uptime, message queue length, and debug flags. Use to monitor node health and diagnose issues.

**Serial Only:** Yes

**Example:**
```
stats-core
```
Use to check battery status or debug system behavior.

---


### Radio Stats - Noise floor, Last RSSI/SNR, Airtime, Receive errors
**Usage:** `stats-radio`

**Description:**
Shows radio performance metrics, such as noise floor, last received signal strength (RSSI/SNR), airtime usage, and receive errors. Use to optimize antenna placement or troubleshoot radio issues.

**Serial Only:** Yes

**Example:**
```
stats-radio
```
Use to analyze radio environment and performance.

---


### Packet stats - Packet counters: Received, Sent
**Usage:** `stats-packets`

**Description:**
Displays counters for packets received and sent. Use to monitor network activity and detect communication problems.

**Serial Only:** Yes

**Example:**
```
stats-packets
```
Use to verify if the node is actively communicating.

---

## Logging


### Begin capture of rx log to node storage
**Usage:** `log start`

**Description:**
Starts recording received packet logs to the node's storage. Use to capture network activity for later analysis.

**Example:**
```
log start
```
Use before a test or when debugging intermittent issues.

---


### End capture of rx log to node storage
**Usage:** `log stop`

**Description:**
Stops recording the received packet log. Use after capturing enough data for analysis.

**Example:**
```
log stop
```
Use to end a logging session.

---


### Erase captured log
**Usage:** `log erase`

**Description:**
Deletes all captured logs from the node's storage. Use to free up space or clear old logs before a new test.

**Example:**
```
log erase
```
Use before starting a new experiment.

---


### Print the captured log to the serial terminal
**Usage:** `log`

**Description:**
Outputs the captured log to the serial terminal. Use to review recorded network activity.

**Serial Only:** Yes

**Example:**
```
log
```
Use to analyze captured logs after a test.

---

## Info


### Get the Version
**Usage:** `ver`

**Description:**
Displays the firmware version running on the node. Use to verify compatibility or before reporting issues.

**Example:**
```
ver
```
Use to check if the node is up to date.

---


### Show the hardware name
**Usage:** `board`

**Description:**
Shows the hardware model or board name. Use to confirm the device type, especially when managing multiple nodes.

**Example:**
```
board
```
Use to identify the node's hardware.

---

## Configuration

### Radio


#### View or change this node's radio parameters
**Usage:**
- `get radio`
- `set radio <freq>,<bw>,<sf>,<cr>`

**Parameters:**
- `freq`: Frequency in MHz (e.g., 869.525)
- `bw`: Bandwidth in kHz (e.g., 250)
- `sf`: Spreading factor (5-12, higher = longer range, lower = faster)
- `cr`: Coding rate (5-8, higher = more robust, lower = faster)

**Description:**
Displays or sets the LoRa radio parameters. Use to optimize range, speed, and reliability for your deployment. Changing these values can help adapt to local regulations or environmental conditions.

**Example:**
```
set radio 868.1,125,7,5
```
Use a lower spreading factor (e.g., 7) for faster data in short-range, or a higher one (e.g., 11) for long-range, low-speed links.

**Set by build flag:** `LORA_FREQ`, `LORA_BW`, `LORA_SF`, `LORA_CR`

**Default:** `869.525,250,11,5`

**Note:** Requires reboot to apply

---


#### View or change this node's transmit power
**Usage:**
- `get tx`
- `set tx <dbm>`

**Parameters:**
- `dbm`: Power level in dBm (1-22)

**Description:**
Shows or sets the LoRa transmit power. Use higher values for longer range, but be aware of legal limits and hardware capabilities. Lower values save battery and reduce interference.

**Example:**
```
set tx 14
```
Use 14 dBm for typical EU/US operation. Use 20+ dBm only if your hardware supports it and local laws allow.

**Set by build flag:** `LORA_TX_POWER`

**Default:** Varies by board

**Notes:** This setting only controls the LoRa chip. Some nodes have extra amplifiers. Check your board's manual. **Setting too high may be illegal.**

---


#### Change the radio parameters for a set duration
**Usage:**
- `tempradio <freq>,<bw>,<sf>,<cr>,<timeout_mins>`

**Parameters:**
- `freq`: Frequency in MHz (300-2500)
- `bw`: Bandwidth in kHz (7.8-500)
- `sf`: Spreading factor (5-12)
- `cr`: Coding rate (5-8)
- `timeout_mins`: Duration in minutes (must be > 0)

**Description:**
Temporarily changes the radio settings for a specified time. Use for testing or temporary compliance with local rules. Settings revert after timeout or reboot.

**Example:**
```
tempradio 868.3,125,7,5,10
```
Use to test a new frequency for 10 minutes without permanent change.

**Note:** Not saved to preferences; clears on reboot.

---


#### View or change this node's frequency
**Usage:**
- `get freq`
- `set freq <frequency>`

**Parameters:**
- `frequency`: Frequency in MHz (e.g., 868.1)

**Description:**
Shows or sets the radio frequency. Use to comply with local regulations or avoid interference.

**Example:**
```
set freq 868.1
```
Use a frequency allowed in your country. Requires reboot to apply.

### System


#### View or change this node's name
**Usage:**
- `get name`
- `set name <name>`

**Parameters:**
- `name`: Node name (max 24-32 bytes, depending on location)

**Description:**
Shows or sets the node's advertised name. Use to identify nodes in the network. Useful for deployments with many devices.

**Example:**
```
set name repeater-west
```
Use a descriptive name for easier management.

**Set by build flag:** `ADVERT_NAME`

**Default:** Varies by board

**Note:** Max length varies. Emoji/unicode may use more bytes.

---


#### View or change this node's latitude
**Usage:**
- `get lat`
- `set lat <degrees>`

**Parameters:**
- `degrees`: Latitude in decimal degrees (e.g., 51.5074)

**Description:**
Shows or sets the node's latitude. Use for location-aware features or to help identify node placement.

**Example:**
```
set lat 51.5074
```
Set to your node's actual location for accurate network mapping.

**Set by build flag:** `ADVERT_LAT`

**Default:** `0`

---


#### View or change this node's longitude
**Usage:**
- `get lon`
- `set lon <degrees>`

**Parameters:**
- `degrees`: Longitude in decimal degrees (e.g., -0.1278)

**Description:**
Shows or sets the node's longitude. Use for mapping and location-based routing.

**Example:**
```
set lon -0.1278
```
Set to your node's actual longitude for accurate network mapping.

**Set by build flag:** `ADVERT_LON`

**Default:** `0`

---


#### View or change this node's identity (Private Key)
**Usage:**
- `get prv.key`
- `set prv.key <private_key>`

**Parameters:**
- `private_key`: 64-character hex string

**Description:**
Shows or sets the node's private key. Use only for advanced configuration or key recovery. Changing this will change the node's identity.

**Example:**
```
set prv.key 0123456789abcdef... (64 hex chars)
```
Use only if you need to restore a backup or assign a specific identity.

**Serial Only:**
- `get prv.key`: Yes
- `set prv.key`: No

**Note:** Requires reboot to take effect.

---


#### View or change this node's admin password
**Usage:**
- `get password`
- `set password <password>`

**Parameters:**
- `password`: Admin password

**Description:**
Shows or sets the admin password. Use to secure access to sensitive commands. Only nodes with this password can perform admin actions.

**Example:**
```
set password mysecret
```
Change the password after initial setup for security.

**Set by build flag:** `ADMIN_PASSWORD`

**Default:** `password`

**Note:** Echoed back for confirmation. Any node using this password is added to the admin ACL.

---


#### View or change this node's guest password
**Usage:**
- `get guest.password`
- `set guest.password <password>`

**Parameters:**
- `password`: Guest password

**Description:**
Shows or sets the guest password. Use to allow limited access for non-admin users.

**Example:**
```
set guest.password visitor
```
Set a guest password to allow read-only access for visitors.

**Set by build flag:** `ROOM_PASSWORD` (Room Server only)

**Default:** `<blank>`

---


#### View or change this node's owner info
**Usage:**
- `get owner.info`
- `set owner.info <text>`

**Parameters:**
- `text`: Owner information (use `|` for newlines)

**Description:**
Shows or sets the owner information text. Use to store contact info or deployment notes.

**Example:**
```
set owner.info John Doe|john@example.com
```
Use to help identify the owner if the device is lost.

**Default:** `<blank>`

**Note:** `|` becomes newline. Requires firmware 1.12+.

---


#### Fine-tune the battery reading
**Usage:**
- `get adc.multiplier`
- `set adc.multiplier <value>`

**Parameters:**
- `value`: ADC multiplier (0.0-10.0)

**Description:**
Adjusts the multiplier for battery voltage readings. Use to calibrate battery measurements if readings are inaccurate.

**Example:**
```
set adc.multiplier 1.05
```
Increase if voltage reads too low, decrease if too high.

**Default:** `0.0` (board-defined)

**Note:** Not all boards support this.

---


#### View or change this node's power saving flag (Repeater Only)
**Usage:**
- `powersaving <state>`
- `powersaving`

**Parameters:**
- `state`: `on` or `off`

**Description:**
Enables or disables power saving mode. When enabled, the device sleeps between transmissions to save battery. Use for battery-powered repeaters.

**Example:**
```
powersaving on
```
Enable for solar or battery nodes; disable for mains-powered always-on repeaters.

**Default:** `on`

**Note:** Saves power at the cost of slightly higher latency.

---

### Routing


#### View or change this node's repeat flag
**Usage:**
- `get repeat`
- `set repeat <state>`

**Parameters:**
- `state`: `on` or `off`

**Description:**
Enables or disables repeating (relaying) of packets. Use to control whether this node acts as a repeater.

**Example:**
```
set repeat off
```
Disable on end-nodes or sensors to reduce network traffic.

**Default:** `on`

---


#### View or change the retransmit delay factor for flood traffic
**Usage:**
- `get txdelay`
- `set txdelay <value>`

**Parameters:**
- `value`: Transmit delay factor (0-2, e.g., 0.5, 1, 2)

**Description:**
Sets the random delay before retransmitting flood packets. Lower values mean faster flooding but higher risk of collisions; higher values slow down flooding but reduce collisions.

**Example:**
```
set txdelay 0.5
```
Use 0.5 for small, low-traffic networks (fast propagation). Use 1 for medium networks. Use 2 for large, dense networks to avoid packet collisions.

**Default:** `0.5`

---


#### View or change the retransmit delay factor for direct traffic
**Usage:**
- `get direct.txdelay`
- `set direct.txdelay <value>`

**Parameters:**
- `value`: Direct transmit delay factor (0-2)

**Description:**
Sets the delay for direct (unicast) retransmissions. Lower values are faster but may cause more collisions in busy networks.

**Example:**
```
set direct.txdelay 0.2
```
Use 0.2 for most cases. Increase for very busy networks.

**Default:** `0.2`

---


#### [Experimental] View or change the processing delay for received traffic
**Usage:**
- `get rxdelay`
- `set rxdelay <value>`

**Parameters:**
- `value`: Receive delay base (0-20)

**Description:**
Adds a base delay to processing received packets. Use for testing or to simulate latency.

**Example:**
```
set rxdelay 5
```
Use only for experiments or debugging.


**Default:** `0.0`

---

### In-Depth: txdelay and rxdelay Explained

#### How txdelay Works
The `txdelay` parameter introduces a random delay before retransmitting flood packets. This delay is calculated based on the estimated airtime of the packet and the `txdelay` factor you set. The actual delay is randomized to help reduce the chance of packet collisions, especially in busy networks. Internally, the code multiplies the packet's airtime by a factor and applies a random multiplier, so larger packets and higher `txdelay` values result in longer delays.

**When to use different txdelay values:**
- **Low values (e.g., 0.5):** Use for small, low-traffic networks where collisions are rare and fast propagation is desired.
- **Medium values (e.g., 1):** Use for medium-sized networks with moderate traffic, balancing speed and collision avoidance.
- **High values (e.g., 2):** Use for large, dense, or high-traffic networks to minimize collisions, at the cost of slower message propagation.

**Special note:** In KISS modem mode, `txdelay` is used as a timer before sending (in units of 10ms). Setting it too low may cause more collisions; too high increases latency.

#### How rxdelay Works

The `rxdelay` parameter introduces a base delay before processing and forwarding received packets. This delay is critical for duplicate suppression in mesh networks:

- **How it works in code:** When a packet is received, the node waits for the rxdelay period (plus any score-based delay) before deciding to forward it. During this window, if the node receives the same packet from another neighbor, it recognizes the duplicate and suppresses its own retransmission. This reduces redundant flooding and network congestion, especially in dense deployments.

- **Why/when to use:**
  - **Normal operation:** Use a small but nonzero rxdelay in dense or multi-hop networks to maximize duplicate suppression and minimize unnecessary retransmissions.
  - **Testing/debugging:** Increase rxdelay to simulate higher latency or to observe duplicate suppression in action.
  - **Low-traffic/simple networks:** rxdelay can be set to 0 for fastest forwarding, but this may increase redundant traffic.

- **Parameter effects:**
  - Higher rxdelay increases the window for duplicate detection, improving efficiency but adding latency.
  - Too high values can slow down message propagation unnecessarily.

- **Best practice:**
  - Tune rxdelay based on network density and traffic patterns. For most mesh deployments, a small rxdelay (e.g., 1-5 ms) balances speed and efficiency.

**Example scenario:**
In a dense mesh, Node A and Node B both receive a flood packet. With rxdelay set, Node B may receive a duplicate from Node A during the delay window and suppress its own retransmission, reducing network load.

**Summary:**
rxdelay is a key tool for efficient mesh flooding and duplicate suppression, not just for simulating latency.

#### Summary Table

| Parameter | Use Low Value | Use High Value | Notes |
|-----------|--------------|---------------|-------|
| txdelay   | Small, fast, low-traffic networks | Large, dense, high-traffic networks | Reduces collisions at the cost of speed |
| rxdelay   | Normal operation (keep at 0) | Testing, debugging, simulating latency | Not for production use |


These settings are powerful tools for tuning network performance, reliability, and efficiency. Adjust them based on your deployment size, density, and testing needs.
# Deep Code-Driven CLI Command Reference
...existing code...

---

## Additional Code-Driven Notes and Corrections

- **clear stats**: Also resets all internal counters used for diagnostics, not just user-facing stats. Useful before automated tests.

- **log start/stop/erase**: Logging is only for received packets, not all events. Erasing logs does not affect system operation.

- **set radio**: If invalid parameters are provided, the command is rejected and no changes are made. Requires reboot to take effect.

- **set tx**: Setting a value outside hardware or regulatory limits may be ignored or clamped by the firmware. Always check the board's capabilities.

- **tempradio**: Temporary settings override persistent ones only for the specified duration or until reboot. If the timeout is too short or parameters are invalid, the command is ignored.

- **set name**: Names with forbidden characters ([, ], \, :, ,, ?, *) are rejected by the code. Max length is enforced per board.

- **set prv.key**: Only valid, properly formatted keys are accepted. Invalid keys are rejected with an error.

- **setperm**: Omitting the permissions argument removes the entry from the ACL, not just disables it.

- **region load**: Indentation in the region list is significant and creates parent-child relationships. Max depth is 8 levels.

- **region allowf/denyf**: Wildcard `*` applies to all regions without a specific code.

- **gps**: The output includes status, fix, and satellite count. If GPS is not present or enabled, the command returns an error or 'off'.

- **sensor set**: Only custom or supported sensor keys can be set. Invalid keys are rejected.

- **bridge commands**: All bridge-related settings are only available if the firmware is compiled with bridge support. Invalid values (e.g., out-of-range baud/channel) are rejected.

---

---


#### View or change the airtime factor (duty cycle limit)
**Usage:**
- `get af`
- `set af <value>`

**Parameters:**
- `value`: Airtime factor (0-9)

**Description:**
Limits the node's radio duty cycle. Lower values reduce airtime (better for compliance and battery), higher values allow more traffic.

**Example:**
```
set af 1
```
Use 1 for normal operation. Lower for strict duty cycle regions.

**Default:** `1.0`

---


#### View or change the local interference threshold
**Usage:**
- `get int.thresh`
- `set int.thresh <value>`

**Parameters:**
- `value`: Interference threshold value

**Description:**
Sets the threshold for detecting local interference. Use to tune sensitivity to noisy environments.

**Example:**
```
set int.thresh 0.5
```
Increase if you see too many false positives for interference.

**Default:** `0.0`

---


#### View or change the AGC Reset Interval
**Usage:**
- `get agc.reset.interval`
- `set agc.reset.interval <value>`

**Parameters:**
- `value`: Interval in seconds (rounded down to nearest multiple of 4)

**Description:**
Sets how often the Automatic Gain Control (AGC) is reset. Use to improve reception in changing radio environments.

**Example:**
```
set agc.reset.interval 16
```
Increase interval if you see unstable signal levels.

**Default:** `0.0`

---


#### Enable or disable Multi-Acks support
**Usage:**
- `get multi.acks`
- `set multi.acks <state>`

**Parameters:**
- `state`: `0` (disable) or `1` (enable)

**Description:**
Enables or disables support for multiple acknowledgments per packet. Use to improve reliability in dense networks.

**Example:**
```
set multi.acks 1
```
Enable for networks with many nodes to increase delivery success.

**Default:** `0`

---


#### View or change the flood advert interval
**Usage:**
- `get flood.advert.interval`
- `set flood.advert.interval <hours>`

**Parameters:**
- `hours`: Interval in hours (3-168)

**Description:**
Sets how often the node sends a flood advert. Use to control network update frequency.

**Example:**
```
set flood.advert.interval 12
```
Use shorter intervals for dynamic networks, longer for stable ones.

**Default:** `12` (Repeater), `0` (Sensor)

---


#### View or change the zero-hop advert interval
**Usage:**
- `get advert.interval`
- `set advert.interval <minutes>`

**Parameters:**
- `minutes`: Interval in minutes (rounded down to nearest multiple of 2, 60-240)

**Description:**
Sets how often the node sends a zero-hop (local) advert. Use to control how frequently the node announces itself to direct neighbors.

**Example:**
```
set advert.interval 120
```
Increase for less frequent updates, decrease for more visibility.

**Default:** `0`

---


#### Limit the number of hops for a flood message
**Usage:**
- `get flood.max`
- `set flood.max <value>`

**Parameters:**
- `value`: Maximum flood hop count (0-64)

**Description:**
Limits how far a flood message can propagate. Use to contain traffic within a region or prevent network overload.

**Example:**
```
set flood.max 8
```
Use lower values for small networks, higher for large or multi-hop deployments.

**Default:** `64`

---

### ACL


#### Add, update or remove permissions for a companion
**Usage:**
- `setperm <pubkey> <permissions>`

**Parameters:**
- `pubkey`: Companion public key
- `permissions`: 0 (Guest), 1 (Read-only), 2 (Read-write), 3 (Admin)

**Description:**
Sets or updates access permissions for a companion node. Omit `permissions` to remove the entry. Use to manage network access and security.

**Example:**
```
setperm 1a2b3c4d 2
setperm 1a2b3c4d
```
First line grants read-write, second removes the entry.

---


#### View the current ACL
**Usage:**
- `get acl`

**Description:**
Displays the Access Control List (ACL) for the node. Use to review which public keys have what permissions.

**Serial Only:** Yes

**Example:**
```
get acl
```
Use to audit network access.

---


#### View or change this room server's 'read-only' flag
**Usage:**
- `get allow.read.only`
- `set allow.read.only <state>`

**Parameters:**
- `state`: `on` (enable) or `off` (disable)

**Description:**
Enables or disables read-only mode for the room server. Use to restrict changes from guests.

**Example:**
```
set allow.read.only on
```
Enable to prevent guests from making changes.

**Default:** `off`

---

### Region Management (v1.10.+)


#### Bulk-load region lists
**Usage:**
- `region load`
- `region load <name> [flood_flag]`

**Parameters:**
- `name`: Region name (use `*` for wildcard)
- `flood_flag`: Optional `F` to enable flooding for the region

**Description:**
Loads a list of regions in bulk, optionally enabling flooding. Use to quickly configure region hierarchies and permissions.

**Example:**
```
region load
#Europe F
  #UK
  #France
<blank line to end>
region save
```
Use to set up regional routing and flooding rules efficiently.

**Note:** Indentation creates parent-child relationships (max 8 levels). `region load` with an empty name is interactive and not available remotely.

---


#### Save any changes to regions made since reboot
**Usage:**
- `region save`

**Description:**
Saves all region changes to persistent storage. Use after modifying regions to make changes permanent.

**Example:**
```
region save
```
Always run after `region load` or other region modifications.

---


#### Allow a region
**Usage:**
- `region allowf <name>`

**Parameters:**
- `name`: Region name (or `*` for wildcard)

**Description:**
Allows traffic from the specified region. Use to permit packets from a region or all regions (with `*`).

**Example:**
```
region allowf #Europe
region allowf *
```
Use `*` to allow packets without region codes.

---


#### Block a region
**Usage:**
- `region denyf <name>`

**Parameters:**
- `name`: Region name (or `*` for wildcard)

**Description:**
Blocks traffic from the specified region. Use to deny packets from a region or all regions (with `*`).

**Example:**
```
region denyf #Europe
region denyf *
```
Use `*` to block packets without region codes.

---


#### Show information for a region
**Usage:**
- `region get <name>`

**Parameters:**
- `name`: Region name (or `*` for wildcard)

**Description:**
Displays details about a region, such as permissions and flooding status. Use to audit or debug region settings.

**Example:**
```
region get #Europe
```
Check region status before making changes.

---


#### View or change the home region for this node
**Usage:**
- `region home`
- `region home <name>`

**Parameters:**
- `name`: Region name

**Description:**
Shows or sets the node's home region. Use to define the default region for routing and permissions.

**Example:**
```
region home #Europe
```
Set after moving the node to a new location.

---


#### Create a new region
**Usage:**
- `region put <name> [parent_name]`

**Parameters:**
- `name`: Region name
- `parent_name`: Parent region name (optional, defaults to wildcard)

**Description:**
Creates a new region, optionally as a child of another. Use to expand or reorganize the region hierarchy.

**Example:**
```
region put #London #UK
```
Add new cities or subregions as needed.

---


#### Remove a region
**Usage:**
- `region remove <name>`

**Parameters:**
- `name`: Region name

**Description:**
Deletes a region. All child regions must be removed first. Use to clean up unused or obsolete regions.

**Example:**
```
region remove #London
```
Remove subregions before deleting a parent region.

---


#### View all regions
**Usage:**
- `region list <filter>`

**Parameters:**
- `filter`: `allowed` or `denied`

**Description:**
Lists all defined regions, filtered by allowed or denied status. Use to audit region configuration.

**Example:**
```
region list allowed
```
See which regions are currently permitted.

**Serial Only:** Yes

**Note:** Requires firmware 1.12+

---


#### Dump all defined regions and flood permissions
**Usage:**
- `region`

**Description:**
Prints all regions and their flood permissions. Use for a complete overview, especially on older firmware.

**Example:**
```
region
```
Use to export or review all region settings.

**Serial Only:** For firmware < 1.12.0

---

### Region Examples

**Example 1: Using F Flag with Named Public Region**
```
region load
#Europe F
<blank line to end region load>
region save
```

**Explanation:**
- Creates a region named `#Europe` with flooding enabled
- Packets from this region will be flooded to other nodes

---

**Example 2: Using Wildcard with F Flag**
```
region load 
* F
<blank line to end region load>
region save
```

**Explanation:**
- Creates a wildcard region `*` with flooding enabled
- Enables flooding for all regions automatically
- Applies only to packets without transport codes

---

**Example 3: Using Wildcard Without F Flag**
```
region load 
*
<blank line to end region load>
region save
```
**Explanation:**
- Creates a wildcard region `*` without flooding
- This region exists but doesn't affect packet distribution
- Used as a default/empty region

---

**Example 4: Nested Public Region with F Flag**
```
region load 
#Europe F
  #UK
    #London
    #Manchester
  #France
    #Paris
    #Lyon
<blank line to end region load>
region save
```

**Explanation:**
- Creates `#Europe` region with flooding enabled
- Adds nested child regions (`#UK`, `#France`)
- All nested regions inherit the flooding flag from parent

---

**Example 5: Wildcard with Nested Public Regions**
```
region load 
* F
  #NorthAmerica
    #USA
      #NewYork
      #California
    #Canada
      #Ontario
      #Quebec
<blank line to end region load>
region save
```

**Explanation:**
- Creates wildcard region `*` with flooding enabled
- Adds nested `#NorthAmerica` hierarchy
- Enables flooding for all child regions automatically
- Useful for global networks with specific regional rules

---
### GPS (When GPS support is compiled in)


#### View or change GPS state
**Usage:**
- `gps`
- `gps <state>`

**Parameters:**
- `state`: `on` or `off`

**Description:**
Shows or sets the GPS module state. Use to enable GPS for time/location or disable to save power.

**Example:**
```
gps on
```
Enable GPS when you need accurate time or location.

**Default:** `off`

**Note:** Output: `{status}, {fix}, {sat count}`

---


#### Sync this node's clock with GPS time
**Usage:**
- `gps sync`

**Description:**
Sets the node's clock to GPS time. Use for precise timekeeping, especially in isolated deployments.

**Example:**
```
gps sync
```
Use after enabling GPS.

---


#### Set this node's location based on the GPS coordinates
**Usage:**
- `gps setloc`

**Description:**
Updates the node's stored location to the current GPS coordinates. Use to automate location setup.

**Example:**
```
gps setloc
```
Use after moving the node to a new site.

---


#### View or change the GPS advert policy
**Usage:**
- `gps advert`
- `gps advert <policy>`

**Parameters:**
- `policy`: `none` (no location), `shared` (share GPS), `prefs` (use stored lat/lon)

**Description:**
Controls how the node advertises its location. Use `shared` for real-time GPS, `prefs` for static, or `none` for privacy.

**Example:**
```
gps advert shared
```
Use `none` if you want to hide the node's location.

**Default:** `prefs`

---

### Sensors (When sensor support is compiled in)


#### View the list of sensors on this node
**Usage:** `sensor list [start]`

**Parameters:**
- `start`: Optional starting index (default 0)

**Description:**
Lists all available sensors and their current values. Use to monitor environmental or system data.

**Example:**
```
sensor list
```
Use to check all sensor readings at once.

---


#### View or change the value of a sensor
**Usage:**
- `sensor get <key>`
- `sensor set <key> <value>`

**Parameters:**
- `key`: Sensor name
- `value`: Value to set

**Description:**
Reads or sets a specific sensor value. Use to query or calibrate sensors.

**Example:**
```
sensor get temp
sensor set temp 25
```
Use to check or adjust sensor calibration.

---

### Bridge (When bridge support is compiled in)


#### View or change the bridge enabled flag
**Usage:**
- `get bridge.enabled`
- `set bridge.enabled <state>`

**Parameters:**
- `state`: `on` or `off`

**Description:**
Enables or disables the bridge feature. Use to connect the node to an external network or interface.

**Example:**
```
set bridge.enabled on
```
Enable when connecting to a gateway or external system.

**Default:** `off`

---


#### View the bridge source
**Usage:**
- `get bridge.source`

**Description:**
Shows the current source for bridged packets. Use to verify bridge configuration.

**Example:**
```
get bridge.source
```

---


#### Add a delay to packets routed through this bridge
**Usage:**
- `get bridge.delay`
- `set bridge.delay <ms>`

**Parameters:**
- `ms`: Delay in milliseconds (0-10000)

**Description:**
Sets a delay for packets routed through the bridge. Use to throttle or synchronize traffic.

**Example:**
```
set bridge.delay 1000
```
Increase delay to avoid flooding the external interface.

**Default:** `500`

---


#### View or change the source of packets bridged to the external interface
**Usage:**
- `get bridge.source`
- `set bridge.source <source>`

**Parameters:**
- `source`: `rx` (received) or `tx` (transmitted)

**Description:**
Controls which packets are sent to the bridge. Use `rx` to bridge incoming, `tx` for outgoing packets.

**Example:**
```
set bridge.source rx
```
Use `rx` for monitoring, `tx` for forwarding.

**Default:** `tx`

---


#### View or change the speed of the bridge (RS-232 only)
**Usage:**
- `get bridge.baud`
- `set bridge.baud <rate>`

**Parameters:**
- `rate`: Baud rate (`9600`, `19200`, `38400`, `57600`, `115200`)

**Description:**
Sets the serial speed for the bridge. Use to match the speed of connected equipment.

**Example:**
```
set bridge.baud 57600
```
Lower speeds for legacy devices, higher for modern systems.

**Default:** `115200`

---


#### View or change the channel used for bridging (ESPNow only)
**Usage:**
- `get bridge.channel`
- `set bridge.channel <channel>`

**Parameters:**
- `channel`: Channel number (1-14)

**Description:**
Sets the WiFi/ESPNow channel for bridging. Use to avoid interference or match your network.

**Example:**
```
set bridge.channel 6
```
Choose a channel with minimal interference.

---


#### Set the ESP-Now secret
**Usage:**
- `get bridge.secret`
- `set bridge.secret <secret>`

**Parameters:**
- `secret`: 16-character encryption secret

**Description:**
Sets the encryption key for ESP-Now bridging. Use to secure communications between nodes.

**Example:**
```
set bridge.secret mysecretkey12345
```
Change if you suspect the key is compromised.

**Default:** Varies by board

---
