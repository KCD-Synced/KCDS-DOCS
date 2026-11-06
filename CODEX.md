# KCDS resources: instructions for Codex

Save this file as `AGENTS.md` in your resource's folder, or in the server's `resources` folder to cover all of
them. Codex reads `AGENTS.md` when it starts working there, and so do other coding assistants that follow that
convention. It then writes KCDS resources the way the SDK expects them. The same text for Claude Code is
[CLAUDE.md](https://kcdsynced.com/docs/claude).

When Codex runs without web access, put the SDK's declarations for your language into the project
(see [Look it up, never guess](#look-it-up-never-guess)) so it can search them.

Everything below comes from the KCDS documentation at <https://kcdsynced.com/docs>. When this file and the
documentation disagree, the documentation is right.

## What you are writing

KCDS adds multiplayer to Kingdom Come: Deliverance II. A server's rules, commands and content come from
resources. A resource is a folder in the server's `resources` folder whose `package.json` has a `"kcds"`
section. Its server scripts run on the server, and its client scripts are downloaded by each player and run in
their game. Nobody builds the server or the mod to write a resource.

A resource's sides can be in JavaScript (CommonJS), TypeScript compiled to CommonJS, Lua 5.4 or C# (.NET 9,
built to `.dll` files). Each side uses one language, picked from its scripts' extensions or set with
`"language"`, so a Lua server can go with a JavaScript client.

## Look it up, never guess

- Every type has a reference page at `https://kcdsynced.com/docs/api/<type>`, with the type's name in lower
  case: `/docs/api/player`, `/docs/api/npc`, `/docs/api/hud`. Read the page before you use a type. A member
  that is not on its page does not exist.
- Each reference page says whether a type works in server scripts, in client scripts or on both sides. Client
  types such as `Hud`, `Key`, `Web` and `NativeUI` are missing from server scripts.
- Use the SDK of the version the server runs. The declarations list every member with its description, so
  search them when the web is out of reach:
  - TypeScript and JavaScript: `node_modules/@kcds/types/` (`npm install --save-dev @kcds/types@<version>`)
  - Lua: `kcds-server.lua` and `kcds-client.lua` from the `lua/` folder of `KCDS-Types-<version>.zip`
  - C#: `Kcds.Sdk.dll` and `Kcds.Sdk.xml` from the `csharp/` folder of the same zip
- The guide pages are listed under [Where to read more](#where-to-read-more).

## The manifest

```json
{
  "name": "market",
  "version": "1.0.0",
  "kcds": {
    "serverScripts": ["server/main.js"],
    "clientScripts": ["client/main.js"],
    "sharedScripts": ["shared/items.js"],
    "files": ["ui/**"],
    "exports": ["priceOf"],
    "requires": ["economy"],
    "errorBehavior": "stop"
  }
}
```

- A resource is known by its `name`, not by its folder. The console command `ensure market` uses the name.
- `serverScripts`, `clientScripts` and `sharedScripts` run in the listed order. `sharedScripts` run first on
  each side. `"server": "server.js"` and `"client": "client.js"` are short for a list of one script.
- `files` holds the files that are sent to players but do not run, such as pages, pictures and data.
- `exports` lists the names a resource offers to other resources. `requires` names the resources that must
  run first, and `{ "name": "economy", "optional": true }` names one that may be missing.
- `errorBehavior` sets what an uncaught error does: `stop` (the default), `restart` or `continue`.
- A C# side lists the `.dll` files it was built into, never `.cs` sources.

## The API

The SDK is the global `kcds` object, so you need no `require` or import for it. It has six domains:
`kcds.core` (events, chat, sharing, runtime), `kcds.math` (vectors, quaternions, colours), `kcds.players`,
`kcds.characters` (NPCs, animals, dialogue, quests, shops), `kcds.world` (places, objects, world systems)
and `kcds.ui` (screens, input, camera, client only).

- Always write the full path, such as `kcds.world.Door`, or unpack it at the top of a file:
  `const { Door } = kcds.world;`. A bare `Door` is a `ReferenceError`.
- In Lua, constructors are `Vector3.new(...)`, objects use `:` (`player:emit(...)`), lists start at 1, and
  `nil` stands for `null`.
- In C#, names start with a capital letter (`player.Nickname`, `Hud.ShowNotification`), but event names stay
  as they are (`"playerReady"`). The template imports the namespaces `Kcds.Core`, `Kcds.Players`, `Kcds.World`
  and the rest. Values without a type are `JsValue`.

## The two sides talk through events

| From              | Send with                                                         | Receive with                    |
| ----------------- | ----------------------------------------------------------------- | ------------------------------- |
| client to server  | `kcds.core.Events.toServer(name, payload)`                        | `kcds.core.Events.fromClient(name, (player, payload) => ...)` on the server |
| server to one     | `player.emit(name, payload)`                                      | `kcds.core.Events.on(name, ...)` in the client |
| server to all     | `kcds.core.Events.broadcast(name, payload)`                       | `kcds.core.Events.on(name, ...)` in the client |
| within one side   | `kcds.core.Events.emit(name, ...args)`                            | `kcds.core.Events.on(name, ...)` |

- `Events.on` on the server never hears players. Events from players arrive only through `Events.fromClient`.
- The server fills in the first argument of a `fromClient` handler, so the sending player can be trusted. The
  payload cannot: a modified game can send anything, so check its type, length and range before your rules use
  it.
- A payload is JSON of at most 32 KB. A bigger one throws `Event payload too long`.
- Prefix your own event names with the resource's name, such as `market:buy`.

```js
// server
kcds.core.Events.fromClient('hello', (sender, payload) => {
  if (!payload || typeof payload.message !== 'string' || payload.message.length > 128) {
    return;
  }
  kcds.core.Chat.sendToPlayer(sender, payload.message);
});
```

```ts
// server
kcds.core.Events.fromClient('hello', (sender, payload) => {
  if (!payload || typeof payload.message !== 'string' || payload.message.length > 128) {
    return;
  }
  kcds.core.Chat.sendToPlayer(sender, payload.message);
});
```

```lua
-- server
kcds.core.Events.fromClient("hello", function(sender, payload)
  if type(payload) ~= "userdata" or type(payload.message) ~= "string" or #payload.message > 128 then
    return
  end
  kcds.core.Chat.sendToPlayer(sender, payload.message)
end)
```

```cs
// server
Events.FromClient("hello", (Player sender, JsValue payload) =>
{
    var message = payload["message"];
    if (message.Type != "string" || message.AsString()!.Length > 128)
    {
        return;
    }
    Chat.SendToPlayer(sender, message);
});
```

## Rules to follow

1. **The server decides.** Money, items, permissions and anything else that matters is checked and changed in
   server scripts. A client script asks, and the server answers.
2. **Nothing private goes to players.** Players receive `clientScripts`, `sharedScripts`, `files`, every file
   those scripts load, and everything under `stream/` and `replace/`. Keep passwords, keys and server settings in
   server files only. Keep the client code in a folder of its own, such as `client/`, apart from `server/`.
3. **Commands.** `playerCommand(player, command, args)` gets every command, so compare `command` first. A
   command arrives without its slash, and its arguments are split at spaces and keep their case.
4. **Refusing.** Events that ask before something happens, such as `doorInteract`, `playerLoot` or
   `playerConsuming`, are refused when a handler returns `false`, but not when it returns a promise of `false`.
   The [Events](https://kcdsynced.com/docs/events) page lists them.
5. **Start and stop.** `resourceStart(name)` and `resourceStop(name)` fire for every resource, so compare
   `name` with your own. KCDS waits for async handlers, up to 30 seconds at the start and 10 at the stop.
   Handlers, timers, exports and anything the resource spawned are removed when it stops, so no clean-up code
   is needed for them.
6. **Client restarts.** A player's client scripts can start again from nothing, and the server then gets
   `playerResourcesReady` again. Send what a client needs from that event, not only on the first join.
7. **Saving.** Nothing in memory survives a restart of the resource or the server. In JavaScript, save under
   `kcds.core.Runtime.serverDir`, because the resource's own folder may be replaced. Lua and C# scripts have no
   files: call the exports of a JavaScript resource that saves, or a web service.
8. **Errors.** An uncaught error stops the resource by default. Catch errors where you can handle them,
   especially around `await`. SDK functions throw `TypeError` for a wrong type and `RangeError` for a value out
   of range, naming the argument.
9. **Between resources.** Exports and imports share values and functions on the same side
   ([Imports and exports](https://kcdsynced.com/docs/imports-exports)). `kcds.core.Messages` calls another
   resource by name and may wait for its reply ([Messages](https://kcdsynced.com/docs/sharing)).
   `entity.state` carries values the server sets on players, NPCs, horses and dogs, with the scopes `owner` and
   `server` for values others must not see ([Shared state](https://kcdsynced.com/docs/state)).
10. **What a script can reach.** Server scripts in JavaScript have Node's modules, packages from the
    resource's `node_modules`, `fetch` and `WebSocket`. Lua server scripts have `fetch` and `WebSocket`, and C#
    server scripts have `Network.Fetch`. Client scripts can load only the resource's own `.js`, `.cjs` and
    `.json` files (in Lua, its own `.lua` files): no Node modules, no `process` and no disk.
11. **Time limits.** A call into Lua fails after 5 seconds without returning. C# has no limit, so an endless
    loop holds up every script of its side. Keep handlers short and use timers or `await` to wait.

## A small resource in each language

`package.json` for JavaScript (for Lua, name the scripts `server.lua` and `client.lua`):

```json
{
  "name": "hello",
  "version": "1.0.0",
  "kcds": {
    "serverScripts": ["server.js"],
    "clientScripts": ["client.js"]
  }
}
```

The server script welcomes players and answers `/hello`:

```js
const { Events, Chat } = kcds.core;
const { Color } = kcds.math;

// Runs when a player has joined and loaded the world.
Events.on('playerReady', (player) => {
  Chat.sendToPlayer(player, `Welcome, ${player.nickname}!`, { author: 'Server' });
});

// Runs for "/hello" or "/hello some words" in the chat.
Events.on('playerCommand', (player, command, args) => {
  if (command.toLowerCase() !== 'hello') {
    return;
  }
  const reply = args.length > 0 ? args.join(' ') : 'Hello!';
  Chat.sendToPlayer(player, reply, {
    author: 'Server',
    color: Color.fromRGB(255, 210, 77),
  });
});
```

```ts
const { Events, Chat } = kcds.core;
const { Color } = kcds.math;

// Runs when a player has joined and loaded the world.
Events.on('playerReady', (player) => {
  Chat.sendToPlayer(player, `Welcome, ${player.nickname}!`, { author: 'Server' });
});

// Runs for "/hello" or "/hello some words" in the chat.
Events.on('playerCommand', (player, command, args) => {
  if (command.toLowerCase() !== 'hello') {
    return;
  }
  const reply = args.length > 0 ? args.join(' ') : 'Hello!';
  Chat.sendToPlayer(player, reply, {
    author: 'Server',
    color: Color.fromRGB(255, 210, 77),
  });
});
```

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
  Chat.sendToPlayer(player, reply, {
    author = "Server",
    color = Color.fromRGB(255, 210, 77),
  })
end)
```

```cs
// Runs when a player has joined and loaded the world.
Events.On("playerReady", (Player player) =>
{
    Chat.SendToPlayer(player, $"Welcome, {player.Nickname}!", new { author = "Server" });
});

// Runs for "/hello" or "/hello some words" in the chat.
Events.On("playerCommand", (Player player, string command, string[] args) =>
{
    if (command.ToLowerInvariant() != "hello")
    {
        return;
    }
    var reply = args.Length > 0 ? string.Join(" ", args) : "Hello!";
    Chat.SendToPlayer(player, reply, new
    {
        author = "Server",
        color = Color.FromRGB(255, 210, 77),
    });
});
```

The client script asks the server for the player count when F7 is pressed:

```js
// client
const { Events } = kcds.core;
const { Hud, Key } = kcds.ui;

Events.on('playerReady', () => {
  Hud.showNotification('The hello resource is running.');
});

// F7 asks the server how many players are online.
Key.bind('f7', () => {
  Events.toServer('hello:count');
});

Events.on('hello:count', (data) => {
  Hud.showNotification(`${data.players} players online`);
});
```

```ts
// client
const { Events } = kcds.core;
const { Hud, Key } = kcds.ui;

Events.on('playerReady', () => {
  Hud.showNotification('The hello resource is running.');
});

// F7 asks the server how many players are online.
Key.bind('f7', () => {
  Events.toServer('hello:count');
});

Events.on('hello:count', (data) => {
  Hud.showNotification(`${data.players} players online`);
});
```

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

```cs
// client
Events.On("playerReady", () =>
{
    Hud.ShowNotification("The hello resource is running.");
});

// F7 asks the server how many players are online.
Key.Bind("f7", new Action(() =>
{
    Events.ToServer("hello:count");
}));

Events.On("hello:count", (JsValue data) =>
{
    Hud.ShowNotification($"{data["players"]} players online");
});
```

And the server answers it:

```js
kcds.core.Events.fromClient('hello:count', (player) => {
  player.emit('hello:count', { players: kcds.players.Player.all().length });
});
```

```ts
kcds.core.Events.fromClient('hello:count', (player) => {
  player.emit('hello:count', { players: kcds.players.Player.all().length });
});
```

```lua
kcds.core.Events.fromClient("hello:count", function(player)
  player:emit("hello:count", { players = #kcds.players.Player.all() })
end)
```

```cs
Events.FromClient("hello:count", (Player player) =>
{
    player.Emit("hello:count", new { players = Player.All().Length });
});
```

A C# resource starts from `csharp/template` in `KCDS-Types-<version>.zip`. Each script there is a public
class deriving from `Script`, and its code goes in `OnStart`. Build each side with `dotnet build -c Release`,
and list `bin/server/Server.dll` and `bin/client/Client.dll` in the manifest. See
[C#](https://kcdsynced.com/docs/csharp).

A TypeScript resource compiles each side with its own `tsconfig`, one with `"types": ["@kcds/types/server"]`
and one with `"types": ["@kcds/types/client"]`, to CommonJS in `dist/`, and the manifest lists the compiled
files. See [TypeScript](https://kcdsynced.com/docs/typescript).

## Testing a change

1. Save the files, and build them for TypeScript or C#.
2. Enter `ensure <name>` in the server console, or press **Restart** on the Resources tab of KCDS-Admin. The
   resource starts again with its latest files, and online players get the new client part without
   restarting their game.
3. Server output is in the server console and the Console tab of KCDS-Admin. Client output is in the
   player's game log, `client-<process id>.log` in the KCDS folder (`%LOCALAPPDATA%\Programs\KCDS` with the
   installer).

`console.log` joins its values as text, so log `JSON.stringify(value)` (in Lua, `json.encode(value)`) to see
an object.

| Symptom                                               | Likely cause                                                   |
| ----------------------------------------------------- | -------------------------------------------------------------- |
| `ensure hello` does nothing, not even `started`       | the console needs the `name` from `package.json`, not the folder name |
| `ReferenceError: Door is not defined`                 | the type needs its full path, `kcds.world.Door`                |
| `Cannot read properties of undefined` on `kcds.ui.Hud` | a client type used in a server script                         |
| a client event never reaches the server               | the server used `Events.on`, but events from players need `Events.fromClient` |
| `Events.toServer` works but no answer appears         | answer with `player.emit` or `Events.broadcast`, and listen with `Events.on` in the client |
| a command handler runs for every command              | `playerCommand` gets all commands, so compare `command` first  |

## Where to read more

- Start: [Your first resource](https://kcdsynced.com/docs/first-resource),
  [Project structure](https://kcdsynced.com/docs/project-structure),
  [Resources](https://kcdsynced.com/docs/resources), [Client resources](https://kcdsynced.com/docs/client-resources),
  [API overview](https://kcdsynced.com/docs/api), [API changes](https://kcdsynced.com/docs/api-changes)
- Languages: [TypeScript](https://kcdsynced.com/docs/typescript), [Lua](https://kcdsynced.com/docs/lua),
  [C#](https://kcdsynced.com/docs/csharp)
- Scripting: [Events](https://kcdsynced.com/docs/events), [Imports and exports](https://kcdsynced.com/docs/imports-exports),
  [Messages](https://kcdsynced.com/docs/sharing), [Shared state](https://kcdsynced.com/docs/state),
  [Chat and texts](https://kcdsynced.com/docs/chat-locales), [Timers and runtime](https://kcdsynced.com/docs/timers),
  [Math](https://kcdsynced.com/docs/math)
- Players: [Players](https://kcdsynced.com/docs/players), [Restricting a player](https://kcdsynced.com/docs/player-restrictions),
  [Skills and perks](https://kcdsynced.com/docs/progression), [Custom buffs](https://kcdsynced.com/docs/custom-buffs),
  [Animations](https://kcdsynced.com/docs/animations), [Voice chat](https://kcdsynced.com/docs/voice)
- World and objects: [NPCs](https://kcdsynced.com/docs/npcs), [Horses and dogs](https://kcdsynced.com/docs/horses-dogs),
  [Time and weather](https://kcdsynced.com/docs/world), [Areas](https://kcdsynced.com/docs/areas),
  [Markers and blips](https://kcdsynced.com/docs/markers), [Props](https://kcdsynced.com/docs/props),
  [Doors and gates](https://kcdsynced.com/docs/doors), [Stashes](https://kcdsynced.com/docs/stashes)
- Gameplay: [Inventory and items](https://kcdsynced.com/docs/inventory), [Equipment](https://kcdsynced.com/docs/equipment),
  [Quests and conversations](https://kcdsynced.com/docs/quests-dialogue), [Shops and trade](https://kcdsynced.com/docs/vendors),
  [Crafting](https://kcdsynced.com/docs/crafting)
- Client: [HUD and messages](https://kcdsynced.com/docs/client-ui), [Action hints](https://kcdsynced.com/docs/action-hints),
  [Web views](https://kcdsynced.com/docs/web-views), [Keys and controls](https://kcdsynced.com/docs/input),
  [Camera](https://kcdsynced.com/docs/client-camera), [Sounds and game files](https://kcdsynced.com/docs/audio-and-assets)
- Trying and fixing: [Trying code quickly](https://kcdsynced.com/docs/playground),
  [Debugging resources](https://kcdsynced.com/docs/debugging)
