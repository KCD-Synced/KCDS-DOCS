# Debugging resources

## Finding the logs

### Server

Whatever a server script writes with `console.log`, `info`, `warn`, `error` or `debug` goes to the server's
console window, to the server's log file `logs/server.log` ([Running a server](server.md#starting-it)) and
to the **Console** tab of [KCDS-Admin](server.md#kcds-admin), labelled with the resource's name:

```text
RESOURCE [hello] started
RESOURCE [hello] Hans asked for the count
```

Values are joined as text, so an object appears as `[object Object]`. Convert it to JSON first:

```js
console.info('inventory', JSON.stringify(player.getInventory()));
```

```ts
console.info('inventory', JSON.stringify(player.getInventory()));
```

```lua
print("inventory", json.encode(player:getInventory()))
```

```cs
Log.Info("inventory " + player.GetInventory().ToJson());
```

If a server ends abnormally, by a crash or a killed process, it writes a crash report with the last 200
log lines to `crashes/`, next to the resources folder. KCDS-Admin lists these. Ctrl+C, `systemctl stop` and
`docker stop` count as normal stops: resources shut down as usual, and no report is written.

### Client

`console` lines from a client script go to the player's own game log, marked `RESOURCE client <text>`. The
file is `client-<process id>.log` in `%LOCALAPPDATA%\KCDS\logs`, beside the launcher's `launcher.log`; the
folder button next to the gear in the main menu opens that folder. A game's log grows to 32 MB at most; then
its lines so far move to `client-<process id>.old.log` and the log starts again, so a long session keeps its
newest lines. The 20 newest client log files are kept, 256 MB in all at most. Before
1.0.1 the file was next to `kcd2s-game-client-v4.dll` in the KCDS folder (`%LOCALAPPDATA%\Programs\KCDS`
with the installer).

The newest file belongs to the running game. If a problem only happens for one player, ask them to send you
theirs.

The launcher's own log is at `%LOCALAPPDATA%\KCDS\logs\launcher.log`.

## What errors do

An uncaught error, or a rejected promise nobody handles, is logged with its stack. What happens next depends
on `errorBehavior` in the manifest:

| `errorBehavior`    | Result                                                                                                    |
| ------------------ | --------------------------------------------------------------------------------------------------------- |
| `stop` (default)   | the resource stops, and everything it registered and spawned is removed                                   |
| `restart`          | the resource stops and starts again one second later                                                      |
| `continue`         | handlers, timers and exports keep running; what it spawned in the world (NPCs, props, chests, areas and so on) is removed, and nothing else is undone |

So catch errors where you can deal with them, especially around `await`:

```js
kcds.core.Events.fromClient('shop:buy', async (player, payload) => {
  try {
    await buy(player, payload);
  } catch (problem) {
    console.warn(`${player.nickname} could not buy: ${problem.message}`);
    kcds.core.Chat.sendToPlayer(player, 'That did not work.');
  }
});
```

```ts
kcds.core.Events.fromClient('shop:buy', async (player, payload) => {
  try {
    await buy(player, payload);
  } catch (problem) {
    const message = problem instanceof Error ? problem.message : String(problem);
    console.warn(`${player.nickname} could not buy: ${message}`);
    kcds.core.Chat.sendToPlayer(player, 'That did not work.');
  }
});
```

```lua
-- buy answers a promise; :catch is the try/catch of a promise.
kcds.core.Events.fromClient("shop:buy", function(player, payload)
  return buy(player, payload):catch(function(problem)
    print(player.nickname .. " could not buy: " .. tostring(problem.message or problem))
    kcds.core.Chat.sendToPlayer(player, "That did not work.")
  end)
end)
```

```cs
Events.FromClient("shop:buy", async (Player player, JsValue payload) =>
{
    try
    {
        await buy(player, payload);
    }
    catch (Exception problem)
    {
        Log.Warn($"{player.Nickname} could not buy: {problem.Message}");
        Chat.SendToPlayer(player, "That did not work.");
    }
});
```

SDK functions throw `TypeError` for an argument of the wrong type and `RangeError` for a value out of range,
naming the argument in the message. Each reference page says what its members throw or reject with.

## Testing a change without restarting

1. Save your files.
2. Enter `ensure <resource>` or `restart <resource>` in the server console, or press **Restart** on the
   Resources tab of KCDS-Admin.
3. The resource stops, running its `resourceStop` handlers. Its files are read again and it starts.
   Online players download the changed client files and run the new client part.

`ensure` also starts a resource folder you have just added. `refresh` only reads new and changed manifests,
and starts nothing. See [Console commands](server.md#console-commands).

A console command of your own is handy for quick checks:

```js
kcds.core.Events.on('consoleCommand', (command, args) => {
  if (command !== 'where') {
    return;
  }
  for (const player of kcds.players.Player.all()) {
    console.info(`${player.nickname} at ${player.position}`);
  }
});
```

```ts
kcds.core.Events.on('consoleCommand', (command, args) => {
  if (command !== 'where') {
    return;
  }
  for (const player of kcds.players.Player.all()) {
    console.info(`${player.nickname} at ${player.position}`);
  }
});
```

```lua
kcds.core.Events.on("consoleCommand", function(command, args)
  if command ~= "where" then
    return
  end
  for _, player in ipairs(kcds.players.Player.all()) do
    print(player.nickname .. " at " .. tostring(player.position))
  end
end)
```

```cs
Events.On("consoleCommand", (string command, string[] args) =>
{
    if (command != "where")
    {
        return;
    }
    foreach (var player in Player.All())
    {
        Log.Info($"{player.Nickname} at {player.Position}");
    }
});
```

For a fuller setup, see [Trying code quickly](playground.md).

## Common problems

| Symptom                                                  | Likely cause                                                                                 |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `ensure hello` does nothing, not even `started`          | the console needs the `name` from `package.json`, not the folder's name                      |
| nothing in the resource runs                             | scripts are missing from `serverScripts` or `clientScripts`, or `package.json` has invalid JSON (see the server log) |
| `ReferenceError: Door is not defined`                    | the type needs its full path, `kcds.world.Door`                                              |
| `Cannot read properties of undefined` on `kcds.ui.Hud`   | a client type in a server script; each reference page says which side has the type           |
| a client event never reaches the server                  | the server used `Events.on`, but events from players need `Events.fromClient`                |
| `Events.toServer` works but the answer never appears     | the server must answer with `Events.broadcast` or `player.emit`, and the client listens with `Events.on(name)` |
| a command handler runs for every command                 | `playerCommand` gets all commands, so compare `command` first                                |
| an object logs as `[object Object]`                      | log `JSON.stringify(value)`                                                                  |
| a timer keeps going after a fix                          | the old code started it; `ensure` the resource to clear its timers                           |

## Next steps

- [Events](events.md): all events and their order
- [Timers, logging and the runtime](timers.md): timers, `consoleCommand` and `Runtime`
