# Resources

A resource is a folder in the server's `resources` folder with a `package.json`. Its scripts define your
server's commands, gamemodes, shops and anything else. You do not need to build the server or the mod, and
players download a resource's client part when they join.

For a step-by-step start, see [Your first resource](first-resource.md). The smallest working resource is two
files:

```json
{
  "name": "hello",
  "version": "1.0.0",
  "kcds": {
    "serverScripts": ["server.js"]
  }
}
```

```js
const { Events, Chat } = kcds.core;

Events.on('playerCommand', (player, command, args) => {
  if (command.toLowerCase() !== 'hello') {
    return;
  }
  Chat.sendToPlayer(player, args.length > 0 ? args.join(' ') : 'Hello!', { author: 'Server' });
});
```

```ts
const { Events, Chat } = kcds.core;

Events.on('playerCommand', (player, command, args) => {
  if (command.toLowerCase() !== 'hello') {
    return;
  }
  Chat.sendToPlayer(player, args.length > 0 ? args.join(' ') : 'Hello!', { author: 'Server' });
});
```

```lua
local Events, Chat = kcds.core.Events, kcds.core.Chat

Events.on("playerCommand", function(player, command, args)
  if command:lower() ~= "hello" then
    return
  end
  Chat.sendToPlayer(player, #args > 0 and table.concat(args, " ") or "Hello!", { author = "Server" })
end)
```

```cs
Events.On("playerCommand", (Player player, string command, string[] args) =>
{
    if (command.ToLowerInvariant() != "hello")
    {
        return;
    }
    Chat.SendToPlayer(player, args.Length > 0 ? string.Join(" ", args) : "Hello!", new { author = "Server" });
});
```

The API is in the global `kcds` object and needs no import; start from the [API overview](api.md). Commands
arrive without the slash, with arguments split at spaces. Case is kept, so do your own case-insensitive
comparison.

## TypeScript

You can write in TypeScript if you compile it to CommonJS first and point the manifest at the compiled files.
Each release includes the SDK's declarations in `KCDS-Types-<version>.zip`. See [TypeScript](typescript.md) for
a project setup.

<!-- only lua cs -->
## Lua and C#

You can also write either side in Lua 5.4 or in C#, with the same API. See [Lua](lua.md) and [C#](csharp.md).
A C# side lists the `.dll` files its scripts were built into.
<!-- end -->

## The manifest

KCDS reads the `"kcds"` section of `package.json`. A resource is identified by its `name`, not its folder.
`"server": "server.js"` and `"client": "client.js"` are short for a list of one script.

