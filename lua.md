# Lua

You can write a resource's server scripts, client scripts or both in Lua 5.4 instead of JavaScript. A Lua
script gets the same `kcds` object a JavaScript script gets, with every type, member and event, and the same
rules for starting, stopping and errors. KCDS runs Lua in its own engine (Lua compiled to WebAssembly), on
the server and in each player's game; it is not the game's own Lua.

## A Lua resource

```text
resources/
  hello-lua/
    package.json
    server.lua
    client.lua
```

`package.json`:

```json
{
  "name": "hello-lua",
  "version": "1.0.0",
  "kcds": {
    "server": "server.lua",
    "client": "client.lua"
  }
}
```

A side's language comes from its scripts' file extensions: `.lua` is Lua, `.js`, `.cjs` and `.mjs` are
JavaScript. You can also name it with `"language"`: one name for both sides, such as `"lua"`, or
`{ "server": "lua", "client": "javascript" }`. All scripts of one side are in one language, so a resource
can have a Lua server and a JavaScript client, but not Lua and JavaScript server scripts together. `sharedScripts`
run on both sides, so they are in both sides' language.

`server.lua`:

```lua
local Events, Chat = kcds.core.Events, kcds.core.Chat
local Color = kcds.math.Color

-- Runs when a player has joined and loaded the world.
Events.on("playerReady", function(player)
  Chat.sendToPlayer(player, "Welcome, " .. player.nickname .. "!", { author = "Server" })
end)

-- Runs for "/hello" or "/hello some words" in the chat.
Events.on("playerCommand", function(player, command, args)
  if command:lower() ~= "hello" then
    return
  end
  local reply = #args > 0 and table.concat(args, " ") or "Hello!"
  Chat.sendToPlayer(player, reply, { author = "Server", color = Color.fromRGB(255, 210, 77) })
end)

-- A client asks how many players are online.
Events.fromClient("hello:count", function(player)
  player:emit("hello:count", { players = #kcds.players.Player.all() })
end)
```

`client.lua`:

```lua
-- client
local Events = kcds.core.Events
local Hud, Key = kcds.ui.Hud, kcds.ui.Key

Events.on("playerReady", function()
  Hud.showNotification("The hello resource is running.")
end)

-- F7 asks the server how many players are online.
Key.bind("f7", function()
  Events.toServer("hello:count")
end)

Events.on("hello:count", function(data)
  Hud.showNotification(data.players .. " players online")
end)
```

`ensure hello-lua` in the server console starts it, as for any resource.

## Editor support

`KCDS-Types-<version>.zip` on the download page has the whole SDK as annotations for the Lua language
server (LuaLS, the "Lua" extension of VS Code and other editors), in its `lua/` folder: `kcds-server.lua`
for server scripts and `kcds-client.lua` for client scripts. They give your editor every type, member,
parameter, event name and description. Use the version matching your server.

Put the side's file in your resource's folder, such as `types/kcds-server.lua`, and a `.luarc.json` next to
`package.json`:

```json
{
  "runtime.version": "Lua 5.4",
  "workspace.library": ["types/kcds-server.lua"]
}
```

A resource with Lua on both sides keeps its client scripts in their own folder with a `.luarc.json` that
lists `kcds-client.lua`, so each side is checked against what it really has. The files only describe the
SDK and never run, so do not list them among the scripts.

## What is different from JavaScript

The API is the same; only the language's own ways differ.

| JavaScript                          | Lua                                             |
| ----------------------------------- | ----------------------------------------------- |
| `new Vector3(1, 2, 3)`              | `Vector3.new(1, 2, 3)`                          |
| `Player.all()`, `player.emit(...)`  | `Player.all()`, `player:emit(...)`              |
| `list[0]`, `list.length`            | `list[1]`, `#list`                              |
| `null`, `undefined`                 | `nil`                                           |
| `{ author: 'Server' }`              | `{ author = "Server" }`                         |
| `(player) => { ... }`               | `function(player) ... end`                      |
| `try { ... } catch (error) { ... }` | `local ok, problem = pcall(function() ... end)` |
| `require('./util')`                 | `require("util")`                               |
| `console.log(a, b)`                 | `print(a, b)`                                   |

- **Classes** have `new` for their constructor: `kcds.math.Vector3.new(3, 4, 0)`. Their static members are
  read as in JavaScript: `kcds.math.Vector3.zero()`, `kcds.players.Player.getById(7)`.
- **Members** are called with `.` or `:`; both work. Use `:` on objects such as a player (`player:emit(...)`)
  and `.` on the SDK's namespaces (`Chat.sendToAll(...)`). Properties such as `player.nickname` or
  `vector.length` are read with `.`.
