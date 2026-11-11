# What the player looks at

Client scripts can ask their own game about the world and get the answer within a frame or two, with no trip to
the server. This suits aiming, fishing, building, or pressing a key while looking at something.

```js
// client
const { Key, Camera } = kcds.ui;
const { World } = kcds.world;
const { Events } = kcds.core;

Key.bind('e', () => {
  const ray = Camera.rayFromScreen({ range: 4 });
  const hit = ray && World.raycast(ray.origin, ray.target, { mode: 'anything' });
  if (!hit) {
    return;
  }
  if (hit.entityClass === 'AnimDoor') {
    Events.toServer('useDoor', hit.entityGuid);
  } else if (hit.surface === 'mat_water') {
    Events.toServer('fish', hit.position);
  } else if (hit.category === 'tree') {
    Events.toServer('chop', { model: hit.model, position: hit.position });
  } else if (hit.player) {
    Events.toServer('greet', hit.player.id);
  }
});
```

```ts
// client
const { Key, Camera } = kcds.ui;
const { World } = kcds.world;
const { Events } = kcds.core;

Key.bind('e', () => {
  const ray = Camera.rayFromScreen({ range: 4 });
  const hit = ray && World.raycast(ray.origin, ray.target, { mode: 'anything' });
  if (!hit) {
    return;
  }
  if (hit.entityClass === 'AnimDoor') {
    Events.toServer('useDoor', hit.entityGuid);
  } else if (hit.surface === 'mat_water') {
    Events.toServer('fish', hit.position);
  } else if (hit.category === 'tree') {
    Events.toServer('chop', { model: hit.model, position: hit.position });
  } else if (hit.player) {
    Events.toServer('greet', hit.player.id);
  }
});
```

```lua
-- client
local Key, Camera = kcds.ui.Key, kcds.ui.Camera
local World = kcds.world.World
local Events = kcds.core.Events

Key.bind("e", function()
  local ray = Camera.rayFromScreen({ range = 4 })
  local hit = ray and World.raycast(ray.origin, ray.target, { mode = "anything" })
  if not hit then
    return
  end
  if hit.entityClass == "AnimDoor" then
    Events.toServer("useDoor", hit.entityGuid)
  elseif hit.surface == "mat_water" then
    Events.toServer("fish", hit.position)
  elseif hit.category == "tree" then
    Events.toServer("chop", { model = hit.model, position = hit.position })
  elseif hit.player then
    Events.toServer("greet", hit.player.id)
  end
end)
```

```cs
// client
Key.Bind("e", new Func<Task>(async () =>
{
    var ray = Camera.RayFromScreen(new { range = 4 });
    if (ray.IsNull)
    {
        return;
    }
    var hit = await World.Raycast(ray["origin"], ray["target"], new { mode = "anything" });
    if (hit.IsNull)
    {
        return;
    }
    if (hit["entityClass"].AsString() == "AnimDoor")
    {
        Events.ToServer("useDoor", hit["entityGuid"]);
    }
    else if (hit["surface"].AsString() == "mat_water")
    {
        Events.ToServer("fish", hit["position"]);
    }
    else if (hit["category"].AsString() == "tree")
    {
        Events.ToServer("chop", new { model = hit["model"], position = hit["position"] });
    }
    else if (!hit["player"].IsNull)
    {
        Events.ToServer("greet", hit["player"]["id"]);
    }
}));
```

| Call                                                                 | Result                                                                                     |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `Camera.getPose()`                                                   | `{ position, forward, right, up, fov, aspectRatio }`, or `null` in a menu or while loading |
| `Camera.rayFromScreen({ x, y, range })`                              | `{ origin, direction, target }`; x and y run from -1 to 1, default centre                  |
| `World.raycast(from, to, { mode, ignoreSelf })`                      | the first thing hit, or `null`                                                             |
| `World.traceAll(from, to, { mode, maxHits })`                        | everything along the line, nearest first, up to 8                                          |
| `World.groundHeight(position, { up, down })`                         | the ground's height under a point; a bridge or floor counts                                |
| `World.snapToGround(...)`                                            | the same, as a hit                                                                         |
| `World.entitiesAround(centre, radius, { class, max, physicalOnly })` | entities in a sphere up to 256 m, nearest first, with class, name and level GUID           |

`mode` is `'cover'` (the world, props and doors), `'anything'` (bodies too) or `'ground'`.

## What a hit contains

| Field                       | Meaning                                                                            |
| --------------------------- | ---------------------------------------------------------------------------------- |
| `kind`                      | `'terrain'`, `'entity'`, `'vegetation'`, `'brush'` or `'static'`                   |
| `surface`                   | such as `mat_wood` or `mat_water`                                                  |
| `model`, `category`         | a tree, bush or placed mesh; category is `'tree'`, `'bush'`, `'plant'` or `'rock'` |
| `entityClass`, `entityName` | for an entity                                                                      |
| `entityGuid`                | the level's sixteen hex digits, the same on all machines                           |
| `player`                    | whose body it was                                                                  |

Do not treat a client's answer as proof. A server that must rely on it should check for itself. On the server,
`World.raycast` asks a player's game and returns a promise.

