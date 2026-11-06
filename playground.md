# Trying code quickly

The fastest way to learn the SDK is a private server of your own with one throwaway resource.
Save a file, reload the resource, and see the result in your game moments later.

## 1. Start a private server

The server runs on Linux or in Docker ([Running a server](server.md)): a Linux machine on your
network or a small rented one is enough. This `server.json` keeps it off the public list:

```json
{
  "name": "My test server",
  "bind": "0.0.0.0",
  "master": "off"
}
```

Run `./start-server.sh`, open KCDS and use **Direct connect** in the server browser to join the
server's address, such as `192.168.1.20:7777`. KCDS Master must still be reachable, because your game
proves it is a legal copy on this server as on any other.

## 2. Make a scratch resource

Create `resources/scratch/package.json`:

```json
{
  "name": "scratch",
  "kcds": {
    "serverScripts": ["server.js"],
    "clientScripts": ["client/main.js"]
  }
}
```

Add a `js` console command in `resources/scratch/server.js`. It evaluates one line with the full
server API and prints the value:

```js
kcds.core.Events.on('consoleCommand', (command, words) => {
  if (command !== 'js') {
    return;
  }
  try {
    const value = eval(words.join(' '));
    console.info(typeof value === 'object' ? JSON.stringify(value) : String(value));
  } catch (problem) {
    console.warn(problem.message);
  }
});
```

```lua
-- "lua <code>" in the server console runs that Lua and prints its answer.
kcds.core.Events.on("consoleCommand", function(command, words)
  if command ~= "lua" then
    return
  end
  local chunk, problem = load("return " .. table.concat(words, " "))
  if not chunk then
    chunk, problem = load(table.concat(words, " "))
  end
  if not chunk then
    print(problem)
    return
  end
  local ok, value = pcall(chunk)
  print(ok and (type(value) == "userdata" and json.encode(value) or tostring(value)) or tostring(value))
end)
```

<!-- only cs -->
C# has no such command: a C# script is built into a `.dll` before the server runs it, so there is
no typed line to run. Keep a JavaScript or Lua scratch resource for one-liners, and try C# code in
a scratch resource of its own ([C#](csharp.md)), built again and restarted after each change.
<!-- end -->

Now try it in the server console:

```text
> js kcds.world.World.hour + 1
RESOURCE [scratch] 10.000095239166667
> js kcds.players.Player.all().map((p) => p.nickname)
RESOURCE [scratch] ["Hans"]
> js kcds.players.Player.all()[0].setHealth(20)
RESOURCE [scratch] true
```

The **Console** tab in KCDS-Admin runs the same commands.

> **Warning:** this command runs any code typed into the console. Use it only on a test server.

For client code, `resources/scratch/client/main.js` can put an experiment on a key:

```js
// client
kcds.ui.Key.bind('f6', () => {
  const ray = kcds.ui.Camera.rayFromScreen({ range: 30 });
  const hit = ray && kcds.world.World.raycast(ray.origin, ray.target, { mode: 'anything' });
  kcds.ui.Hud.showNotification(hit ? `${hit.kind} at ${hit.position}` : 'nothing within 30 m');
});
```

```lua
-- client
kcds.ui.Key.bind("f6", function()
  local ray = kcds.ui.Camera.rayFromScreen({ range = 30 })
  local hit = ray and kcds.world.World.raycast(ray.origin, ray.target, { mode = "anything" })
  kcds.ui.Hud.showNotification(hit and (hit.kind .. " at " .. tostring(hit.position)) or "nothing within 30 m")
end)
```

```cs
// client
Key.Bind("f6", new Func<Task>(async () =>
{
    var ray = Camera.RayFromScreen(new { range = 30 });
    var hit = ray.IsNull ? default : await World.Raycast(ray["origin"], ray["target"], new { mode = "anything" });
    Hud.ShowNotification(hit.IsNull ? "nothing within 30 m" : $"{hit["kind"]} at {hit["position"]}");
}));
```

## 3. Change, reload, test

1. Save `server.js` or `client/main.js`.
2. Enter `ensure scratch` in the server console. The resource restarts, and your game picks up the
   new client files without a restart of its own.
3. Use `js ...` in the console or press F6 in the game.

Server output appears in the console window and client output in your game's log; see
[Debugging resources](debugging.md).

## 4. Move finished code out

When an idea works, give it a proper resource ([Your first resource](first-resource.md),
[Project structure](project-structure.md)). Never leave the scratch resource on a public server.
