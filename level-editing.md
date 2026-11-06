# Editing the level

## Hiding or moving what the level placed

`kcds.world.LevelEdit` hides or moves a brush or level entity for all players in a dimension, whether to open up a wall, remove a
fence or move a cart out of the road. A brush is found by its mesh and the position the level gave it, within 5 cm. An entity is
found by its GUID.

```js
const fence = kcds.world.LevelEdit.apply({
  name: 'objects/manmade/structures/logistical/fences/fence_4rod_01.cgf',
  at: new kcds.math.Vector3(512.4, 830.1, 31.2),
  hidden: true,
});

// Later:
fence.restore();
```

```ts
const fence = kcds.world.LevelEdit.apply({
  name: 'objects/manmade/structures/logistical/fences/fence_4rod_01.cgf',
  at: new kcds.math.Vector3(512.4, 830.1, 31.2),
  hidden: true,
});

// Later:
fence.restore();
```

```lua
local fence = kcds.world.LevelEdit.apply({
  name = "objects/manmade/structures/logistical/fences/fence_4rod_01.cgf",
  at = kcds.math.Vector3.new(512.4, 830.1, 31.2),
  hidden = true,
})

-- Later:
fence:restore()
```

```cs
var fence = LevelEdit.Apply(new
{
    name = "objects/manmade/structures/logistical/fences/fence_4rod_01.cgf",
    at = new Vector3(512.4, 830.1, 31.2),
    hidden = true,
});

// Later:
fence.Restore();
```

**Entities:** `kind: 'entity'` with a `guid` edits a level entity, and `position`, `rotation` and `scale` move it.

**Swapping a model:** `model` replaces the object with another mesh from the props catalogue, as static scenery at the same spot
or wherever a move puts it. `LevelEdit.apply({ ...object, model: 'stump_fagus_sylvatica_a' })` leaves a stump where a tree was, and
`edit.model = ''` brings the object back.

**Collision:** a hidden brush takes its collision with it, including a building's colliders. In the villages, a house's outside,
inside and distant copy stand in one place, and the level keeps their collision separately. Hiding any of those brushes removes
every collider at exactly that place, so the house is gone for walking and for rays, until `restore()`. A moved house also takes its
colliders, so no invisible walls stay behind, but it has no collision at its new place.

**Lifetime:** edits are in memory, so a restart restores the level. `LevelEdit.all()` and `LevelEdit.restoreAll()` list and undo
them.

### Finding an object from a client

A client script can find the object with a ray. A hit on a brush, on a tree, bush or plant the level paints, or on a level entity,
has `levelObject`, containing all the fields a definition needs. A house's wall names the house itself, not its inside or a nearby
brush. A tree counts as a `'brush'` object, so an edit can hide or move it but not turn or resize it. Its `unseen` is true for
an invisible collision brush of the level's (a layer hides it); hiding it with a LevelEdit takes its collision away.

```js
// client: the first object two metres north of the player's chest
const { Vector3 } = kcds.math;

const feet = kcds.players.LocalPlayer.position;
const chest = new Vector3(feet.x, feet.y, feet.z + 1.3);
const ahead = new Vector3(feet.x, feet.y + 2, feet.z + 1.3);
const hit = kcds.world.World.raycast(chest, ahead, { mode: 'anything' });
if (hit?.levelObject) {
  kcds.core.Events.toServer('clear', hit.levelObject);
}
```

```ts
// client: the first object two metres north of the player's chest
const { Vector3 } = kcds.math;

const me = kcds.players.LocalPlayer;
if (me) {
  const feet = me.position;
  const chest = new Vector3(feet.x, feet.y, feet.z + 1.3);
  const ahead = new Vector3(feet.x, feet.y + 2, feet.z + 1.3);
  const hit = kcds.world.World.raycast(chest, ahead, { mode: 'anything' });
  if (hit?.levelObject) {
    kcds.core.Events.toServer('clear', hit.levelObject);
  }
}
```

