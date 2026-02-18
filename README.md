# 1. Meshcore region guide
Meshcore regions are introduced to limit the load on our mesh network. By adding regions, the network will contain fewer unnecessary hops.  
IMPORTANT NOTE: At the moment of writing, regions will only apply to channels and will not affect direct messages (DMs).

# 2. The core concept
1. A repeater adds regions
2. A repeater adds allow flood rules
3. A client node user adds a region flag to a message in a channel
4. The chosen region will apply to the sent message
5. Only repeaters that contain the provided region flag will repeat that message

# 3. Important notes
- By default, a repeater will have a region set to *. This * means a repeater will forward messages without a region flag added to that message.  
This will prevent blocking messages and allow public channels to be world-scoped.
- At the time of writing, there is no working hierarchy. This means nl (The Netherlands) is a scope, but nl-li (The Netherlands - Limburg) is as well, so it will allow both.

# 4. Setting a region on your repeater
There are two ways to set the region on your repeater:
1. Command line
2. App

## 4.1 What does it mean?
- Region: The regions that apply to the repeater
- Allowf: Means allowing floods/forwards from a specified region

## 4.2 Set region via Command line
Say my repeater is located in The Netherlands — region Limburg. It is possible to add more than one region. This is recommended when you live in an area that borders multiple regions.  
Setting your region:
```
region put nl-li (setting it to the Netherlands and Limburg (li))
region put nl-nb (setting it to the Netherlands and North Brabant (nb))
region save (By saving, you'll not need a reboot)
```
Setting allow floods:
```
region allowf nl-li (setting it to the Netherlands and Limburg (li))
region allowf nl-nb (setting it to the Netherlands and North Brabant (nb))
region save (By saving, you'll not need a reboot)
```

## 4.2 Set region in App 
1. Open the app
2. Sign in to your repeater
3. Go to settings
4. Click 'Manage regions'
<img width="1170" height="332" alt="image" src="https://github.com/user-attachments/assets/b9565a47-d925-4a17-9996-ff499fef979d" />

5. Click the '+' icon
6. Add your region (for example 'nl-li')
7. Save
8. By default, allow flood is disabled — enable this if needed
<img width="473" height="1024" alt="image" src="https://github.com/user-attachments/assets/275e71c2-adb9-4fd4-86d8-19486119e4e4" />

# 5. Add regions to be used for channels in the app



