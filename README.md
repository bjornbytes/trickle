Trickle
===

Trickle is a bitstream for Lua.  When writing data to a trickle stream, you must specify its size in
bits.  The stream will automatically pack this data into compressed bytes, which often leads to
greatly reduced network bandwidth.  This is useful in networking applications where it is necessary
to reduce network traffic, such as realtime multiplayer games.

Usage
---

```lua
local trickle = require 'trickle'
local stream = trickle.create()

stream:write(7, '3bits')
stream:write(true, 'bool')
stream:write(1.8, 'float')

-- send over the network

stream:read('3bits') -- 7
stream:read('bool') -- true
stream:read('float') -- 1.8
```

Signatures
---

A signature describes a high level template for a message.  It can be used to reduce duplication in
reading and writing of messages, and allows reading and writing of tables of data without having to
write each individual field in a specific order.  First, define a signature:

```lua
local signature = {
  { 'playerId', '4bits' },
  { 'x', '8bits' },
  { 'y', '8bits' },
  { 'direction', '9bits' } -- direction in degrees
}
```

Then, pack a message into a trickle stream using the signature:

```lua
local playerData = {
  playerId = 1,
  x = 63,
  y = 41
  direction = 270
}

stream:pack(playerData, signature)
```

`unpack` may be used to decode the stream:

```lua
local playerData = stream:unpack(signature)
```

Let's say we only want to send the position of the player if it has changed.  Right now this isn't
possible with our signature, because we have to send 8 bits for the x position and 8 bits for the y
position to keep the structure of the signature uniform for the receiving end.  **Delta groups**
provide a mechanism to specify these optional groups of data.

To declare a delta group, specify a `delta` key in the signature table:

```lua
local signature = {
  { 'playerId', '4bits'},
  { 'x', '8bits' },
  { 'y', '8bits' },
  { 'direction', '9bits' },
  delta = {
    { 'x', 'y' },
    'direction'
  }
}
```

What we've done is defined two optional pieces of our message: a group for the position of the
player and a group for the direction of the player.  Now, if we pack a message like this:

```lua
local playerData = {
  playerId = 1,
  x = nil,
  y = nil,
  direction = nil
}

stream:pack(playerData, signature)
```

trickle will only pack the 4 bit player id into the message, which saves 25 bits!  Internally,
a single bit is added to the beginning of the message for each delta group, signifying whether or
not its data is present.  This means that if a delta group contains more than one key (such as the
'x' and 'y' group above), either all of the data must be present or all of it must be `nil`.  This,
for example, would result in an error:

```lua
local playerData = {
  playerId = 1,
  x = 17,
  y = nil,
  direction = 90
}

stream:pack(playerData, signature) -- Error: Only part of message delta group "x, y" was provided.
```

Documentation
---

Coming soon.

License
---

MIT, see [`LICENSE`](LICENSE) for details.