```lua
-- client: the first object two metres north of the player's chest
local Vector3 = kcds.math.Vector3

local feet = kcds.players.LocalPlayer.position
local chest = Vector3.new(feet.x, feet.y, feet.z + 1.3)
local ahead = Vector3.new(feet.x, feet.y + 2, feet.z + 1.3)
local hit = kcds.world.World.raycast(chest, ahead, { mode = "anything" })
if hit and hit.levelObject then
  kcds.core.Events.toServer("clear", hit.levelObject)
end
```

```cs
// client: the first object two metres north of the player's chest
var feet = LocalPlayer.Current!.Position;
var chest = new Vector3(feet.X, feet.Y, feet.Z + 1.3);
var ahead = new Vector3(feet.X, feet.Y + 2, feet.Z + 1.3);
var hit = await World.Raycast(chest, ahead, new { mode = "anything" });
if (!hit["levelObject"].IsNull)
{
    Events.ToServer("clear", hit["levelObject"]);
}
```

```js
// server
kcds.core.Events.fromClient('clear', (player, object) => {
  kcds.world.LevelEdit.apply({
    kind: object.kind,
    name: object.name,
    guid: object.guid,
    at: object.at,
    hidden: true,
  });
});
```

```ts
// server
kcds.core.Events.fromClient('clear', (player, object) => {
  kcds.world.LevelEdit.apply({
    kind: object.kind,
    name: object.name,
    guid: object.guid,
    at: object.at,
    hidden: true,
  });
});
```

```lua
-- server
kcds.core.Events.fromClient("clear", function(player, object)
  kcds.world.LevelEdit.apply({
    kind = object.kind,
    name = object.name,
    guid = object.guid,
    at = object.at,
    hidden = true,
  })
end)
```

```cs
// server
Events.FromClient("clear", (Player player, JsValue found) =>
{
    LevelEdit.Apply(new
    {
        kind = found["kind"],
        name = found["name"],
        guid = found["guid"],
        at = found["at"],
        hidden = true,
    });
});
```

## Weapon strikes on the world

`playerWeaponHit(player, hit)` on the server and `weaponHit(hit)` on the client, for the local player only, report a weapon striking
the world rather than a character: a tree, wall, rock, prop or the ground. For players, NPCs and animals, use `playerHit` and the
damage events.

Each object counts once per attack, on first contact. Keeping the blade against a wall sends nothing new, while hitting a chair and
then a wall sends two events. Unarmed attacks send nothing.

`hit` has the same fields as a `World.raycast` hit, and `distance` is measured from the player's feet. This is what the server read
for a longsword's overhead swing into the inn wall at Zelejov, with `position`, `normal` and the rest of `levelObject` left out:

```json
{
  "kind": "brush",
  "surface": "mat_wood",
  "model": "objects/manmade/structures/municipal/pubs/unique/zelejov/zelejov_inn.cgf",
  "category": null,
  "distance": 2.57,
  "terrain": false,
  "entity": null,
  "levelObject": {
    "kind": "brush",
    "name": "objects/manmade/structures/municipal/pubs/unique/zelejov/zelejov_inn.cgf"
  }
}
```

The server takes this on the word of the player's game. It checks only that the hit is within 8 m of where the server has the player,
and allows at most 16 per second per player.

For a tree, `category` is `'tree'` and `levelObject` can go straight into `LevelEdit.apply`:

```js
const blowCounts = new Map();

kcds.core.Events.on('playerWeaponHit', (player, hit) => {
  if (hit.category !== 'tree' || !hit.levelObject) {
    return;
  }
  const key = `${hit.model}@${hit.levelObject.at.x},${hit.levelObject.at.y}`;
  const blows = (blowCounts.get(key) ?? 0) + 1;
  blowCounts.set(key, blows);
  if (blows >= 5) {
    kcds.world.LevelEdit.apply({ ...hit.levelObject, model: 'stump_fagus_sylvatica_a' });
    blowCounts.delete(key);
  }
});
```

