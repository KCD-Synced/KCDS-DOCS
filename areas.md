# Areas

`kcds.world.Area` represents a volume in the world, and events fire when a player, horse or NPC goes in or out. The
server does this with no timer. It only checks bodies whose position or dimension changed, and only against nearby areas,
so a body standing still costs nothing.

An area is one of two kinds:

| `kind`     | What it is                                                                                              |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| `'level'`  | every `TriggerArea` and `SmartAreaShape` on the server's map: about 3,300 on Trosky (`trosecko`) and 6,900 on Kuttenberg (`kutnohorsko`) |
| `'script'` | a box, sphere or polygon with a height, made by your resource with `Area.create`                        |

```js
const { Area } = kcds.world;
const { Vector3 } = kcds.math;
const { Events } = kcds.core;

Area.create({
  id: 'town-gate',
  type: 'box',
  position: new Vector3(100, 200, 10),
  min: new Vector3(-5, -5, -2),
  max: new Vector3(5, 5, 4),
  labels: ['town'],
});

Events.on('areaEnter', (area, entity, matchingDimension) => {
  if (area.id === 'town-gate' && matchingDimension) {
    console.info(`entered: ${entity.id}`);
  }
});

Events.on('areaExit', (area, entity) => {
  console.info(`left ${area.id}: ${entity.id}`);
});
```

```lua
local Area = kcds.world.Area
local Vector3 = kcds.math.Vector3
local Events = kcds.core.Events

Area.create({
  id = "town-gate",
  type = "box",
  position = Vector3.new(100, 200, 10),
  min = Vector3.new(-5, -5, -2),
  max = Vector3.new(5, 5, 4),
  labels = { "town" },
})

Events.on("areaEnter", function(area, entity, matchingDimension)
  if area.id == "town-gate" and matchingDimension then
    print("entered: " .. entity.id)
  end
end)

Events.on("areaExit", function(area, entity)
  print("left " .. area.id .. ": " .. entity.id)
end)
```

```cs
Area.Create(new
{
    id = "town-gate",
    type = "box",
    position = new Vector3(100, 200, 10),
    min = new Vector3(-5, -5, -2),
    max = new Vector3(5, 5, 4),
    labels = new[] { "town" },
});

Events.On("areaEnter", (Area area, Entity entity, bool matchingDimension) =>
{
    if (area.Id == "town-gate" && matchingDimension)
    {
        Log.Info($"entered: {entity.Id}");
    }
});

Events.On("areaExit", (Area area, Entity entity) =>
{
    Log.Info($"left {area.Id}: {entity.Id}");
});
```

## Level areas

These include houses, inns, interiors, private grounds, stealth areas, settlements and places where a horse cannot be
called. Each has the level's name and the labels the game's AI reads, such as `private`, `personal`, `semipersonal`,
`semipublic`, `interior`, `prohibited`, `stealthArea_crouch`, `dogForbidden` and `suppressHorseCall`.

**Stable id:** a level area's `id` is its EntityGuid, as 16 lower-case hex characters. It is the same on all computers and
after restarts, so it is safe to store.

**Mostly read-only:** only `enabled` can change, and switching it off fires `areaExit` for everyone inside. `setShape` and
`destroy` return `false`, `toJSON()` returns `null`, and assigning anything else throws. A level area belongs to all
dimensions.

```js
const { Area } = kcds.world;
const { Player } = kcds.players;

// Tell players when they step onto private ground.
kcds.core.Events.on('areaEnter', (area, entity) => {
  if (area.kind === 'level' && entity instanceof Player && area.labels.includes('private')) {
    kcds.core.Chat.sendToPlayer(entity, `You are on private ground (${area.name}).`);
  }
});

// The inn of Troskovice, by its id.
const inn = Area.getById('108c8fabab6e4bf9');
console.info(inn.type, inn.height, inn.players.length);

// All interiors on the map, and those a player is in.
const interiors = Area.all('level').filter((area) => area.labels.includes('interior'));
const here = Area.aroundEntity(player).filter((area) => area.labels.includes('interior'));

// Turn off all stealth areas on this server.
for (const area of Area.taggedWith('stealthArea_crouch')) {
  area.enabled = false;
}
```

