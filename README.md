Manages the enderstorages and API.

# Documentation

**Connection**
    *This requires ecnet2 https://github.com/migeyel/ecnet
    *And ccryptolib https://github.com/migeyel/ccryptolib
    *The server's address is ```c76qTSo54lzNaqVyizQlM_pbHdHinhUFKU2mnIe19WE```
    *Protocol is set to ```shipify```

Example Client:

```lua
local ecnet2 = require "ecnet2"
local random = require "ccryptolib.random"

-- https://www.random.org/strings/?num=1&len=32&digits=on&upperalpha=on&loweralpha=on&unique=on&format=plain&rnd=new
-- Initialize the random generator.
local postHandle = assert(http.post("https://krist.dev/ws/start", "{}"))
local data = textutils.unserializeJSON(postHandle.readAll())
postHandle.close()
random.init(data.url)
http.websocket(data.url).close()

-- Open the top modem for comms.
ecnet2.open("top")


local id = ecnet2.Identity("/.ecnet2")

local ping = id:Protocol {

    name = "shipify",

    -- Objects must be serialized before they are sent over.
    serialize = textutils.serialize,
    deserialize = textutils.unserialize,
}

-- The server's address.
local server = "c76qTSo54lzNaqVyizQlM_pbHdHinhUFKU2mnIe19WE="

local function main()
    -- Connect to the server.
    local connection = ping:connect(server, "top")

    -- Wait for the greeting.
    print(select(2, connection:receive()))

    -- Read inputs and print ping outputs.
    connection:send({
        key = "key",
        route = "",
        data = {}
    })
    local pretty = require("cc.pretty").pretty
    print(pretty(select(2, connection:receive() )) )
end

parallel.waitForAny(main, ecnet2.daemon)
```


## Structure
```lua
{
  key = "key",
  route = "",
  data = {}
}
````

---

## Routes

### getEnderstorages

```lua id="r1k2lm"
{
  route = "getEnderstorages",
  key = "key",
  data = {
    id = ""
  }
}
```

**Fields**

* **id**: The user's name or UUID

**Response**

```lua id="w8n3qp"
-- Success
{ ok = true, result = {} }

-- Failure
{ ok = false, error = "This user doesnt exist!" }
```

---

### send

```lua id="t6bz91"
{
  key= "",
  route = "send",
  data = {
    fromAddress = "",
    toAddress = "",
    slots = {}
  }
}
```

**Fields**

* **fromAddress**: Address you are sending from (`name@user`)
* **toAddress**: Full destination address (e.g., `primary@sethgamer1223`)
* **slots**: Table of slot ranges, e.g.:

```lua
{
  {1, 5},
  {10, 12}
}
```

**Response**

```lua id="p0x4de"
-- Success
{ ok = true }

-- Failures
{ ok = false, error = "From address invalid!" }
{ ok = false, error = "To address invalid!" }
{ ok = false, error = "From address does not exist!" }
{ ok = false, error = "To address does not exist!" }
{ ok = false, error = "Receiving enderstorage full." }
```

**Notes**

* Transfers are processed per slot range.
* Partial transfers can occur; if any item fails to fully transfer, the request returns `ok = false`.

---

# Admin

### canAddEnderstorage

```lua id="z7mf42"
{
  route = "canAddEnderstorage",
  key = "key",
  data = {
    id = "",
    frequency = {},
    name = ""
  }
}
```

**Fields**

* **id**: The user's name or UUID
* **frequency**: `{color1, color2, color3}`
* **name**: Desired name (case-insensitive, stored lowercase)

**Response**

```lua id="n3q8la"
-- Success
{ ok = true }

-- Failures
{ ok = false, error = "You already have this frequency!" }
{ ok = false, error = "You already have a box with this name!" }
```

---

### addPlacementData

```lua id="c2v9hs"
{
  route = "addPlacementData",
  key = "key",
  data = {
    ownerUUID = "",
    user = "",
    frequency = {},
    name = ""
  }
}
```

**Fields**

* **ownerUUID**: UUID of the owner
* **user**: Username of the owner
* **frequency**: `{color1, color2, color3}`
* **name**: Name of the enderstorage (stored lowercase)

**Response**

```lua id="m5k1xr"
-- Success
{ ok = true }
```

---

## General Response Format

All routes return a table in this format:

```lua id="g8v2op"
-- Success
{ ok = true, result = any? }

-- Failure
{ ok = false, error = "error message" }
```