- **Lists** the SDK returns are indexed from 1, with `#list`, `ipairs(list)` and `table.concat(list)`.
- **Tables** you pass become JavaScript objects, or lists when their keys are 1 to n.
- **Nothing**: a member that answers `null` or `undefined` in JavaScript answers `nil` in Lua. A `nil` you
  pass is an argument not given, so the member's default applies:
  `kcds.characters.Horse.spawn(position, nil, nil, "Pebbles")`. Where a member asks for `null` itself, such
  as `state:onChange(null, handler)` for every key, pass the value `null`.
- **JSON**: `json.encode(value)` and `json.decode(text)` are JavaScript's `JSON.stringify` and `JSON.parse`.
  An empty table encodes as `{}`.
- **Errors**: an SDK error is a Lua error, which `pcall` catches; its text is the JavaScript error's. A Lua
  error in a handler is that handler's error, as a JavaScript throw is, and an error while a file starts
  stops the resource (or restarts it, as its `errorBehavior` says).

```lua
local Vector3 = kcds.math.Vector3
local a = Vector3.new(3, 4, 0)
print(a.length, a:distance(Vector3.zero()))

local ok, problem = pcall(kcds.core.Chat.sendToAll, 5)
print(ok, problem) -- false   TypeError: Chat text must be string

for index, player in ipairs(kcds.players.Player.all()) do
  print(index, player.nickname)
end
```

## Promises

A member that answers a promise in JavaScript answers the same promise in Lua. Use `:next(fn)`,
`:catch(fn)` and `:finally(fn)` to run code when it settles:

```lua
fetch("https://example.com/status.json")
  :next(function(response)
    return response:json()
  end)
  :next(function(status)
    print("players online", status.players)
  end)
  :catch(function(problem)
    print("request failed", problem)
  end)
```

At the top level of a file, `:await()` waits for a promise and answers its value:

```lua
local response = fetch("https://example.com/status.json"):await()
print("status", response.status, response:json():await().players)
```

`:await()` does not work inside a handler (a function KCDS calls, such as an event handler or a timer),
because Lua cannot wait inside a call that came from the host. Use `:next` there, and return the promise:
a handler that returns a promise is waited for, as an async JavaScript handler is
(`kcds.core.Events.emit` settles once it does).

`Promise.create(function(resolve, reject) ... end)` makes a promise of your own, `Promise.resolve(value)`
one that is already fulfilled and `Promise.all(list)` one for a list of promises:

```lua
local function after(ms, value)
  return Promise.create(function(resolve)
    setTimeout(function() resolve(value) end, ms)
  end)
end

kcds.core.Events.on("round:end", function(winner)
  return after(100, winner):next(function(name)
    print("announced", name)
  end)
end)
```

## Modules

`require("name")` runs another Lua file of the same resource once and answers what it returns. The name is
the file's path from the resource's folder, whichever script requires it, with a dot for each folder and
without `.lua`: `require("shared.util")` is `shared/util.lua`, and a script in `client/` requires
`client/util.lua` as `require("client.util")`. A client script can require only files the server sends to
players, which by default are the files in its client scripts' folders.

```lua
-- shared/util.lua
local util = {}

function util.double(value)
  return value * 2
end

return util
```

```lua
local util = require("shared.util")
print(util.double(21))
```

## Timers, logging and the web

`setTimeout`, `setInterval`, `clearTimeout`, `clearInterval` and `queueMicrotask` are the same functions as
in JavaScript ([Timers](timers.md)), and stopping a resource cancels its timers. `print` writes to the
resource's log, with its values separated by tabs.

Server scripts also have the same `fetch`, `WebSocket`, `URL`, `TextEncoder` and the web platform's other
classes as JavaScript ([Network access](resources.md#network-access-from-server-scripts)), with `new` for
the classes: `WebSocket.new("wss://example.com/feed")`. Node's own `Buffer` and `process` are JavaScript
only.

```lua
local heartbeat = setInterval(function()
  print("still running")
end, 5000)
clearInterval(heartbeat)
```

## What Lua scripts cannot do

- Lua's `io`, `package`, `debug`, `dofile` and `loadfile` are not there, and `os` keeps only `time`, `clock`,
  `date` and `difftime`. Files, processes and the environment are out of reach. For data that must last, call a
  JavaScript resource's [exports](imports-exports.md) that save it, or a web service with `fetch`.
- `load` takes Lua source only, not compiled chunks.
- A call into Lua may run for at most 5 seconds without returning: a file's code between two `:await()`
  calls, a handler or a timer. Past that it fails with a timeout error, so an endless loop cannot stop the
  server or the game.

A resource can also be written in [C#](csharp.md), and resources in Lua, JavaScript and C# share events,
exports and messages with each other.
