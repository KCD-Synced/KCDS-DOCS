# Project structure

Once a resource outgrows a single file, this layout keeps server code, client code and assets apart.

## Suggested layout

```text
resources/
  market/
    package.json
    server/
      main.js          server entry script
      prices.js        server module, loaded with require('./prices')
      config.json      server settings, never sent to players
    client/
      main.js          client entry script
      hud.js           client module
    shared/
      items.js         code that runs on both sides
    ui/
      shop.html        a page for Web.open
      shop.css
    stream/
      objects/kcds/market/stall.cgf   a model the game loads
    node_modules/      packages for server scripts (run npm install inside market/)
```

Its `package.json`:

```json
{
  "name": "market",
  "version": "1.2.0",
  "dependencies": {
    "pg": "^8.13.0"
  },
  "kcds": {
    "serverScripts": ["server/main.js"],
    "clientScripts": ["client/main.js"],
    "sharedScripts": ["shared/items.js"],
    "files": ["client/**", "ui/**"],
    "exports": ["priceOf"]
  }
}
```

For files under `stream/`, see [Sounds and game files](audio-and-assets.md).

## Choosing what players download

Players receive `clientScripts`, `sharedScripts` and every file those scripts load, whatever
`files` matches, and everything under `stream/` and `replace/`. They never receive server code:
`serverScripts` and every file a server script loads (followed import by import: `require` and
`import` in JavaScript, `require("folder.name")` in Lua) stay on the server unless a client script <!-- only lua -->
`import`) stay on the server unless a client script <!-- without lua -->
loads the same file. A `files` entry that names server code exactly is refused when the resource
starts, with the file named:

```text
files names server code, which is never sent to players: server/prices.js (a server script loads it and no client script does)
```

A wider pattern such as `**` just leaves server code out. `package.json` is never sent either.

**What a client folder sends by default.** Without a `files` list, each folder holding a client
entry script sends its pages, pictures, sounds and other assets, but its code and data
(`.js`, `.cjs`, `.mjs`, `.ts`, `.lua`, `.json`, `.dll`, `.cs`, `.pdb`, `.env`, `.ini`, `.yaml`, <!-- only lua cs -->
(`.js`, `.cjs`, `.mjs`, `.ts`, `.json`, `.env`, `.ini`, `.yaml`, <!-- without lua cs -->
`.toml`, `.sql`, `.db`, keys) only when a client script loads them. A C# client assembly's folder <!-- only cs -->
also sends the assemblies built beside it, its packages; never its sources or `.pdb` symbols. A <!-- only cs -->
`.toml`, `.sql`, `.db`, keys) only when a client script loads them. A <!-- without cs -->
server file loaded by a path built at run time is not followed, so it is not sent either. Keeping
client and server in separate folders still keeps things clear.

**A `files` list is exact.** Patterns can use `*`, `?`, `**`, `[a-z]` and `{css,json}`. Hidden files
match only when a pattern names the dot.

