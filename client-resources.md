# Client resources

The client side of a resource is downloaded when a player joins, and it runs in their game. The mod has
no gamemode of its own built in, so everything a server does in a player's game comes from these files.

## What players download

Only a resource's `clientScripts`, `sharedScripts` and `files` are sent. Server scripts, `package.json`
and other server files never are.

> **Never put server code, passwords or private settings in those three lists.** Every player gets
> them.

```json
{
  "name": "hello",
  "kcds": {
    "serverScripts": ["server.js"],
    "clientScripts": ["client.js"],
    "files": ["ui/**"]
  }
}
```

<!-- only sealed -->
Every file of up to 4 MB a resource sends is sealed (AES-256-GCM) with a secret the server makes when
the resource starts. Each player's game gets the secret over its session to the server and opens the
files in memory. The download cache keeps them sealed, so a copy of the cache opens nowhere: not in
the plain game, not in another mod, and not on another server. The resource's scripts see the plain
files as before, nothing changes for them. The files a screen, a book or a stream mount uses stay
in the game's memory too: none is written to disk as a plain copy. Restarting a resource gives it a
new secret, and its files download again on the next join.

A bigger file (a video, a large model) is not sealed: it downloads as it is and stays in the
download cache on the disk, so a player's game never holds it all in memory until a page or the
game reads it. Keep what must not be copied in files of up to 4 MB.
<!-- end -->

The game caches downloads in `%LOCALAPPDATA%\KCDS\cache\resources`, named by SHA-256, up to 1 GB, and
removes the least recently used files first. Rejoining the server while the resource runs reads
from this cache. Each file is checked against its hash before use, and a damaged or changed one
downloads again. Changed files download before the new version starts.

<!-- only sealed -->
What a player's game has opened can still be read from its memory while it runs: sealing stops
copying files out of the cache and reusing them, not a determined attacker with a debugger.
<!-- end -->

## What client scripts can use

Client scripts are CommonJS and can `require` the resource's own `.js`, `.cjs` and `.json` files. They
do not have Node modules, `process` or disk access. To use an npm library, bundle it into your client
files.

The client side of the API works with this player's game. It includes the HUD and notifications
([Hud](/docs/api/hud)), key binds ([Key](/docs/api/key)), HTML pages over the game
([Web](/docs/api/web)), the game's interface ([NativeUI](/docs/api/nativeui)), books and letters
([Book](/docs/api/book)), the free camera ([NoClip](/docs/api/noclip)) and the character creator
([CharacterCreator](/docs/api/charactercreator)). See the [API overview](api.md) for more.

Both sides have the math classes, `Events`, `Messages`, `Imports`, `Exports` and timers.

## Sending from client to server

```js
// client
kcds.core.Events.on('playerReady', () => {
  kcds.core.Events.toServer('hello', { message: 'Hello from my game' });
});
```

```lua
-- client
kcds.core.Events.on("playerReady", function()
  kcds.core.Events.toServer("hello", { message = "Hello from my game" })
end)
```

```cs
// client
Events.On("playerReady", () =>
{
    Events.ToServer("hello", new { message = "Hello from my game" });
});
```

```js
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

The handler receives two things:

- `sender`: the player the connection belongs to. The server determines this, so it cannot be faked.
- `payload`: parsed JSON of up to 32 KB. It comes from the client, so check it before your rules use it.

`fromClient` handlers are kept apart from the rest. A client cannot trigger the server's game events or
its `Events.on` handlers.

## Sending from server to clients

To reach everyone:

```js
// server
kcds.core.Events.broadcast('hello:announce', { text: 'The market opens at noon.' });
```

```lua
-- server
kcds.core.Events.broadcast("hello:announce", { text = "The market opens at noon." })
```

```cs
// server
Events.Broadcast("hello:announce", new { text = "The market opens at noon." });
```

```js
// client
kcds.core.Events.on('hello:announce', (data) => {
  kcds.ui.Hud.showNotification(data.text);
});
```

```lua
-- client
kcds.core.Events.on("hello:announce", function(data)
  kcds.ui.Hud.showNotification(data.text)
end)
```

```cs
// client
Events.On("hello:announce", (JsValue data) =>
{
    Hud.ShowNotification(data["text"]);
});
```

To reach one player, use `player.emit(name, payload)`. The client hears it the same way, through
`Events.on(name)`. It returns `false` if the player has left.

```js
// server
kcds.core.Events.fromClient('shop:prices.ask', (player) => {
  player.emit('shop:prices', { bread: 3, ale: 2 });
});
```

```lua
-- server
kcds.core.Events.fromClient("shop:prices.ask", function(player)
  player:emit("shop:prices", { bread = 3, ale = 2 })
end)
```

```cs
// server
Events.FromClient("shop:prices.ask", (Player player) =>
{
    player.Emit("shop:prices", new { bread = 3, ale = 2 });
});
```

## How data is encoded

`toServer` and `broadcast` turn values into JSON. The exception is a string, which is sent unchanged
and must already be valid JSON, such as `'{"text":"hi"}'`. An event with invalid JSON is dropped, with
a warning in the log.

## Starting, stopping and restarting

`resourceStart(name)` may return a promise, and resources that depend on yours wait for it. Use
`resourceStop(name)` to clean up. A stopping resource loses its timers, handlers and module cache.

`ensure <name>` in the server console restarts a resource and its running dependents. Players switch to
the new client part without restarting their game.

## If client scripts stop by themselves

If the browser engine running the scripts ends, the game restarts them after two seconds, up to 5 times
per session, and loads all resources again as on joining. The player's log shows `runtime started
again`.

The scripts start with nothing, and `playerResourcesReady` fires again on the server. So send what a
client needs from that event, not just on the first join:

```js
// server
kcds.core.Events.on('playerResourcesReady', (player) => {
  player.emit('shop:prices', { bread: 4, beer: 2 });
});
```

```lua
-- server
kcds.core.Events.on("playerResourcesReady", function(player)
  player:emit("shop:prices", { bread = 4, beer = 2 })
end)
```

```cs
// server
Events.On("playerResourcesReady", (Player player) =>
{
    player.Emit("shop:prices", new { bread = 4, beer = 2 });
});
```

## Errors on the server

`errorBehavior` in the manifest sets what an uncaught server error does: `stop`, `restart` or
`continue`. Catch errors in your callbacks and promise chains, because `continue` does not repair game
state.