| Key                     | Default  | Meaning                                                                                     |
| ----------------------- | -------- | ------------------------------------------------------------------------------------------- |
| `serverScripts`         | `[]`     | server scripts (CommonJS, Lua, or C# `.dll`s), in the order they run                        | <!-- only lua cs -->
| `serverScripts`         | `[]`     | server scripts (CommonJS), in the order they run                                            | <!-- without lua cs -->
| `language`              | by file  | `"javascript"`, `"lua"` or `"csharp"` for both sides, or `{ "server": ..., "client": ... }` | <!-- only lua cs -->
| `clientScripts`         | `[]`     | scripts each player's game runs; see [Client resources](client-resources.md)                |
| `sharedScripts`         | `[]`     | scripts for both sides, run before that side's own list                                     |
| `files` (`clientFiles`) | `[]`     | files sent to players but not run, such as pages, pictures and data                         |
| `loadingScreen`         | none     | your page over joining and loading, sent first; see [Web views](web-views.md)               |
| `exports`               | `[]`     | names shared with other resources; see [Imports and exports](imports-exports.md)            |
| `requires`              | `[]`     | resources that must run first: names, or `{ "name": "x", "optional": true }`                |
| `errorBehavior`         | `"stop"` | what an error at start, in a timer or in a promise nothing handles does: `stop`, `restart` (after one second) or `continue`; one thrown in an event handler is only logged ([Debugging](debugging.md#what-errors-do)) |
| `priority`              | `0`      | recorded, but does not change the start order                                               |

### File patterns

`files` takes relative paths with `/`, and patterns can use `*`, `?`, `**`, `[a-z]` and `*.{css,json}`.

If `files` is empty, the folders of the client entry scripts are sent. With `client.js` next to `package.json`,
that is every file except the server entry scripts. Some files are never sent that way:

- hidden files such as `.env` and `.git`, and `node_modules`;
- the folders of the server entry scripts, such as `server/` for `server/main.js`, unless a client entry
  script is in the same folder;
- the server's data and keys: `package.json`, `package-lock.json`, `server.json`, and files ending in `.sql`,
  `.db`, `.sqlite`, `.sqlite3`, `.pem`, `.key`, `.p12`, `.pfx` or `.log`.

A `files` pattern that names one of them still sends it.

A pattern matches hidden files only if it names the dot, but a broad one like `**` does include `node_modules`.
Symbolic links are not followed.

> **Keep keys, settings and server modules out of client folders and broad patterns.** The simplest way is a
> separate client folder; see [Project structure](project-structure.md).

### Size limits

What a server sends to players, all its resources together:

| Limit                                  | Value  |
| -------------------------------------- | ------ |
| Resources                              | 5120   |
| Files                                  | 81920  |
| One file                               | 512 MB |
| All files                              | 4 GB   |
| A loading page and what it loads first | 32 MB  |

The files download over HTTP on TCP with the server's game port ([Running a server](server.md)). Players
keep them on disk by their hash, so a second join reads them from the disk. Files over 4 MB stay there and
are read from the disk when the game needs them. Scripts and the loading page's files are always held in
memory.

Only a connected player's game can download the files: each one gets its own token with the catalogue, and
the token stops working when the player leaves. Any other request gets `403 Not a connected player`.

Without TCP on the game port, the files come over the game connection, one file up to 32 MB. A resource with
a larger file then cannot load, and the player's log says `A resource file this large needs the server's
HTTP files`.

A player's game hands a resource only files from the current server's catalogue. A script cannot read
files another server left in the download cache.

A network event (`Events.toServer`, `player.emit`, `Events.broadcast`) carries up to 32 KB of JSON; a
bigger one throws `Event payload too long`. Split big data into several events or ask for it in parts.

A resource over a limit is not sent to players. The server logs
`<name>: not sent to players: ...`, its server scripts still run, and everything else carries on. If a refresh
pushes a resource over a limit, players keep the version they have.

### Limits

What the server holds each resource to while it runs, so one resource cannot hold up the others:

| What                                     | Limit                                                                                      |
| ---------------------------------------- | ------------------------------------------------------------------------------------------ |
| `fetch`                                  | 16 requests in flight and 256 waiting per resource; 30 s for an answer; 16 MiB of body     |
| `WebSocket`                              | 64 open sockets per resource; a further one is refused                                     |
| `Http` routes                            | 64 routes per resource; 10 s for an answer; 20 requests a second per address, bursts of 40 |
| `Messages.request`                       | 10 000 waiting in all, 1 000 per resource                                                  |
| `Inventory.beforeCommit`                 | 8 actions of one player waiting, then the next is refused as `busy`; a reason up to 128 characters |
| `player.emit` before client scripts run  | 512 events held per player; the oldest goes first                                          |
| Chat text                                | 2 048 bytes sent to a player, 1 024 bytes from a player                                    |
| Run time                                 | a call into Lua fails after 5 s; JavaScript and C# have none, so an endless loop holds up every script of its side |

## What scripts can load

Both sides can `require` the resource's own `.js`, `.cjs` and `.json` files by relative path. Client scripts can
load nothing else.

Server scripts can also load:

- Node's modules, such as `node:fs` and `node:path`
- packages in the resource's folder, after `npm install pg` there
- all of the above through `globalThis.require`, with relative paths from the resource's folder, which compiled
  code often uses

`process.cwd()` is the resource's folder, and a deployment replaces it. Save data under
`kcds.core.Runtime.serverDir` instead, the server's folder containing `resources/` and `server.json`.

## Network access from server scripts

Server scripts have `fetch` and `WebSocket` for things like a chat bot or a link to your services. When the
resource stops, its requests are aborted and its sockets closed. A resource can have up to 64 sockets open.

```js
const socket = new WebSocket('wss://example.com/bot');

socket.addEventListener('open', () => socket.send(JSON.stringify({ hello: 'kcds' })));
socket.addEventListener('message', (event) => console.info(`bot says ${event.data}`));
```

```ts
const socket = new WebSocket('wss://example.com/bot');

socket.addEventListener('open', () => socket.send(JSON.stringify({ hello: 'kcds' })));
socket.addEventListener('message', (event: { data: string }) => console.info(`bot says ${event.data}`));
```

```lua
local socket = WebSocket.new("wss://example.com/bot")

socket:addEventListener("open", function() socket:send(json.encode({ hello = "kcds" })) end)
socket:addEventListener("message", function(event) print("bot says " .. event.data) end)
```

```cs
// C# server scripts have Network.Fetch and no sockets.
var response = await Network.Fetch("https://example.com/bot/status");
var status = await Network.Json(response);
Log.Info("bot says " + status["text"]);
```

`fetch` is the standard one of browsers and Node.js: it answers a `Response`, with `json()`, `text()` and the
status, and takes a method, headers and a body.

```js
const response = await fetch('https://example.com/api/report', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ online: kcds.players.Player.all().length }),
});
if (!response.ok) {
  console.warn(`report refused: ${response.status}`);
}
```

```ts
const response = await fetch('https://example.com/api/report', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ online: kcds.players.Player.all().length }),
});
if (!response.ok) {
  console.warn(`report refused: ${response.status}`);
}
```

```lua
fetch("https://example.com/api/report", {
  method = "POST",
  headers = { ["Content-Type"] = "application/json" },
  body = json.encode({ online = #kcds.players.Player.all() }),
}):next(function(response)
  if not response.ok then
    print("report refused: " .. response.status)
  end
end)
```

```cs
var response = await Network.Fetch("https://example.com/api/report", new
{
    method = "POST",
    headers = new Dictionary<string, string> { ["Content-Type"] = "application/json" },
    body = "{\"online\": " + Player.All().Length + "}",
});
Log.Info($"report answered {response["status"]}");
```

The other standard Web APIs a server script has, as in browsers and Node.js:

| For                  | APIs                                                                                   |
| -------------------- | -------------------------------------------------------------------------------------- |
| HTTP                 | [fetch](/docs/api/fetch), [Request](/docs/api/request), [Response](/docs/api/response), [Headers](/docs/api/headers), [FormData](/docs/api/formdata) |
| Sockets              | [WebSocket](/docs/api/websocket), [MessageEvent](/docs/api/messageevent), [CloseEvent](/docs/api/closeevent) |
| Addresses            | [URL](/docs/api/url), [URLSearchParams](/docs/api/urlsearchparams)                     |
| Cancelling           | [AbortController](/docs/api/abortcontroller), [AbortSignal](/docs/api/abortsignal)     |
| Data                 | [Blob](/docs/api/blob), [File](/docs/api/file), [TextEncoder](/docs/api/textencoder), [TextDecoder](/docs/api/textdecoder), [atob](/docs/api/atob), [btoa](/docs/api/btoa), [structuredClone](/docs/api/structuredclone) |
| Streams              | [ReadableStream](/docs/api/readablestream), [WritableStream](/docs/api/writablestream), [TransformStream](/docs/api/transformstream) |
| Events and errors    | [EventTarget](/docs/api/eventtarget), [DOMException](/docs/api/domexception)          |
| Randomness and time  | [crypto](/docs/api/crypto) (random values, UUIDs, Web Crypto), [performance](/docs/api/performance) (`performance.now()`) |

## Starting and stopping

Once a resource's scripts have run, all resources receive `resourceStart(name)`, so compare the name with your
own.

KCDS waits for async handlers. Until the handler finishes, the resource is still starting and resources that require
it wait. If it rejects or takes over 30 seconds, startup fails, and KCDS removes everything the resource
registered.

When a resource stops, all resources first receive `resourceStop(name)`, while the stopping one still works fully.
Its dependents stop before it. Async handlers get up to 10 seconds. Clean-up of handlers, timers, exports,
messages and anything it spawned always happens.

```js
const { Events } = kcds.core;

Events.on('resourceStart', async (name) => {
  if (name === 'hello') {
    console.info('hello is ready');
  }
});

Events.on('resourceStop', (name) => {
  if (name === 'hello') {
    console.info('saving before hello stops');
  }
});
```

```ts
const { Events } = kcds.core;

Events.on('resourceStart', async (name) => {
  if (name === 'hello') {
    console.info('hello is ready');
  }
});

Events.on('resourceStop', (name) => {
  if (name === 'hello') {
    console.info('saving before hello stops');
  }
});
```

```lua
local Events = kcds.core.Events

Events.on("resourceStart", function(name)
  if name == "hello" then
    print("hello is ready")
  end
end)

Events.on("resourceStop", function(name)
  if name == "hello" then
    print("saving before hello stops")
  end
end)
```

```cs
Events.On("resourceStart", (string name) =>
{
    if (name == "hello")
    {
        Log.Info("hello is ready");
    }
});

Events.On("resourceStop", (string name) =>
{
    if (name == "hello")
    {
        Log.Info("saving before hello stops");
    }
});
```

<!-- only examples -->
## Complete examples

The server comes with `resources/freeroam` and `resources/dice`, which show working server and client scripts.
<!-- end -->
