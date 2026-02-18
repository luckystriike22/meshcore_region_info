## Table of Contents

- [1. Meshcore region guide](#1-meshcore-region-guide)
- [2. The core concept](#2-the-core-concept)
- [3. Important notes](#3-important-notes)
- [4. Setting a region on your repeater](#4-setting-a-region-on-your-repeater)
  - [4.1 What does it mean?](#41-what-does-it-mean)
  - [4.2 Set region via Command line](#42-set-region-via-command-line)
  - [4.2 Set region in App](#42-set-region-in-app)
- [5. Add regions that can be used for channels (client app)](#5-add-regions-that-can-be-used-for-channels-client-app)
- [5 Flow charts](#5-flow-charts)


# 1. Meshcore region guide
Meshcore regions are introduced to limit the load on our mesh network. By adding regions, the network will contain fewer unnecessary hops. At the moment of writing, regions will only apply to channels and will not affect direct messages (DMs).

# 2. The core concept
1. A repeater adds regions
2. A repeater adds allow flood rules
3. A client node user adds a region flag to a message in a channel
4. The chosen region will apply to the sent message
5. Only repeaters that contain the provided region flag will repeat that message

# 3. Important notes
- At the moment of writing, regions will only apply to channels and will not affect direct messages (DMs).
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
It is possible to add more than one region. This is recommended when you live in an area that borders multiple regions.  
For example: my repeater is located in The Netherlands — region Limburg. Because limburg borders it is best practice to allow nearby regions (North Brabant)
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
<img width="585" height="166" alt="image" src="https://github.com/user-attachments/assets/b9565a47-d925-4a17-9996-ff499fef979d" />

5. Click the '+' icon
6. Add your region (for example 'nl-li')
7. Save
8. By default, allow flood/forwarding is disabled — enable this if needed
<img width="237" height="512" alt="image" src="https://github.com/user-attachments/assets/275e71c2-adb9-4fd4-86d8-19486119e4e4" />

# 5. Add regions that can be used for channels (client app)
1. Open a channel
2. Click the dots in the top right corner
3. Click "Set Region Scope"
<img width="223" height="237" alt="image" src="https://github.com/user-attachments/assets/1c383be3-b3da-46e2-8e82-4b5c4b9e11d2" />

4. Add regions
<img width="293" height="305" alt="image" src="https://github.com/user-attachments/assets/9deda9d0-2af2-4cac-8c9c-be747dda8a56" />

5. After creating regions, you can apply one to a channel. You can change it by clicking the channel name. The app will remember the region for each channel.
<img width="293" height="292" alt="image" src="https://github.com/user-attachments/assets/f70d8236-f4d5-45a4-9445-407d122ae94f" />


# 5 Flow charts
<img width="538" height="433" alt="image" src="https://github.com/user-attachments/assets/c2b0c94c-2771-4ebb-9174-6891ef54f6b4" />
<img width="1158" height="348" alt="image" src="https://github.com/user-attachments/assets/47c15eca-a427-4bd8-9e55-b79d15572a16" />