```ts
const blowCounts = new Map();

kcds.core.Events.on('playerWeaponHit', (player, hit) => {
  if (hit.category !== 'tree' || !hit.levelObject) {
    return;
  }
  const key = `${hit.model}@${hit.levelObject.at.x},${hit.levelObject.at.y}`;
  const blows = (blowCounts.get(key) ?? 0) + 1;
  blowCounts.set(key, blows);
  if (blows >= 5) {
    kcds.world.LevelEdit.apply({ ...hit.levelObject, model: 'stump_fagus_sylvatica_a' });
    blowCounts.delete(key);
  }
});
```

```lua
local blowCounts = {}

kcds.core.Events.on("playerWeaponHit", function(player, hit)
  if hit.category ~= "tree" or not hit.levelObject then
    return
  end
  local at = hit.levelObject.at
  local key = hit.model .. "@" .. at.x .. "," .. at.y
  local blows = (blowCounts[key] or 0) + 1
  blowCounts[key] = blows
  if blows >= 5 then
    local edit = {}
    for field, value in pairs(hit.levelObject) do
      edit[field] = value
    end
    edit.model = "stump_fagus_sylvatica_a"
    kcds.world.LevelEdit.apply(edit)
    blowCounts[key] = nil
  end
end)
```

```cs
var blowCounts = new Dictionary<string, int>();

Events.On("playerWeaponHit", (Player player, JsValue hit) =>
{
    var found = hit["levelObject"];
    if (hit["category"].AsString() != "tree" || found.IsNull)
    {
        return;
    }
    var key = $"{hit["model"]}@{found["at"]["x"]},{found["at"]["y"]}";
    var blows = blowCounts.GetValueOrDefault(key) + 1;
    blowCounts[key] = blows;
    if (blows >= 5)
    {
        var edit = found.Keys.ToDictionary((field) => field, (field) => (object?)found[field]);
        edit["model"] = "stump_fagus_sylvatica_a";
        LevelEdit.Apply(edit);
        blowCounts.Remove(key);
    }
});
```

For ready-made tree chopping, see [Felling trees](trees.md).

## World exports

A world export is a JSON file of content the server spawns at startup, the same way a script would. It suits a map's fixed content,
and can hold props, effects, lights, level edits, ground items, doors, smart objects (seats, beds, tables and stations), areas and
patrol routes.

```json
{
  "level": "trosecko",
  "props": [{ "model": "beer_barrel", "position": { "x": 100, "y": 200, "z": 10 } }],
  "effects": [
    { "effect": "WH_Particels.other.magic", "position": { "x": 100, "y": 205, "z": 10 } }
  ],
  "lights": [{ "position": { "x": 100, "y": 205, "z": 12 }, "radius": 8, "intensity": 2 }],
  "world": [
    {
      "name": "objects/manmade/structures/logistical/fences/fence_4rod_01.cgf",
      "at": { "x": 512.4, "y": 830.1, "z": 31.2 },
      "hidden": true
    }
  ],
  "items": [
    {
      "item": "arrow_crude",
      "position": { "x": 101, "y": 200, "z": 10 },
      "amount": 20,
      "name": "quiver"
    }
  ],
  "doors": [
    {
      "model": "lvl1_door_a_left",
      "position": { "x": 106, "y": 200, "z": 10 },
      "name": "barn",
      "locked": true
    }
  ],
  "smartObjects": [
    {
      "blueprint": "Table/table_dice",
      "position": { "x": 104, "y": 200, "z": 10 },
      "name": "dice"
    }
  ],
  "areas": [],
  "patrolRoutes": []
}
```

Name the file `<name>.world.json` and list it in the `KCDS_WORLD_RESOURCES` environment variable, separated like `PATH`.
`worldResourcesReady` fires when all have loaded, and `WorldResource.find(name)` shows what each one spawned. A file with an error
adds nothing.

## Removing stations

`kcds.world.SmartObject.stations()` lists the stations in the server's level: dice tables, smithies, alchemy tables, drying racks and
smokehouses. Each has its GUID as `id`, `kind`, `position` and `placed: false`.

