luv
===

libuv bindings for luajit and lua 5.1/5.2.

This library makes libuv available to lua scripts.  It was made for the [luvit](http://luvit.io/) project but should usable from nearly any lua project.

The library can be used by multiple threads at once.  Each thread is assumed to load the library from a different `lua_State`.  Luv will create a unique `uv_loop_t` for each state.  You can't share uv handles between states/loops.

The best docs currently are the [libuv docs](http://docs.libuv.org/) themselves.  Hopfully soon we'll have a copy locally tailored for lua.

```lua
local uv = require('luv')

-- This will wait 1000ms and then continue inside the callback
uv.timer_start(uv.new_timer(), 1000, 0, function (timer)
  -- timer here is the value we passed in before from new_timer.

  print ("Awake!")

  -- You must always close your uv handles or you'll leak memory
  -- We can't depend on the GC since it doesn't know enough about libuv.
  uv.close(timer)
end)

print("Sleeping");

-- uv.run will block and wait for all events to run.
-- When there are no longer any active handles, it will return
uv.run()
```


Here is an example of an TCP echo server
```lua
local function create_server(host, port, on_connection)

  local server = uv.new_tcp()
  p(1, server)
  uv.tcp_bind(server, host, port)

  uv.listen(server, 128, function(self)
    local client = uv.new_tcp()
    uv.accept(server, client)
    on_connection(client)
  end)

  return server
end

local server = create_server("0.0.0.0", 0, function (client)
  p("new client", client, uv.tcp_getsockname(client), uv.tcp_getpeername(client))
  uv.read_start(client, function (self, err, chunk)
    p("onread", {self=self, err=err,chunk=chunk})

    -- Crash on errors
    assert(not err, err)

    if chunk then
      -- Echo anything heard
      uv.write(client, chunk)
    else
      -- When the stream ends, close the socket
      uv.close(client)
    end
  end)
end)
```

More examples can be found in the examples and tests folders.
