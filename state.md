# Shared state

Each player, horse, NPC and dog has `entity.state`, a set of values that the server writes and that players' games can read. It
is the place for anything other resources or clients should see, such as a job, a faction or a "wanted" flag.

```js
// server
player.state.set('job', 'blacksmith'); // all players see it
player.state.set('money', 120, { scope: 'owner' }); // only this player's game
player.state.set('notes', 'none', { scope: 'server' }); // stays on the server

player.state.onChange('job', (value) => {
  console.info(`job is now ${value}`);
});
```

```ts
// server
player.state.set('job', 'blacksmith'); // all players see it
player.state.set('money', 120, { scope: 'owner' }); // only this player's game
player.state.set('notes', 'none', { scope: 'server' }); // stays on the server

player.state.onChange('job', (value: unknown) => {
  console.info(`job is now ${value}`);
});
```

```lua
-- server
player.state:set("job", "blacksmith") -- all players see it
player.state:set("money", 120, { scope = "owner" }) -- only this player's game
player.state:set("notes", "none", { scope = "server" }) -- stays on the server

player.state:onChange("job", function(value)
  print("job is now " .. tostring(value))
end)
```

```cs
// server
player.State.Set("job", "blacksmith"); // all players see it
player.State.Set("money", 120, new { scope = "owner" }); // only this player's game
player.State.Set("notes", "none", new { scope = "server" }); // stays on the server

player.State.OnChange("job", (JsValue value) =>
{
    Log.Info($"job is now {value}");
});
```

```js
// client
kcds.core.Events.on('playerReady', () => {
  const job = kcds.players.LocalPlayer.state.get('job');
  kcds.ui.Hud.showNotification(`You work as a ${job}`);
});
```

```ts
// client
kcds.core.Events.on('playerReady', () => {
  const job = kcds.players.LocalPlayer?.state.get('job');
  kcds.ui.Hud.showNotification(`You work as a ${job}`);
});
```

```lua
-- client
kcds.core.Events.on("playerReady", function()
  local job = kcds.players.LocalPlayer.state:get("job")
  kcds.ui.Hud.showNotification("You work as a " .. tostring(job))
end)
```

```cs
// client
Events.On("playerReady", () =>
{
    var job = LocalPlayer.Current?.State.Get("job");
    Hud.ShowNotification($"You work as a {job}");
});
```

## Who sees a value

| `scope`       | Sent to                                |
| ------------- | -------------------------------------- |
| `'broadcast'` | all players; the default               |
| `'owner'`     | only the player the entity belongs to  |
| `'server'`    | nobody                                 |

## Limits

- Keys are strings of up to 64 bytes.
- Values are booleans, numbers, strings or anything else JSON can hold, up to 4096 bytes.
- One entity's state holds up to 128 keys.

## Calls

`get(key)`, `has(key)`, `keys()`, `toObject()` and `remove(key)` work as their names suggest.

`onChange(key, handler)` watches a key, or any key if you pass `null`. It returns a function that removes the handler.

## How long it lasts

State lasts as long as its entity, so a player's goes when they leave. Store anything that must survive a reconnect yourself.

Reference: [SharedState](/docs/api/sharedstate).