`SmartObject.remove(id, dimension?)` removes one from a dimension with a level edit on its entity. It is hidden on all clients in that
dimension and no longer listed there, and the server refuses any new dice match, crafting session, drying or smoking at it. Sessions
already running can finish. Removing it from the global dimension removes it from all of them.

```js
const { SmartObject } = kcds.world;

// No dice in the arena dimension, and no alchemy table at the Troskovice pharmacy.
const diceTable = SmartObject.stations().find((station) => station.kind === 'dice');
if (diceTable) {
  SmartObject.remove(diceTable.id, 5);
}
SmartObject.remove('2229e312da70017e');

console.info(JSON.stringify(SmartObject.removed()));
SmartObject.restore('2229e312da70017e');
```

```ts
const { SmartObject } = kcds.world;

// No dice in the arena dimension, and no alchemy table at the Troskovice pharmacy.
const diceTable = SmartObject.stations().find((station) => station.kind === 'dice');
if (diceTable) {
  SmartObject.remove(diceTable.id, 5);
}
SmartObject.remove('2229e312da70017e');

console.info(JSON.stringify(SmartObject.removed()));
SmartObject.restore('2229e312da70017e');
```

```lua
local SmartObject = kcds.world.SmartObject

-- No dice in the arena dimension, and no alchemy table at the Troskovice pharmacy.
for _, station in ipairs(SmartObject.stations()) do
  if station.kind == "dice" then
    SmartObject.remove(station.id, 5)
    break
  end
end
SmartObject.remove("2229e312da70017e")

print(json.encode(SmartObject.removed()))
SmartObject.restore("2229e312da70017e")
```

```cs
// No dice in the arena dimension, and no alchemy table at the Troskovice pharmacy.
var diceTable = SmartObject.Stations().FirstOrDefault((station) => station["kind"].AsString() == "dice");
if (!diceTable.IsNull)
{
    SmartObject.Remove(diceTable["id"], 5);
}
SmartObject.Remove("2229e312da70017e");

Log.Info(SmartObject.Removed().ToJson());
SmartObject.Restore("2229e312da70017e");
```

Furniture around a station is level scenery and stays, so use `LevelEdit.apply` for that. A world export's level edit that hides a
station also counts as a removal.

## Placing smart objects

`SmartObject.spawn(definition, dimension?)` places one of the game's interactive prefabs: a bench, chair, bed, dice table, alchemy table,
smithy, drying rack or smokehouse. Each client in the dimension, by default the global one, builds the whole prefab from the game's
template, including its furniture, solids and interaction entities. Later joiners build it too.

```js
const guildBench = kcds.world.SmartObject.spawn({
  blueprint: 'Bench/bench_2places_low', // from SmartObject.blueprints()
  position: [1650.5, 2160, 35.4], // the prefab's origin
  rotation: [0, 0, 0.7071, 0.7071], // optional: x, y, z, w
  name: 'guild bench',
});

console.info(guildBench.kind); // 'seat'
const saved = JSON.stringify(guildBench); // { blueprint, position, rotation, name }
guildBench.destroy();
```

```ts
const guildBench = kcds.world.SmartObject.spawn({
  blueprint: 'Bench/bench_2places_low', // from SmartObject.blueprints()
  position: [1650.5, 2160, 35.4], // the prefab's origin
  rotation: [0, 0, 0.7071, 0.7071], // optional: x, y, z, w
  name: 'guild bench',
});

console.info(guildBench.kind); // 'seat'
const saved = JSON.stringify(guildBench); // { blueprint, position, rotation, name }
guildBench.destroy();
```

```lua
local guildBench = kcds.world.SmartObject.spawn({
  blueprint = "Bench/bench_2places_low", -- from SmartObject.blueprints()
  position = { 1650.5, 2160, 35.4 }, -- the prefab's origin
  rotation = { 0, 0, 0.7071, 0.7071 }, -- optional: x, y, z, w
  name = "guild bench",
})

print(guildBench.kind) -- "seat"
local saved = json.encode(guildBench) -- { blueprint, position, rotation, name }
guildBench:destroy()
```

