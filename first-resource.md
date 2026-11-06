# Your first resource

In about ten minutes, you will make a resource called `hello` on a server you run yourself ([Running a
server](server.md)). It welcomes players, answers a chat command, and shows a player count when a key
is pressed.

## 1. Make the folder

A folder in the server's `resources` folder is a resource when its `package.json` has a `"kcds"`
section. Create these files:

```text
resources/
  hello/
    package.json
    server.js
    client.js
```

## 2. Write the manifest

`package.json` gives the resource its name and lists its scripts. The name comes from `name`, not
from the folder.

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

`serverScripts` run only on the server. `clientScripts` are sent to each player and run in their game.
[Resources](resources.md#the-manifest) lists all the keys.

<!-- only lua cs -->
Every example on these pages also has a Lua and a C# tab. To write the resource in Lua, name the scripts
`server.lua` and `client.lua` instead, here and in the manifest; [Lua](lua.md) has the rest. A C# resource
starts from the template of [C#](csharp.md) and lists the `.dll` files it builds.
<!-- end -->

## 3. Add the server script

Put this in `server.js`:

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

The global `kcds` object holds the API, so you need no `require`. A command arrives without its slash.
Its arguments are split at spaces and keep their upper and lower case.

## 4. Run it

Start the server, or if it is already running, enter this in its console:

```text
ensure hello
```

`ensure` and `restart` start the resource again with its latest files. Players who are online receive
the new client part without restarting their game. The console confirms it:

```text
RESOURCE [hello] started
```

Join the server and type `/hello Good morning` in the chat.

## 5. Add the client script

`client.js` runs in each player's game and can use things only the game has, such as the HUD, keys and
camera ([API overview](api.md)).

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

## 6. Connect the two sides

The client sends `hello:count` to the server. Add the answer to `server.js`:

```js
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

The first argument of a `fromClient` handler is the sending player. The server fills it in, so a
client cannot pretend to be someone else. A payload is JSON of at most 32 KB. A modified game can send
anything, so check a payload before you use it.

Enter `ensure hello` again and press F7 in the game. The count appears as a notification.

## How the two sides talk

| File        | Runs                    | Sends with                                                         |
| ----------- | ----------------------- | ------------------------------------------------------------------ |
| `server.js` | once, on the server     | `player.emit(name, data)` for one player, `kcds.core.Events.broadcast(name, data)` for all |
| `client.js` | in each player's game   | `kcds.core.Events.toServer(name, data)`                            |

## Where to go next

- Track down a mistake: [Debugging resources](debugging.md)
- Share values with players and other resources: [Shared state](state.md),
  [Imports and exports](imports-exports.md)
- Use TypeScript: [Resources](resources.md#typescript)
- Read complete examples: `resources/freeroam` and `resources/dice` come with the server <!-- only examples -->