The server has no world of its own, so `World.snapToGround(player, position)` also asks a player's game for the
ground under a point. With `null` for the player, any ready player on the map answers, and with nobody there it
resolves with `answered: false` and `reason: 'no player'`. An answer traced from the top (no `z` given) is kept for
ten minutes per half-metre cell, so the same place is known again without asking anyone (`reason: 'kept'`).

```js
// server: the ground under a camp, whoever is online
const { answered, z } = await kcds.world.World.snapToGround(null, { x: 512, y: 1800 });
if (answered && z !== null) {
  console.info(`the camp stands at ${z.toFixed(1)} m`);
}
```

```ts
// server: the ground under a camp, whoever is online
const { answered, z } = await kcds.world.World.snapToGround(null, { x: 512, y: 1800 });
if (answered && z !== null) {
  console.info(`the camp stands at ${z.toFixed(1)} m`);
}
```

```lua
-- server: the ground under a camp, whoever is online
local ground = kcds.world.World.snapToGround(nil, { x = 512, y = 1800 }):await()
if ground.answered and ground.z ~= nil then
  print("the camp stands at " .. tostring(ground.z) .. " m")
end
```

```cs
// server: the ground under a camp, whoever is online
var result = await World.SnapToGround(null, new { x = 512, y = 1800 });
if (result["answered"].AsBool() && !result["z"].IsNull)
{
    Log.Info($"the camp stands at {result["z"].AsNumber():F1} m");
}
```

## Acting on the target, checked by the server

A target action takes three parts:

1. Offer it on the game's own prompt with an [action hint](action-hints.md), beside or instead of the game's own Talk
   or Loot.
2. When the player presses it, the client finds the target with `World.raycast` (line of sight included) and tells the
   server.
3. The server checks it again with `player.canReach(target, range)` before it acts: the player is up, the target is in
   their dimension and within range (3.5 m by default).

```js
// client
const { ActionHint, Camera } = kcds.ui;
const { World } = kcds.world;

ActionHint.create({ key: 'script_action', text: 'Greet' }, () => {
  const ray = Camera.rayFromScreen({ range: 4 });
  const hit = ray && World.raycast(ray.origin, ray.target, { mode: 'anything' });
  if (hit && hit.player) {
    kcds.core.Events.toServer('greet', hit.player.id);
  }
});
```

```ts
// client
const { ActionHint, Camera } = kcds.ui;
const { World } = kcds.world;

ActionHint.create({ key: 'script_action', text: 'Greet' }, () => {
  const ray = Camera.rayFromScreen({ range: 4 });
  const hit = ray && World.raycast(ray.origin, ray.target, { mode: 'anything' });
  if (hit && hit.player) {
    kcds.core.Events.toServer('greet', hit.player.id);
  }
});
```

```lua
-- client
local ActionHint, Camera = kcds.ui.ActionHint, kcds.ui.Camera
local World = kcds.world.World

ActionHint.create({ key = "script_action", text = "Greet" }, function()
  local ray = Camera.rayFromScreen({ range = 4 })
  local hit = ray and World.raycast(ray.origin, ray.target, { mode = "anything" })
  if hit and hit.player then
    kcds.core.Events.toServer("greet", hit.player.id)
  end
end)
```

```cs
// client
ActionHint.Create(new { key = "script_action", text = "Greet" }, new Func<Task>(async () =>
{
    var ray = Camera.RayFromScreen(new { range = 4 });
    if (ray.IsNull)
    {
        return;
    }
    var hit = await World.Raycast(ray["origin"], ray["target"], new { mode = "anything" });
    if (!hit.IsNull && !hit["player"].IsNull)
    {
        Events.ToServer("greet", hit["player"]["id"]);
    }
}));
```

```js
// server
kcds.core.Events.fromClient('greet', (player, targetId) => {
  const target = kcds.players.Player.getById(Number(targetId));
  if (!target || !player.canReach(target)) {
    return;
  }
  kcds.core.Chat.sendToPlayer(target, `${player.nickname} greets you.`);
});
```

```ts
// server
kcds.core.Events.fromClient('greet', (player, targetId) => {
  const target = kcds.players.Player.getById(Number(targetId));
  if (!target || !player.canReach(target)) {
    return;
  }
  kcds.core.Chat.sendToPlayer(target, `${player.nickname} greets you.`);
});
```

```lua
-- server
kcds.core.Events.fromClient("greet", function(player, targetId)
  local target = kcds.players.Player.getById(tonumber(targetId))
  if not target or not player:canReach(target) then
    return
  end
  kcds.core.Chat.sendToPlayer(target, player.nickname .. " greets you.")
end)
```

```cs
// server
Events.FromClient("greet", (Player player, JsValue targetId) =>
{
    var target = Player.GetById(targetId.AsNumber());
    if (target == null || !player.CanReach(target))
    {
        return;
    }
    Chat.SendToPlayer(target, $"{player.Nickname} greets you.");
});
```

Reference: [Camera](/docs/api/camera), [World](/docs/api/world), [ActionHint](/docs/api/actionhint),
[Player](/docs/api/player).
