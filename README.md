# Nelson
A wrapper for inline analysis of traffic in Tshark/Wireshark with Lua

In your Lua script for Wireshark, load Nelson:
``` lua
local nelson = require('nelson')
```

Define a function that will handle the packet when it is dissected according to the dissector spec.:
``` lua
local function processPacket(tvb, pinfo, pmatch)
end
```

Register packet handler Lua functions with Nelson by calling `nelson.register` and passing an object to register. Each object is a Lua table that should contain information about where to find the dissector to intercept and the Lua function to intercept with.

Nelson registr object fields:
```
  dt_name: name of the table the target dissector lives in
  dt_enum: dt_name's enumeration in the next highest table in the chain
dt_values: list of values in the dissector table to intercept (e.g. TCP ports)
 analyzer: the Lua function to run for the intercepted packet
```

##Examples
``` lua
nelson.register({
	-- IP traffic
	dt_name = "ethertype",
	dt_enum = 0,
	dt_values = {2048},
	analyzer = processPacket
})
nelson.register({
	-- Arbitrary TCP ports
	dt_name = "tcp.port",
	dt_enum = 6,
	dt_values = {17337},
	analyzer = processPacket
})
nelson.register({
	-- HTTP
	dt_name = "tcp.port",
	dt_enum = 6,
	dt_values = {80, 8080, 8088},
	analyzer = processPacket
})
nelson.register({
	-- CFLOW (NetFlow)
	dt_name = "udp.port",
	dt_enum = 17,
	dt_values = {2055},
	analyzer = processPacket
})
nelson.register({
	-- WLAN Radiotap header
	dt_name = "wtap_encap",
	dt_enum = 0,
	dt_values = {23},
	analyzer = processPacket
})
```

Adding a "listener" instance to your Lua script is recommended so that Wireshark initializes Nelson and so that end-of-run activity can be done:
``` lua
local tap = Listener.new(nil, nil)
function tap.draw()
	print("done!")
end
```
