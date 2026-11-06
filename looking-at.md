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

Reference: [Camera](/docs/api/camera), [World](/docs/api/world).