```lua
local Area = kcds.world.Area
local Player = kcds.players.Player

local function hasLabel(area, label)
  for _, own in ipairs(area.labels) do
    if own == label then
      return true
    end
  end
  return false
end

-- Tell players when they step onto private ground.
kcds.core.Events.on("areaEnter", function(area, entity)
  local player = Player.getById(entity.id)
  if area.kind == "level" and player == entity and hasLabel(area, "private") then
    kcds.core.Chat.sendToPlayer(entity, "You are on private ground (" .. area.name .. ").")
  end
end)

-- The inn of Troskovice, by its id.
local inn = Area.getById("108c8fabab6e4bf9")
print(inn.type, inn.height, #inn.players)

-- All interiors on the map, and those a player is in.
local interiors, here = {}, {}
for _, area in ipairs(Area.all("level")) do
  if hasLabel(area, "interior") then
    table.insert(interiors, area)
  end
end
for _, area in ipairs(Area.aroundEntity(player)) do
  if hasLabel(area, "interior") then
    table.insert(here, area)
  end
end

-- Turn off all stealth areas on this server.
for _, area in ipairs(Area.taggedWith("stealthArea_crouch")) do
  area.enabled = false
end
```

```cs
// Tell players when they step onto private ground.
Events.On("areaEnter", (Area area, Entity entity) =>
{
    if (area.Kind.AsString() == "level" && entity is Player visitor && area.Labels.Contains("private"))
    {
        Chat.SendToPlayer(visitor, $"You are on private ground ({area.Name}).");
    }
});

// The inn of Troskovice, by its id.
var inn = Area.GetById("108c8fabab6e4bf9");
Log.Info($"{inn?.Type} {inn?.Height} {inn?.Players.Length}");

// All interiors on the map, and those a player is in.
var interiors = Area.All("level").Where((area) => area.Labels.Contains("interior")).ToList();
var here = Area.AroundEntity(player).Where((area) => area.Labels.Contains("interior")).ToList();

// Turn off all stealth areas on this server.
foreach (var area in Area.TaggedWith("stealthArea_crouch"))
{
    area.Enabled = false;
}
```

**Shapes:**

- `TriggerArea` is a prism baked into the level: an outline, and how far `height` reaches above its lowest corner.
- `SmartAreaShape` is the outline of a settlement or location, often with no ceiling, so `height` is 0.
- `AreaUnion` is a house of several rooms, treated as one area with the house's labels. A body is inside while in any room,
  and `members` lists the rooms, so a whole house fires one enter and one exit.

The catalogue is built from the game's level files by `Dev/tools/areas/build_level_areas.py`.

## Creating an area

`Area.create(definition)`

| Field              | Meaning                                                                                        |
| ------------------ | ---------------------------------------------------------------------------------------------- |
| `id`               | required; 1 to 128 characters, unique; must not be sixteen hex digits, the form level areas use |
| `name`             | up to 128 characters; `id` by default                                                          |
| `type`             | `'box'`, `'sphere'` or `'shape'` (a polygon)                                                   |
| `position`         | the centre as `{ x, y, z }`; `(0, 0, 0)` by default                                            |
| `rotation`         | a quaternion or finite Euler angles; none by default                                           |
| `min`, `max`       | for `box`; each component of `max` must exceed `min`                                           |
| `radius`           | for `sphere`; above 0                                                                          |
| `points`, `height` | for `shape`; 3 to 256 points in the XY plane; `height` is 0 or more, and 0 means no floor or ceiling |
| `labels`           | up to 64 strings of 1 to 64 characters                                                         |
| `metadata`         | an object that fits in JSON, up to 65536 characters                                            |
| `enabled`          | `true` (default) or `false`                                                                    |
| `dimension`        | the dimension it belongs to, or `null` for none                                                |
| `collision`        | `true` adds invisible walls; see below; default `false`                                        |
| `wallBottom`, `wallTop` | with `collision`, closes a box or shape at the bottom or top; default `false`             |
| `wallThickness`    | wall thickness in metres, centred on the outline; 0.05 to 2; default `0.2`                     |

