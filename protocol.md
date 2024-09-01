# Soulseek protocol overview

NOTE: Most of this is from [https://nicotine-plus.org/doc/SLSKPROTOCOL.html](here)

## Important commands
### Getting File Shares
[https://nicotine-plus.org/doc/SLSKPROTOCOL.html#peer-code-4](GetShareFileList)
[https://nicotine-plus.org/doc/SLSKPROTOCOL.html#peer-code-5](SharedFileListResponse)

### File Searching
[https://nicotine-plus.org/doc/SLSKPROTOCOL.html#peer-code-9](FileSearchResponse)
[https://nicotine-plus.org/doc/SLSKPROTOCOL.html#server-code-42](UserSearch)
[https://nicotine-plus.org/doc/SLSKPROTOCOL.html#server-code-26](FileSearch)

### Peer Init Messages
[https://nicotine-plus.org/doc/SLSKPROTOCOL.html#peer-init-messages](Peer init messages)

## Packing
### 8-bit int
* Type: Number
* Size: 1 byte

### 16-bit int
* Type: Number
* Size: 2 bytes (little-endian)

### 32-bit int
* Type: Number
* Size: 4 bytes (little-endian)

### 64-bit int
* Type: Number
* Size: 8 bytes (little-endian)

### Bool
* Type: Number
* Size: 1 byte (0 or 1)

### String
#### Length of String
* Type: uint32
#### String
* Type: byte string

### Bytes
#### Length of Bytes 
* Type: uint32
#### String
* Type: byte string

## Constants
### Connection Types
* ``P`` - Peer To Peer
* ``F`` - File Transfer
* ``D`` - Distributed Network

### Login Failure Reasons
* ``INVALIDUSERNAME`` - Username longer than 30 characters or contains invalid chars (non-ASCII)
* ``INVALIDPASS`` - Password for existing user is incorrect
* ``INVALIDVERSION`` - Client version outdated

### User Status Codes
* ``0`` - Offline
* ``1`` - Away
* ``2`` - Online

### Upload Permissions
* ``0`` - No One
* ``1`` - Everyone
* ``2`` - Users in List
* ``3`` - Permitted Users

### Transfer Directions
* ``0`` - Download from Peer
* ``1`` - Upload to Peer

### Transfer Rejection Reasons
#### Currently in use
* ``Banned`` - Note: SoulseekQt uses 'File not shared.' instead
* ``Cancelled``
* ``File not shared.`` - Note: Ends with dot
* ``File read error.`` - Note: Ends with dot
* ``Pending shutdown.`` - Note: Ends with dot
* ``Queued``
* ``Too many files``
* ``Too many megabytes``

#### Depreciated
* ``Blocked country`` - Note: Exclusive to Nicotine+, not used since V3.2.0
* ``Disallowed extension`` - Note: Sent by Soulseek NS for filtered extensions
* ``File not shared`` - Note: Exclusive to Nicotine+, not used since V3.1.1
* ``Remote file error`` - Note: Sent by SoulSeek NS in response to legacy download requests
* ``User limit of x megabytes exceeded`` - Note: Exclusive to Nicotine+, not used since V3.1.1
* ``User limit of x files exceeded`` - Note: Exclusive to Nicotine+, not used since V3.1.1

### File Attribute Types
* ``0`` - Bitrate (kbps)
* ``1`` - Duration (seconds)
* ``2`` - VBR (0 or 1) - Note: Variable bitrate
* ``3`` - Encoder (unused)
* ``4`` - Sample Rate (Hz)
* ``4`` - Bit Depth (bits)

#### File Attribute Combinations
* Soulseek NS, SoulseekQt (2015-2-21 or earlier), Nicotine+ (lossy), Museek+, SoulSeeX, slskd (lossy):
```{0: bitrate, 1: duration, 2: VBR}```
* SoulseekQt (2015-6-12 and later):
```{0: bitrate, 1: duration}``` (MP3, OGG, WMA, M4A)
```{1: duration, 4: sample rate, 5: bit depth}``` (FLAC, WAV, APE)
```{0: bitrate, 1: duration, 4: sample rate, 5: bit depth}``` (WV)
* Nicotine+ (lossless formats), slskd (lossless formats):
```{1: duration, 4: sample rate, 5: bit depth}```

## Server Messages
### Format
| Message Length | Code   | Message Contents |
| unit32         | uint32 | ...              |

### Server Message Codes

#### Connection details
1. Login
2. Set Listen Port
3. Get Peer Address

#### User Watching/Unwatching and status
5. Watch User
6. Unwatch User
7. Get User Status

#### Chat room status and management
13. Stay in Chat Room
14. Join room
15. Leave Room
16. User Joined Room
17. User Left Room
18. Connect To Peer

#### Private Messages
22. Private Messages
23. Ack Private Messages

#### File Searching
25. File Search Room (Obsolete)
26. File Search

#### Online Status
28. Set Online Status

#### Ping
32. Ping

#### Stats and files
35. Shared Folders and Files
36. Get User Stats

#### Kicked
41. Kicked from Server

#### User Search
42. User Search

#### Room listings
64. Room List

#### Admin Messages
66. Global/Admin  Message

#### Getting privileged Users
69. Privileged Users

#### Parent Stats
71. Have No Parents
83. Parent Min Speed
84. Parent Speed Ratio

#### Privilege checks
92. Check Privileges

#### Embedded Message
93. Embedded Message

#### Accepting Children
100. Accept Children

#### Parents search
102. Possible Parents

#### Wishlist checking
103. Wishlist Search
104. Wishlist Interval

#### Room Ticker management
113. Room Tickers
114. Room Ticker Add
115. Room Ticker Remove
116. Set Room Ticker

#### Searching for rooms
120. Room Search

#### Upload speed setting
121. Send Upload Speed

#### Privilege management
123. Give Privileges

#### Branches
126. Branch Level
127. Branch Root

#### Resetting stuff
130. Reset Distributed

#### Private room management
133. Private Room Users
134. Private Room Add User
135. Private Room Remove User
136. Private Room Cancel Membership
137. Private Room Disown
138. Private Room Unknown (Obsolete)
139. Private Room Added
140. Private Room Removed
141. Private Room Toggle
142. New Password
143. Private Room Add Operator
144. Private Room Remove Operator
145. Private Room Operator Added
146. Private Room Operator Removed
148. Private Room Operators

#### Message all users
149. Message Users

#### Search filtering
160. Excluded Search Phrases

#### Error Messages
1001. Can't Connect To Peer
1003. Can't Create Room

## Examples
### Login
Sent once connection is made.
#### Format
{Message length} {Message code} {Username length} {Username} {Password Length} {Password} {Version} {Hash Length} {Hash} {Minor Version}
#### Data Order
* Send:
  1. string *username*
  2. string *password* Note: Non-empty string required
  3. uint32 *version number* Note: 160 for Nicotine+. (Maybe set our own?)
  4. string *hash* Note: MD5 hex digest of concatenated username and password
  5. uint32 *minor version* Note: 0x13000000 for 157 ns 13e (Set our own here as well?)
* Receive
  1. bool *success*
  If *success* is true:
      2. string *greet* Note: MOTD string
      3. uint32 *own IP address*
      4. string *hash* Note: MD5 hex digest of password string
      5. bool *is supporter* Note: probably not respond with this true ever
  If *success* is false:
      2. bool *failure*
      3. string *reason* Note: See Login Failure Reasons
      
### SetWaitPort
Sent to the server to indicate what port number the client listens to (usually 2234).
If not sent, things go bad.
#### Data Order
* Send:
  1. uint32 *port*
  2. uint32 *unknown* Note: SoulseekQt uses a value of 1
  3. uint32 *obfuscated port* Note: Not sure what this is
* Receive
  * No Message

### GetPeerAddress
Sent to server to ask for peer's address (IP address and port), given the peer's username
#### Data Order
* Send
  1. string *username*
* Receive
  1. string *username*
  2. ip *ip*
  3. uint32 *port*
  4. uint32 *unknown* Note: SoulseekQt uses a value of 1
  5. uint16 *obfuscated port*

### WatchUser
Used to keep updated about a user's status. Server sends GetUserStatus message when a user's status changes.
Note: This will return the initial stats when WatchUser was initially received. 
#### Data Order
* Send
  1. string *username*
* Receive
  1. string *username*
  2. bool *exists*

Eh fuck it look at the Nicotine+ docs at the top