**Watch out for broad patterns.** By default, KCDS does not send `.env`, `.git`, `node_modules`, the server
scripts' folders, or data and key files such as `*.sqlite`, `*.key` and `package.json` (the full list is in
[Resources](resources.md#file-patterns)). A pattern such as `**` still picks up `node_modules`.

Downloaded files are kept and checked by hash, so unchanged files do not download again; see [Client
resources](client-resources.md).

## How the parts connect

| From          | To                    | How                                                                                        |
| ------------- | --------------------- | ------------------------------------------------------------------------------------------ |
| any script    | its own module        | `require('./prices')`, with `.js`, `.cjs` or `.json`, relative to the file                 |
| server script | Node and npm packages | `require('node:fs')`, or `require('pg')` from the resource's `node_modules`                |
| server script | its own files         | `process.cwd()` is the resource folder                                                     |
| client        | server                | `kcds.core.Events.toServer(name, data)`; answer with `player.emit` ([Events](events.md))   |
| one resource  | another               | exports and imports ([Imports and exports](imports-exports.md)), or [Messages](sharing.md) |
| web page      | its client script     | `window.sendToGame` and `kcds.ui.Web.on` ([Web views](web-views.md))                       |

```js
// server/main.js
const prices = require('./prices');
const settings = require('./config.json');

kcds.core.Events.on('resourceStart', (name) => {
  if (name === 'market') {
    console.info(`market opens with ${Object.keys(prices).length} prices`);
  }
});
```

```lua
-- server/main.lua: Lua requires .lua files by their path from the resource's folder, so the
-- settings are a server/config.lua that returns a table.
local prices = require("server.prices")
local settings = require("server.config")

kcds.core.Events.on("resourceStart", function(name)
  if name == "market" then
    local count = 0
    for _ in pairs(prices) do
      count = count + 1
    end
    print("market opens with " .. count .. " prices")
  end
end)
```

```cs
// server/Main.cs: every .cs file of the project is built into the same assembly, so a class of
// another file (server/Prices.cs below) is used as it is, with no require.
public class Main : Script
{
    public override void OnStart()
    {
        Events.On("resourceStart", (string name) =>
        {
            if (name == "market")
            {
                Log.Info($"market opens with {Prices.All.Count} prices");
            }
        });
    }
}

// server/Prices.cs
public static class Prices
{
    public static readonly Dictionary<string, double> All = new() { ["bread"] = 3, ["ale"] = 2 };
}
```

## Start order

- On each side, `sharedScripts` run first, then that side's scripts, in the listed order.
- Resources in `requires` start first. A dependent resource stops before the one it depends on. Use
  `{ "name": "economy", "optional": true }` for one that may be missing.
- `resourceStart` handlers can be async, and dependents start once they finish.

## Saving data

Nothing held in memory survives a restart of the resource or the server, so anything important belongs
in a file or database.

Save under the server's folder, because the resource's own folder may be replaced with a new copy:

```js
// server/main.js
const fs = require('node:fs');
const path = require('node:path');

const ledgerFile = path.join(kcds.core.Runtime.serverDir, 'data', 'market', 'balances.json');
const balances = fs.existsSync(ledgerFile) ? JSON.parse(fs.readFileSync(ledgerFile, 'utf8')) : {};

kcds.core.Events.on('resourceStop', (name) => {
  if (name !== 'market') {
    return;
  }
  fs.mkdirSync(path.dirname(ledgerFile), { recursive: true });
  fs.writeFileSync(ledgerFile, JSON.stringify(balances));
});
```

```lua
-- server/main.lua
-- Lua scripts have no files, so a JavaScript resource of yours, "storage" (listed under
-- requires), keeps the data: its exports save(name, value) and load(name) write and read it as
-- the JavaScript example does.
local Events, Exports = kcds.core.Events, kcds.core.Exports

local balances = Exports.get("storage", "load")("market") or {}

Events.on("resourceStop", function(name)
  if name ~= "market" then
    return
  end
  Exports.get("storage", "save")("market", balances)
end)
```

```cs
// server/Main.cs
// C# scripts have no files, so a JavaScript resource of yours, "storage" (listed under
// requires), keeps the data: its exports save(name, value) and load(name) write and read it as
// the JavaScript example does.
public class Main : Script
{
    private JsValue balances;

    public override void OnStart()
    {
        balances = Exports.Get("storage", "load").Invoke("market");

        Events.On("resourceStop", (string name) =>
        {
            if (name != "market")
            {
                return;
            }
            Exports.Get("storage", "save").Invoke("market", balances);
        });
    }
}
```

Backups from KCDS-Admin copy the resources folder with its data, without `node_modules`; see [Running a
server](server.md#kcds-admin).

## Next steps

- [TypeScript](typescript.md): the same layout with a build step
- [Trying code quickly](playground.md): a local server for experiments
