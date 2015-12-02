Trickle
===

Trickle is a bitstream for Lua.  When writing data to a trickle stream, you must specify its size in
bits.  The stream will automatically pack this data into compressed bytes, which often leads to
greatly reduced network bandwidth.  This is useful in networking applications where it is necessary
to reduce network traffic, such as realtime multiplayer games.

How it works
---

Coming soon.

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

Coming soon.

Documentation
---

Coming soon.

License
---

MIT, see [`LICENSE`](LICENSE) for details.