Z is up. A change from `create`, `setShape` or `enabled` is evaluated once your code returns, so its events come after you
have the area. `destroy` fires exits immediately.

## Collision walls

With `collision`, a script area gets invisible walls that block from both sides. A box gets them on four sides, a shape on
each edge from its lowest point up its `height`, and a sphere gets a closed shell. Use `wallBottom` and `wallTop` to close a
box or shape.

The walls follow the area. `setShape` moves them, `enabled = false` removes them, `dimension` limits them to that dimension,
and `destroy` removes them. They belong to the area and are not props, so `Prop.all()` does not list them.

A shape needs a `height` above the wall thickness and edges that do not cross. A box or sphere too small for the thickness is
refused. In these cases, `create` throws a `RangeError` and `setShape` returns `false`. Level areas cannot have walls.

```js
const { Vector3 } = kcds.math;

const pen = kcds.world.Area.create({
  id: 'pen',
  type: 'box',
  position: new Vector3(1650, 2150, 35),
  min: new Vector3(-4, -4, 0),
  max: new Vector3(4, 4, 3),
  collision: true,
  wallTop: true,
});

kcds.core.Events.on('playerCommand', (player, command) => {
  if (command === 'open') {
    pen.collision = false;
  }
});
```

```lua
local Vector3 = kcds.math.Vector3

local pen = kcds.world.Area.create({
  id = "pen",
  type = "box",
  position = Vector3.new(1650, 2150, 35),
  min = Vector3.new(-4, -4, 0),
  max = Vector3.new(4, 4, 3),
  collision = true,
  wallTop = true,
})

kcds.core.Events.on("playerCommand", function(player, command)
  if command == "open" then
    pen.collision = false
  end
end)
```

```cs
var pen = Area.Create(new
{
    id = "pen",
    type = "box",
    position = new Vector3(1650, 2150, 35),
    min = new Vector3(-4, -4, 0),
    max = new Vector3(4, 4, 3),
    collision = true,
    wallTop = true,
});

Events.On("playerCommand", (Player player, string command) =>
{
    if (command == "open")
    {
        pen.Collision = false;
    }
});
```

## Members

**Read only:** `exists`, `id`, `bounds` (`{ min, max }`), `height`, `players`, `occupants`, `kind`, and `type`, which is
`'TriggerArea'`, `'SmartAreaShape'`, `'box'`, `'sphere'` or `'shape'`.

**Read and write:** `enabled`. On a script area, also `name`, `labels`, `metadata`, `dimension`, `position`, `rotation`,
`collision`, `wallBottom`, `wallTop` and `wallThickness`.

**Methods:**

| Method             | Result                                                                                    |
| ------------------ | ----------------------------------------------------------------------------------------- |
| `contains(position)` | whether a point is inside, ignoring `enabled` and dimension                             |
| `holdsEntity(body)` | whether a player, horse or NPC is inside, in the area's dimension                        |
| `setShape(fields)` | moves, turns or reshapes it, leaving out fields unchanged; `false` if gone or a level area |
| `destroy()`        | fires exits and removes it; `false` if already gone or a level area                       |
| `toJSON()`         | a definition to rebuild it; `null` for a level area                                       |

## Finding areas

| Call                                    | Result                                                  |
| --------------------------------------- | ------------------------------------------------------- |
| `Area.getById(id)`                      | the area or `null`; level areas by sixteen hex digits   |
| `Area.getAt(position, dimension?)`      | enabled areas containing a point, of both kinds         |
| `Area.aroundEntity(body)`               | enabled areas the body is in                            |
| `Area.nearby(position, radius, dimension?)` | areas whose bounds are in the radius, nearest first |
| `Area.taggedWith(label)`                | areas with a label, ignoring case; level areas first    |
| `Area.all(kind?)`                       | all areas, level areas first; `'level'` or `'script'` limits them |

## Events

`areaEnter(area, entity, matchingDimension)` and `areaExit(area, entity, matchingDimension)`. Being inside is about geometry,
whichever dimension the body is in. `matchingDimension` says whether the area belongs to the body's dimension, or to none, as
a level area does.

Reference: [Area](/docs/api/area).
