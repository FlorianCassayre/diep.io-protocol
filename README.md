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
-- | -- | -- | --
**uint8** | 1 | An integer between 0 and 255 | Unsigned 8-bit integer
**uint16** | 2 | An integer between 0 and 65535 | Unsigned 16-bit integer
**uint32** | 4 | An integer between 0 and 4294967295 | Unsigned 32-bit integer
**int8** | 1 | An integer between -128 and 127 | Signed 8-bit integer
**int16** | 2 | An integer between -32768 and 32767 | Signed 16-bit integer
**int32** | 4 | An integer between -2147483647 and 2147483646 | Signed 32-bit integer
**float32** | 4 | A floating point number | Floating point number with 32-bit precision
**float64** | 8 | A floating point number | Floating point number with 64-bit precision
**String** | ≥ 1 | A variable-length UTF-8 string | The string ends when byte `0x00` is reached

## Packet format

Every packet begins with **one** byte indicating the packet ID and then the **content** follows. Packets can be split if they are too large but this behavior is automatically corrected by the buffer manager.

Type | Size (bytes) | Notes
-- | -- | --
**Packet ID** | 1 | There can be 256 different packets, but in practice there are only a few. Sending an unknown packet ID will immediately close the connection.
**Content** | ≥ 0 | Packets can have an empty body

## Packets

Some packets might be missing and their purposes haven't been totally discovered.

### Serverbound
Packets sent from the client to the server.

Packet ID | Arbitrary name | Purpose
-- | -- | --
`0x00` | User ID | Sent at the begining of the connection, contains the ID of the player.
`0x01` | Player control | Contains the location of the mouse, if the player is shooting and the state of the 4-directional keys
`0x02` | Start game | Contains the nickname entered by the player
`0x03` | Choose upgrade | Tells the server which upgrade the player wants to choose
`0x05` | Heartbeat | The server checks if the player is still online. This packet is sent every 0.1 second.

### Clientbound
Packets sent from the server to the client.

Packet ID | Arbitrary name | Purpose
-- | -- | --
`0x00` | ??? | ???
`0x02` | ??? | Contains **sometimes** the shop data and the leaderboard (?)
`0x04` | Server location | Tells the client its location
`0x05` | Heartbeat | The client checks if the player is still online. This packet is sent every 0.1 second.
