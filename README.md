# diep.io Protocol

[diep.io](http://diep.io) is a small online war game featuring tanks, bullets and blobs. There are several game modes but this document only concentrates on the FFA mode.

1. [Communication](#communication)
1. [Data types](#data-types)
1. [Packet format](#packet-format)
1. [Packets](#packets)
   1. [Serverbound](#serverbound)
   1. [Clientbount](#clientbound)

## Communication

Data is transmitted through a http websocket pipe. 

## Data types

This game uses standard data formats. Bytes are in **big endian** order. The size of each type might not be constant (eg. strings, varints).

Name | Size (bytes) | Data range | Notes
--- | --- | --- | ---
**uint8** | 1 | An integer between 0 and 255 | Unsigned 8-bit integer
**uint16** | 2 | An integer between 0 and 65535 | Unsigned 16-bit integer
**uint32** | 4 | An integer between 0 and 4294967295 | Unsigned 32-bit integer
**int8** | 1 | An integer between -128 and 127 | Signed 8-bit integer
**int16** | 2 | An integer between -32768 and 32767 | Signed 16-bit integer
**int32** | 4 | An integer between -2147483647 and 2147483646 | Signed 32-bit integer
**float32** | 4 | A floating point number | Floating point number with 32-bit precision
**float64** | 8 | A floating point number | Floating point number with 64-bit precision
**String** | ≥ 1 | A variable-length UTF-8 string | The string ends when byte `0x00` is reached
**T[i]** | ? | A fixed size array of a type | Arrays can contain any type of data

## Packet format

Every packet begins with **one** byte indicating the packet ID and then the **content** follows. Packets can be split if they are too large but this behavior is automatically corrected by the buffer manager.

Type | Size (bytes) | Notes
--- | --- | ---
**Packet ID** | 1 | There can be 256 different packets, but in practice there are only a few. Sending an unknown packet ID will immediately close the connection.
**Content** | ≥ 0 | Packets can have an empty body

## Packets

Some packets might be missing and their purposes haven't been totally discovered.

### Serverbound
Packets sent from the client to the server.

#### Recap

Packet ID | Arbitrary name | Purpose
--- | --- | ---
`0x00` | User ID | Sent at the begining of the connection, contains the ID of the player.
`0x01` | Player control | Contains the location of the mouse, if the player is shooting and the state of the 4-directional keys
`0x02` | Start game | Contains the nickname entered by the player
`0x03` | Tank upgrade | Tells the server which upgrade the player wants to choose
`0x04` | Tank evolution | Tells the server which evolution the player wants to choose
`0x05` | Heartbeat | The server checks if the player is still online. This packet is sent every 0.1 second.

#### `0x00` User ID
Field | Type | Notes
--- | --- | ---
User ID | String | No maximal length specified

#### `0x01` Player control
Field | Type | Notes
--- | --- | ---
??? | 8 to 10 bytes | Mouse location (?)
Keys | uint8 | State (pressed or released) of 5 keys encoded in bits

Codes for each key can be found in the following table. Each key state is added to the stack using the logic OR ( =| ) operator.

Key | Code (byte) | Code (binary)
--- | --- | ---
Mouse left button | `0x01` | `0b00000001`
Arrow up | `0x02` | `0b00000010`
Arrow left | `0x04` | `0b00000100`
Arrow down | `0x08` | `0b00001000`
Arrow right | `0x10` | `0b00010000`

#### `0x02` Start game
Field | Type | Notes
--- | --- | ---
Nickname | String | Nicknames must be between 0 and 15 characters long

#### `0x03` Tank upgrade
Field | Type | Notes
--- | --- | ---
Upgrade ID | uint8 | The upgrade level doesn't matter

Upgrades IDs can be found in the following table.

Upgrade | Code (byte)
--- | ---
Health regeneration | `0x0e`
Maximum health | `0x0c`
Body damage | `0x0a`
Bullet speed | `0x08`
Bullet penetration | `0x06`
Bullet damage | `0x04`
Reload | `0x02`
Movement speed | `0x00`

#### `0x04` Tank evolution
Field | Type | Notes
--- | --- | ---
Tank ID | uint8 | Each tank has its own ID

Tanks IDs are as following.

Tier | Levels requiered
--- | ---
**0** | 0
**1** | 15
**2** | 30
**3** | 45

Tier | Tank name | Obtainable after | Code (byte)
--- | --- | --- | ---
1 | Twin | Normal | `0x02`
1 | Flank Guard | Normal | `0x10`
1 | Machine Gun | Normal | `0x0e`
1 | Sniper | Normal | `0x0c`
2 | Triple Shot | Twin | `0x06`
2 | Twin Flank | Twin **OR** Flank Guard | `0x1a`
2 | Quad Tank | Twin **OR** Flank Guard | `0x08`
2 | Tri Angle | Flank Guard | `0x12`
2 | Destroyer | Machine Gun | `0x14`
2 | Gunner | Machine Gun | `0x28`
2 | Assassin | Sniper | `0x1e`
2 | Overseer | Sniper | `0x16`
2 | Hunter | Sniper | `0x26`
3 | Triplet | Triple Shot | `0x04`
3 | Penta Shot | Triple Shot | `0x1c`
3 | Triple Twin | Twin Flank | `0x24`
3 | Octo Tank | Twin Flank **OR** Quad Tank | `0x0a`
3 | Booster | Tri Angle | `0x2e`
3 | Fighter | Tri Angle | `0x30`
3 | Hybrid | Destroyer | `0x32`
3 | Stalker | Assassin | `0x2a`
3 | Ranger | Assassin | `0x2c`
3 | Manager | Overseer | `0x34`
3 | Overlord | Overseer | `0x18`
3 | Necromancer | Overseer | `0x22`

#### `0x05` Heartbeat
This packet is empty.


### Clientbound
Packets sent from the server to the client.

#### Recap

Packet ID | Arbitrary name | Purpose
--- | --- | ---
`0x00` | ??? | ???
`0x02` | ??? | Contains **sometimes** the shop data and the leaderboard (?)
`0x04` | Server location | Tells the client its location
`0x05` | Heartbeat | The client checks if the server is still online. This packet is sent every 0.1 second.

#### `0x00` ???
Field | Type | Notes
--- | --- | ---
Counter | uint8 | Counts from 128 to 255 (?)
??? | ??? | ???

#### `0x02` ???
Field | Type | Notes
--- | --- | ---
??? | ??? | ???
Nicknames | String[10] | 10 best players on the server
??? | ??? | ???
Scores | float32[10] | And their score
??? | ??? | ???

#### `0x04` Server location
Field | Type | Notes
--- | --- | ---
Location name | String | Contains informations about the server's location

#### `0x05` Heartbeat
This packet is empty.