```cs
var guildBench = SmartObject.Spawn(new
{
    blueprint = "Bench/bench_2places_low", // from SmartObject.Blueprints()
    position = new[] { 1650.5, 2160, 35.4 }, // the prefab's origin
    rotation = new[] { 0, 0, 0.7071, 0.7071 }, // optional: x, y, z, w
    name = "guild bench",
});

Log.Info(guildBench.Kind); // "seat"
var saved = guildBench.AsValue().ToJson(); // { blueprint, position, rotation, name }
guildBench.Destroy();
```

- `SmartObject.blueprints()` lists what can be placed, as `{ name, kind, seats }`, sorted by name.
- `SmartObject.all(dimension?)` and `SmartObject.getById(id)` find placed ones.
- The server keeps nothing after a restart, so save `toJSON()` and place it again at startup. It is removed when the resource that made
  it stops.
- A placed dice table, alchemy table, smithy, drying rack or smokehouse joins `Dice.tables()`, `Crafting.stations()` and
  `SmartObject.stations()` with `placed: true`, and players use it like the level's own. `smartObject.stationKey` is its id there.
- NPCs do not use placed objects.

## Debug drawings

While building, you can draw shapes in the world, with `Gizmo` on the server for chosen players, or `Gizmos` in a client script for that
player alone.

```js
// server
const { Gizmo } = kcds.world;
const { Vector3, Color } = kcds.math;

Gizmo.line(player.position, npc.position, { color: Color.red(), durationMs: 5000 });
Gizmo.box(new Vector3(100, 200, 10), { x: 2, y: 2, z: 1 }, { players: [player] });
```

```ts
// server
const { Gizmo } = kcds.world;
const { Vector3, Color } = kcds.math;

Gizmo.line(player.position, npc.position, { color: Color.red(), durationMs: 5000 });
Gizmo.box(new Vector3(100, 200, 10), { x: 2, y: 2, z: 1 }, { players: [player] });
```

```lua
-- server
local Gizmo = kcds.world.Gizmo
local Vector3, Color = kcds.math.Vector3, kcds.math.Color

Gizmo.line(player.position, npc.position, { color = Color.red(), durationMs = 5000 })
Gizmo.box(Vector3.new(100, 200, 10), { x = 2, y = 2, z = 1 }, { players = { player } })
```

```cs
// server
Gizmo.Line(player.Position, npc.Position, new { color = Color.Red(), durationMs = 5000 });
Gizmo.Box(new Vector3(100, 200, 10), new { x = 2, y = 2, z = 1 }, new { players = new[] { player } });
```

```js
// client
kcds.world.Gizmos.set(
  'spawn',
  { type: 'sphere', position: new kcds.math.Vector3(100, 200, 10), radius: 2 },
  { color: 0xffffcc00 },
);
```

```ts
// client
kcds.world.Gizmos.set(
  'spawn',
  { type: 'sphere', position: new kcds.math.Vector3(100, 200, 10), radius: 2 },
  { color: 0xffffcc00 },
);
```

```lua
-- client
kcds.world.Gizmos.set(
  "spawn",
  { type = "sphere", position = kcds.math.Vector3.new(100, 200, 10), radius = 2 },
  { color = 0xffffcc00 }
)
```

```cs
// client
Gizmos.Set(
    "spawn",
    new { type = "sphere", position = new Vector3(100, 200, 10), radius = 2 },
    new { color = 0xffffcc00 });
```

Also: `edit.clearMove()` puts a moved object back where the level placed it, `worldResource.levelEdits` lists its level
edits still applied, and `Gizmos.setTransform(id, position, rotation)` moves and turns a whole drawing.

Reference: [LevelEdit](/docs/api/leveledit), [WorldResource](/docs/api/worldresource), [Gizmo](/docs/api/gizmo),
[Gizmos](/docs/api/gizmos), [SmartObject](/docs/api/smartobject).
