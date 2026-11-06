# Props and placement

Three types work with models in the world:

| Type         | What it is                                                               |
| ------------ | ------------------------------------------------------------------------ |
| `Prop`       | a model from the game's catalogue, spawned for all players in view       |
| `PropGhost`  | a see-through preview that only one player sees                          |
| `PropPlacer` | lets a player aim a preview with their crosshair, then confirm or cancel |

The server decides again when a placement is confirmed, so the client's word alone is never enough.

```js
const { Prop, PropPlacer } = kcds.world;
const { Vector3 } = kcds.math;
const { Events } = kcds.core;

const crate = Prop.spawn(
  'crate_box_a',
  new Vector3(100, 200, 10),
  new Vector3(0, 0, 90),
  1.5,
  'static',
);
crate.physics = 'rigid';

Events.on('playerReady', (player) => {
  PropPlacer.begin(player, { model: 'crate_box_a', range: 8, spawn: { physics: 'static' } });
});

Events.on('propPlaceConfirm', (player, session, placement) => placement.onSurface);
```

```lua
local Prop, PropPlacer = kcds.world.Prop, kcds.world.PropPlacer
local Vector3 = kcds.math.Vector3
local Events = kcds.core.Events

local crate = Prop.spawn("crate_box_a", Vector3.new(100, 200, 10), Vector3.new(0, 0, 90), 1.5, "static")
crate.physics = "rigid"

Events.on("playerReady", function(player)
  PropPlacer.begin(player, { model = "crate_box_a", range = 8, spawn = { physics = "static" } })
end)

Events.on("propPlaceConfirm", function(player, session, placement) return placement.onSurface end)
```

```cs
var crate = Prop.Spawn("crate_box_a", new Vector3(100, 200, 10), new Vector3(0, 0, 90), 1.5, "static");
crate.Physics = "rigid";

Events.On("playerReady", (Player player) =>
{
    PropPlacer.Begin(player, new { model = "crate_box_a", range = 8, spawn = new { physics = "static" } });
});

Events.On("propPlaceConfirm", (Player player, JsValue session, JsValue placement) => placement["onSurface"].AsBool());
```

## Prop

`Prop.spawn(model, position, orientation?, scale?, physics?, dimension?)`

| Argument      | Accepted values                                                                |
| ------------- | ------------------------------------------------------------------------------ |
| `model`       | a path, or a file name with no extension, in any case; an unknown model throws |
| `orientation` | a quaternion, or angles in degrees                                             |
| `scale`       | a number or vector; each part is limited to 0.01 to 100; default 1             |
| `physics`     | `'static'` (default), `'rigid'` or `'none'`                                    |

**Read only:** `id`, `model`, `meshName`, `meshGroup`, `dimension`, `position`, `rotation`.

**Read and write:** `physics`, `mass`, and `scale`, a number while its parts are equal, otherwise a `Vector3`. A change rebuilds the prop for all
players.

**Methods:** `destroy()`, `Prop.all()`, `Prop.getById(id)`, and `Prop.destroyAll(dimension?)`, which returns how many it removed.

**Events:** `propSpawn(prop)`, `propDestroy(prop)`, `propRest(prop, position, rotation)`, `propHit(prop, speed, position)`.

### Who sees it

Every player of its dimension sees a prop unless you say otherwise. `prop.showOnlyTo(players)` shows it to those
players only, and `prop.hideFrom(players)` hides it from them while everyone else still sees it. Both hold for players
who join later, and `null` undoes them. The two combine: a card held by one player can be two props in the same place,
its face shown only to the holder and its back hidden from the holder.

```js
// A card only `player` sees the face of; everyone else sees its back.
const position = new kcds.math.Vector3(120, 200, 10.8);
const face = kcds.world.Prop.spawn('objects/kcds_cards/as.cgf', position);
face.showOnlyTo(player);
const back = kcds.world.Prop.spawn('objects/kcds_cards/back.cgf', position);
back.hideFrom(player);
```

```lua
-- A card only `player` sees the face of; everyone else sees its back.
local position = kcds.math.Vector3.new(120, 200, 10.8)
local face = kcds.world.Prop.spawn("objects/kcds_cards/as.cgf", position)
face:showOnlyTo(player)
local back = kcds.world.Prop.spawn("objects/kcds_cards/back.cgf", position)
back:hideFrom(player)
```

```cs
// A card only `player` sees the face of; everyone else sees its back.
var position = new Vector3(120, 200, 10.8);
var face = Prop.Spawn("objects/kcds_cards/as.cgf", position);
face.ShowOnlyTo(player);
var back = Prop.Spawn("objects/kcds_cards/back.cgf", position);
back.HideFrom(player);
```

`prop.visibleTo` and `prop.hiddenFrom` read them back.

### A "press E" hint on a prop

`World.addInteraction({ entity: prop, text })` puts the game's "press [key] to ..." hint on a prop. A player who aims
at the prop's mesh within `radius` metres sees it, and using it runs your handler. `key`, `mode` and the rest work as
on any other target ([Action hints](action-hints.md)).

```js
const { World, Prop } = kcds.world;
const { Chat } = kcds.core;

const barrel = Prop.spawn('barrel_a', new kcds.math.Vector3(120, 200, 10));
World.addInteraction({ entity: barrel, text: 'Open the barrel', radius: 2.5 }, (player) => {
  Chat.sendToPlayer(player, 'Only rainwater inside.');
});
```

```lua
local World, Prop = kcds.world.World, kcds.world.Prop
local Chat = kcds.core.Chat

local barrel = Prop.spawn("barrel_a", kcds.math.Vector3.new(120, 200, 10))
World.addInteraction({ entity = barrel, text = "Open the barrel", radius = 2.5 }, function(player)
  Chat.sendToPlayer(player, "Only rainwater inside.")
end)
```

```cs
var barrel = Prop.Spawn("barrel_a", new Vector3(120, 200, 10));
World.AddInteraction(new { entity = barrel, text = "Open the barrel", radius = 2.5 }, (JsValue player) =>
{
    Chat.SendToPlayer(player, "Only rainwater inside.");
});
```

### Gliding

`prop.glideTo(position, rotation?, { ms, arc })` moves a prop smoothly: every player's game glides it there over `ms`
milliseconds (400 by default, up to 10000), eased in and out and turning on the way, lifted on an arc `arc` metres
high at the middle (0 by default, a straight line; up to 5). The server sends the end once, not every step, so a
thrown card or a hopping game piece costs one message. For the server, and for a player who joins meanwhile, the
prop is at the end at once. A rigid prop moves by its physics and throws.

```js
// A card thrown onto the table half a metre away, turning as it flies.
const { Vector3 } = kcds.math;
const card = kcds.world.Prop.spawn('objects/kcds_cards/as.cgf', new Vector3(120, 200, 10.9));
card.glideTo(new Vector3(120, 200.5, 10.75), { x: 0, y: 0, z: 30 }, { ms: 450, arc: 0.12 });
```

```lua
-- A card thrown onto the table half a metre away, turning as it flies.
local Vector3 = kcds.math.Vector3
local card = kcds.world.Prop.spawn("objects/kcds_cards/as.cgf", Vector3.new(120, 200, 10.9))
card:glideTo(Vector3.new(120, 200.5, 10.75), { x = 0, y = 0, z = 30 }, { ms = 450, arc = 0.12 })
```

```cs
// A card thrown onto the table half a metre away, turning as it flies.
var card = Prop.Spawn("objects/kcds_cards/as.cgf", new Vector3(120, 200, 10.9));
card.GlideTo(new Vector3(120, 200.5, 10.75), new { x = 0, y = 0, z = 30 }, new { ms = 450, arc = 0.12 });
```

### How a model collides

`Prop.collisionOf(model)` reads this from the game's meshes:

| Result    | Meaning                                                                                                              |
| --------- | -------------------------------------------------------------------------------------------------------------------- |
| `'mesh'`  | collides as drawn                                                                                                    |
| `'proxy'` | collides only with simplified shapes, so a body can fall through what is drawn, as with the top of the witch's stake |
| `'none'`  | no collision, as with carpets, furs and hangers                                                                      |
| `null`    | the catalogue does not have this mesh                                                                                |

A prop that must hold someone up beyond its collision needs a hidden floor of its own (`visible = false`).

### The game's prefabs

A model of `prefab:` and a name builds one of the game's own prefabs: a house, a wall or a well, laid out as
the game lays it out in its villages, with its windows, roof and shadow. It is solid where it stands. Name it
by its path under the game's `Prefabs/` folder or by its file name:

```js
const { Vector3 } = kcds.math;

const house = kcds.world.Prop.spawn(
  'prefab:house_1s_4r_angled_a_dobesovicko',
  new Vector3(120, 200, 10),
  new Vector3(0, 0, 90),
);
console.info(house.meshName, house.meshGroup);
// house_1s_4r_angled_a_dobesovicko prefabs/structures/living/houses/unique/dobesovicko
```

```lua
local Vector3 = kcds.math.Vector3

local house = kcds.world.Prop.spawn(
  "prefab:house_1s_4r_angled_a_dobesovicko",
  Vector3.new(120, 200, 10),
  Vector3.new(0, 0, 90)
)
print(house.meshName, house.meshGroup)
```

```cs
var house = Prop.Spawn(
    "prefab:house_1s_4r_angled_a_dobesovicko",
    new Vector3(120, 200, 10),
    new Vector3(0, 0, 90));
Log.Info($"{house.MeshName} {house.MeshGroup}");
```

A prefab stays as the game made it:

| Rule                                                | What happens                                         |
| --------------------------------------------------- | ---------------------------------------------------- |
| physics other than `'static'`, a scale other than 1 | `TypeError`, at spawn and when changed               |
| `visible = false`                                   | `TypeError`                                          |
| moving it (`position`, `setPose`)                   | every player's game builds it again at the new place |
| a prefab that holds one the game lacks              | `RangeError`: the game cannot build it               |
| an unknown name                                     | `RangeError`                                         |

Many houses carry their doors as prefabs of their own (`house_1s_2r_a` does): each player's game builds those
doors from their own templates in their places, so the house stands with its doors. 1622 of the game's 1623
prefabs can be built, all 303 houses among them. The one left holds a prefab the game does not have:

```text
RangeError: the game cannot build the prefab exteriordecoration/forest/wild_beehive_01 while it runs: it holds a prefab the game does not have
```

A prefab cannot be previewed with `PropGhost` or hung on a body. The doors in a prefab are the same doors in every
player's game: once a player comes near them, `Door.all()` lists them and `door.setOpen()` opens them for
everyone, as with the level's own doors.

### Rigid props

The body of a `rigid` prop is simulated by one game: the nearest player's whose game shows it, however far away, chosen again each half second.
Everyone else sees the pose that game reports. If nobody's game shows it, because there are no players or `showOnlyTo` names nobody online, it stays
where it last rested. A distant game simulates it against terrain and whatever it has loaded there.

```js
const { Vector3 } = kcds.math;
const { Events, Chat } = kcds.core;

const barrel = kcds.world.Prop.spawn('barrel_a', new Vector3(100, 200, 12), {}, 1, 'rigid');
barrel.mass = 20; // this model's collision has no mass of its own

Events.on('playerCommand', (player, command) => {
  if (command !== 'kick') {
    return;
  }
  const away = barrel.position.sub(player.position).normalize();
  if (!barrel.applyImpulse(new Vector3(away.x * 60, away.y * 60, 90))) {
    Chat.sendToPlayer(player, 'Nobody is near enough to move it.');
  }
});

Events.on('propRest', (prop, position) => {
  console.info(`prop ${prop.id} rests at ${position}`);
});
```

```lua
local Vector3 = kcds.math.Vector3
local Events, Chat = kcds.core.Events, kcds.core.Chat

local barrel = kcds.world.Prop.spawn("barrel_a", Vector3.new(100, 200, 12), {}, 1, "rigid")
barrel.mass = 20 -- this model's collision has no mass of its own

Events.on("playerCommand", function(player, command)
  if command ~= "kick" then
    return
  end
  local away = barrel.position:sub(player.position):normalize()
  if not barrel:applyImpulse(Vector3.new(away.x * 60, away.y * 60, 90)) then
    Chat.sendToPlayer(player, "Nobody is near enough to move it.")
  end
end)

Events.on("propRest", function(prop, position)
  print("prop " .. prop.id .. " rests at " .. tostring(position))
end)
```

```cs
var barrel = Prop.Spawn("barrel_a", new Vector3(100, 200, 12), new { }, 1, "rigid");
barrel.Mass = 20; // this model's collision has no mass of its own

Events.On("playerCommand", (Player player, string command) =>
{
    if (command != "kick")
    {
        return;
    }
    var away = barrel.Position.Sub(player.Position).Normalize();
    if (!barrel.ApplyImpulse(new Vector3(away.X * 60, away.Y * 60, 90)))
    {
        Chat.SendToPlayer(player, "Nobody is near enough to move it.");
    }
});

Events.On("propRest", (Prop prop, Vector3 position) =>
{
    Log.Info($"prop {prop.Id} rests at {position}");
});
```

**Mass:** `mass` is in kilograms, 0 to 100,000. With 0, the default, the mesh's collision gives the mass. Most decoration has none, so a game asked to
simulate it keeps it static and logs `its collision carries no mass`. Give such a prop a mass.

**Moving it:**

| Call                                             | Effect                                                                                                                                                             |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `applyImpulse(impulse, point?, angularImpulse?)` | a push in kg m/s, which changes velocity by impulse divided by mass; a `point` off the centre of mass also turns it, and `angularImpulse` adds a twist in kg m^2/s |
| `setVelocity(velocity, angularVelocity?)`        | up to 200 m/s and 100 rad/s; leaving out the second stops it turning                                                                                               |
| `wake()`                                         | resumes the simulation of a resting prop, and it drops if unsupported                                                                                              |
| `sleep()`                                        | pauses the simulation at once, in the air too, until a hit or a script moves it                                                                                    |

All four return `false` if the prop no longer exists or is not rigid, if a value is invalid or out of range, or if no player is close enough to simulate
it.

**Events:**

- `propRest(prop, position, rotation)` fires each time the body stops, whatever moved it. A report over 100 m from where the prop last rested or spawned
  returns it there and fires nothing.
- `propHit(prop, speed, position)` fires each time the body strikes something, such as the ground, a wall or another body. The simulating game must see it
  moving at 2 m/s or more, then slowing with a velocity change of at least 2 m/s between frames. It reports the speed in m/s, at most once per 0.3 s per prop.

A barrel that breaks on a hard landing:

```js
kcds.core.Events.on('propHit', (prop, speed, position) => {
  if (speed >= 6 && prop.model.endsWith('barrel_a.cgf')) {
    prop.destroy();
  }
});
```

```lua
kcds.core.Events.on("propHit", function(prop, speed, position)
  if speed >= 6 and prop.model:sub(-#"barrel_a.cgf") == "barrel_a.cgf" then
    prop:destroy()
  end
end)
```

```cs
Events.On("propHit", (Prop prop, double speed, Vector3 position) =>
{
    if (speed >= 6 && prop.Model.EndsWith("barrel_a.cgf"))
    {
        prop.Destroy();
    }
});
```

**Stopping it:** `resetPhysics()` stops the physics and returns the prop to its spawn position for all players, with no `propRest`. Moving a rigid prop through
`position`, `rotation` or `setPose` also puts it to rest there. To fix it permanently, set `physics` to `'static'` after `propRest`.

## Things on a player's body

### Attached props

`player.attachProp({ model, joint, hand, position, rotation, scale })` hangs a mesh on a joint for as long as the script wants, such as cuffs on wrists, a
sack on a head or a noose on a neck. All players see it follow the joint, even while the player is cuffed.

```js
const sack = prisoner.attachProp({
  model: 'sack_a',
  joint: 'Head',
  position: new kcds.math.Vector3(0, 0.05, 0.1),
});

prisoner.detachProp(sack); // or sack.id

// the game's rope cuffs are held in the left hand slot
const cuffs = prisoner.attachProp({ model: 'rope_cuffs_front', hand: 'left' });
```

```lua
local sack = prisoner:attachProp({
  model = "sack_a",
  joint = "Head",
  position = kcds.math.Vector3.new(0, 0.05, 0.1),
})

prisoner:detachProp(sack) -- or sack.id
```

```cs
var sack = prisoner.AttachProp(new
{
    model = "sack_a",
    joint = "Head",
    position = new Vector3(0, 0.05, 0.1),
});

prisoner.DetachProp(sack); // or its id
```

- It returns a `Prop` with `attachedTo` and `joint`, and no physics.
- `position` and `rotation` are the offset from the joint, at most 5 m. Changing them moves the offset.
- `joint` is a joint such as `'Head'`, `'Neck'`, `'LeftHand'` or `'RightHand'`. On a client, `jointPosition` and `jointRotation` read where a joint is
  and how it is turned, so an offset can be worked out instead of tried.
- `hand: 'left'` or `'right'` hangs it in the game's own slot of that hand (the joints `LeftHandSlot` and `RightHandSlot`), where the game holds an item:
  its rope cuffs sit in the left one, and the game's item meshes are made for these slots, so they need no offset. Give a `joint` or a `hand`, not both.
- Without a joint, or with one the body lacks, the mesh hangs on the body itself. Its offset is then measured from the body's origin: on the ground
  between the feet, turned as the body faces (x to the right, y forward, z up). It follows where the body walks and turns, but not its animation, so it
  does not follow the hands or the head; for that, give their joint.
- It goes when the player leaves. `detachProp(prop)` removes it sooner, and returns true if it was this player's.

### The game's brand

`player.brand = 1` shows the game's brand on the left side of the neck, fresh and red. `2` shows it darker, as if healed, and `0` removes it. No mesh is needed,
and all players see it, including later joiners. It stays until a script changes it or the player leaves. It is only the mark, so for the game's penalties also
add the buff `crime_punishment_brand`.

### Marks on the skin

`player.addMark({ model, joint, position, rotation, scale })` puts a mark on the skin, such as a brand on a shoulder, a scar on a cheek or paint on a forehead. A
mark is a flat textured mesh, either your resource's or the game's, sitting just off the skin at a joint. All players see it move with the body, and it stays until
`removeMark(id)` or until the player leaves. It returns an id, and `player.marks` lists them.

```js
const shoulderBrand = thief.addMark({
  model: 'brand_mark', // your resource's flat mesh
  joint: 'RightArm',
  position: new kcds.math.Vector3(0.02, 0.06, 0.05),
  scale: 0.1,
});
thief.removeMark(shoulderBrand);
```

```lua
local shoulderBrand = thief:addMark({
  model = "brand_mark", -- your resource's flat mesh
  joint = "RightArm",
  position = kcds.math.Vector3.new(0.02, 0.06, 0.05),
  scale = 0.1,
})
thief:removeMark(shoulderBrand)
```

```cs
var shoulderBrand = thief.AddMark(new
{
    model = "brand_mark", // your resource's flat mesh
    joint = "RightArm",
    position = new Vector3(0.02, 0.06, 0.05),
    scale = 0.1,
});
thief.RemoveMark(shoulderBrand);
```

`joint` is required: `'Head'` for the face, `'Spine3'` for the chest and back, `'RightArm'` and `'LeftArm'` for the shoulders. The mesh does not bend with the skin, so
shape it to fit that spot or keep it small.

## PropGhost

`PropGhost.create(player, options?)`. A player can have up to 32 previews.

| Option                     | Meaning                                                                        |
| -------------------------- | ------------------------------------------------------------------------------ |
| `model` or `models`        | one model, or up to 16                                                         |
| `offsets`, `rotations`     | each piece's offset and rotation, in model order                               |
| `position`, `rotation`     | the place of the whole preview                                                 |
| `scale`                    | a number, or a per-axis `Vector3`; 0.01 to 100                                 |
| `tint`                     | a colour `{ r, g, b, a }` from 0 to 1, or `null` for the models' own materials |
| `validTint`, `invalidTint` | colours for `setValid(true)` and `setValid(false)`; green and red by default   |

**Read:** `id`, `player`, `valid`, `model`, `position`, `rotation`, `scale`, `hidden`.

**Methods:** `setPose(position, rotation, scale)`, leaving out values unchanged, `setTint(tint)`, `setValid(valid)`, `setHidden(hidden)` and `destroy()`. These return
`true` while the preview exists.

**Static:** `PropGhost.all(player?)`, `PropGhost.destroyAll(player?)`.

## PropPlacer

`PropPlacer.begin(player, options?)` returns a session, or `null` if the player already has one. `PropPlacer.get(player)` returns the running session or `null`.

| Option                                               | Meaning                                                                                                                                   |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `model` or `models`, `offsets`, `rotations`, `scale` | as for `PropGhost`                                                                                                                        |
| `validTint`, `invalidTint`                           | the preview's colours                                                                                                                     |
| `range`                                              | how far the player can aim, limited to 1 to 512                                                                                           |
| `fallbackDistance`, `verticalOffset`, `yaw`          | numbers                                                                                                                                   |
| `alignToSurface`, `faceAwayFromPlayer`, `keepOpen`   | flags                                                                                                                                     |
| `reportMoves`                                        | `true` fires `propPlaceMove`                                                                                                              |
| `rules`                                              | `maxSlope`, `minDistance`, `maxDistance`, `clearance` (numbers from 0), `surfaces`, `clearanceClasses` (lists of names), `requireSurface` |
| `spawn`                                              | `{ physics }`: the server builds a `Prop` on confirm, with one model only                                                                 |

**A session** has `id`, `player`, `active`, `model`, `end()`, and `veto(vetoed = true)`, which refuses all spots until cleared.

**Events:** `propPlaceMove(player, session, placement, valid)`, `propPlaceRejected(player, session, placement)`, `propPlaceCancel(player, session)`,
`propPlaceEnd(player, session)`, and `propPlaceConfirm(player, session, placement)`, where returning `false` refuses it.

**`placement`** has `model`, `position`, `rotation`, `scale`, `normal`, `surface` and `onSurface`.

The server refuses a spot outside the `minDistance`, `maxDistance` and `surfaces` rules.

## In client scripts

A client script can show ghosts and run a placement on its own machine, with no trip to the server for each move, which suits a camp builder or furniture editor. It has
the same preview, keys and rules. Reports go to the script, and the server still decides what gets built.

```js
// client
const { PropPlacer } = kcds.world;

PropPlacer.begin({
  model: 'objects/manmade/structures/living/tents/canopy_forest_a.cgf',
  rules: { maxSlope: 20, maxDistance: 8 },
});

PropPlacer.on('confirm', (pose) => {
  kcds.core.Events.toServer('pitchTent', { position: pose.position, rotation: pose.rotation });
});
PropPlacer.on('rejected', () => kcds.ui.Hud.showNotification('Not there.'));
```

```lua
-- client
local PropPlacer = kcds.world.PropPlacer

PropPlacer.begin({
  model = "objects/manmade/structures/living/tents/canopy_forest_a.cgf",
  rules = { maxSlope = 20, maxDistance = 8 },
})

PropPlacer.on("confirm", function(pose)
  kcds.core.Events.toServer("pitchTent", { position = pose.position, rotation = pose.rotation })
end)
PropPlacer.on("rejected", function() kcds.ui.Hud.showNotification("Not there.") end)
```

```cs
// client
PropPlacer.Begin(new
{
    model = "objects/manmade/structures/living/tents/canopy_forest_a.cgf",
    rules = new { maxSlope = 20, maxDistance = 8 },
});

PropPlacer.On("confirm", (JsValue pose) =>
{
    Events.ToServer("pitchTent", new { position = pose["position"], rotation = pose["rotation"] });
});
PropPlacer.On("rejected", () => Hud.ShowNotification("Not there."));
```

- **Placer:** `PropPlacer.begin(options)`, `end()`, `isActive()`, `setRefusal(vetoed)`, `rotate(degrees)`, `getPose()`, and `on(event, handler)` with `move`, `confirm`,
  `rejected`, `cancel` and `end`. Each handler gets `(pose, valid)`.
- **Ghost:** `PropGhost.create(options)`, with no player, then `setPose`, `setTint`, `setHidden` and `destroy`, plus `PropGhost.destroyAll()`.
- A ghost's `getBounds()` is its box `{ min, max }` once the game has built it, otherwise `null`. A pose's `surface` is currently always `''`.

### The game's prefabs on one machine

`Prefab` builds one of the game's prefabs in this game only, as an editor does while its author works. It stands
solid, with its doors, and no other player sees it:

```js
// client
const { Prefab } = kcds.world;
const { Vector3 } = kcds.math;

const [path] = Prefab.find('living/houses/house_1s_2r');
const house = Prefab.place(path, new Vector3(120, 200, 10), new Vector3(0, 0, 90));
house.setPose(new Vector3(124, 200, 10));
house.destroy();
```

```lua
-- client
local Prefab = kcds.world.Prefab
local Vector3 = kcds.math.Vector3

local path = Prefab.find("living/houses/house_1s_2r")[1]
local house = Prefab.place(path, Vector3.new(120, 200, 10), Vector3.new(0, 0, 90))
house:setPose(Vector3.new(124, 200, 10))
house:destroy()
```

```cs
// client
var path = Prefab.Find("living/houses/house_1s_2r")[0];
var house = Prefab.Place(path, new Vector3(120, 200, 10), new Vector3(0, 0, 90));
house.SetPose(new Vector3(124, 200, 10));
house.Destroy();
```

- `Prefab.place(name, position, orientation?)` takes the same names as `Prop.spawn('prefab:...')` and throws for
  the same ones.
- `Prefab.find(text, max?)` lists the buildable prefabs whose path contains the text (200 at most).
- `position`, `rotation` and `setPose()` build it again at the new place; `destroy()` takes it away.
- A game holds up to 2048 of them; they end when their resource stops or the world goes.

### Shaping the ground on one machine

`Terrain` changes the level's ground in this game only, so a building placed with `Prefab` stands on level
ground. The ground is solid at once where it now lies, and every edit can be undone:

```js
// client
const { Prefab, Terrain } = kcds.world;
const { Vector3 } = kcds.math;

// A flat pad 7 m across the middle, fading into the slope over 6 m more.
const pad = Terrain.flatten(new Vector3(120, 200, 0), 7, 14.5, 6);
const house = Prefab.place('house_1s_4r_angled_a_dobesovicko', new Vector3(120, 200, 14.5));

// A hill, then gone again.
const hill = Terrain.raise(new Vector3(140, 200, 0), 3, 4, 6);
hill.undo();
```

```lua
-- client
local Prefab, Terrain = kcds.world.Prefab, kcds.world.Terrain
local Vector3 = kcds.math.Vector3

-- A flat pad 7 m across the middle, fading into the slope over 6 m more.
local pad = Terrain.flatten(Vector3.new(120, 200, 0), 7, 14.5, 6)
local house = Prefab.place("house_1s_4r_angled_a_dobesovicko", Vector3.new(120, 200, 14.5))

-- A hill, then gone again.
local hill = Terrain.raise(Vector3.new(140, 200, 0), 3, 4, 6)
hill:undo()
```

```cs
// client
// A flat pad 7 m across the middle, fading into the slope over 6 m more.
var pad = Terrain.Flatten(new Vector3(120, 200, 0), 7, 14.5, 6);
var house = Prefab.Place("house_1s_4r_angled_a_dobesovicko", new Vector3(120, 200, 14.5));

// A hill, then gone again.
var hill = Terrain.Raise(new Vector3(140, 200, 0), 3, 4, 6);
hill.Call("undo");
```

- `Terrain.flatten(center, radius, height, falloff?)` brings the circle to one height; `Terrain.raise(center,
radius, amount, falloff?)` lifts it, or lowers it with a negative amount. The falloff is the radius by default.
- Each returns an edit with `undo()` and `valid`; `Terrain.edits()` lists the ones still in place.
- An edit reaches at most 90 m from its middle. A resource's edits are undone when it stops or the world goes.

### Trees and bushes on one machine

`Vegetation` plants any kind of vegetation the level has, in this game only. A plant stands solid and bends in
the wind like the level's own:

```js
// client
const { Vegetation } = kcds.world;
const { Vector3 } = kcds.math;

const trees = Vegetation.kinds().filter((path) => path.includes('/trees/'));
const fir = Vegetation.plant('abies_alba_forest_b', new Vector3(120, 200, 14), {
  scale: 1.2,
  turn: 30,
});
fir?.remove();
```

```lua
-- client
local Vegetation = kcds.world.Vegetation
local Vector3 = kcds.math.Vector3

local trees = {}
for _, path in ipairs(Vegetation.kinds()) do
  if path:find("/trees/", 1, true) then
    table.insert(trees, path)
  end
end
local fir = Vegetation.plant("abies_alba_forest_b", Vector3.new(120, 200, 14), { scale = 1.2, turn = 30 })
if fir then
  fir:remove()
end
```

```cs
// client
var trees = Vegetation.Kinds().Where((path) => path.Contains("/trees/")).ToArray();
var fir = Vegetation.Plant("abies_alba_forest_b", new Vector3(120, 200, 14), new { scale = 1.2, turn = 30 });
if (!fir.IsNull)
{
    fir.Call("remove");
}
```

- `Vegetation.kinds()` lists the level's kinds as mesh paths; `plant(kind, position, { scale, turn })` takes a
  kind's index, path or file name and returns `null` when the game cannot plant it.
- A plant has `remove()` and `valid`; `Vegetation.planted()` lists yours. They end when the resource stops or the
  world goes.

### Invisible collision on one machine

`Collision` adds solid boxes that nothing draws, in this game only: a wall that keeps players off a cliff edge, a
floor over a gap, or a ramp up to a door. They block players, horses and anything else the game's physics moves:

```js
// client
const { Collision } = kcds.world;
const { Vector3 } = kcds.math;

const wall = Collision.addBox({
  center: new Vector3(1533, 2143, 17.1),
  size: new Vector3(6, 0.3, 3),
  rotation: { x: 0, y: 0, z: 30 },
});
wall.set({ size: new Vector3(8, 0.3, 3) });
```

```lua
-- client
local Collision = kcds.world.Collision
local Vector3 = kcds.math.Vector3

local wall = Collision.addBox({
  center = Vector3.new(1533, 2143, 17.1),
  size = Vector3.new(6, 0.3, 3),
  rotation = { x = 0, y = 0, z = 30 },
})
wall:set({ size = Vector3.new(8, 0.3, 3) })
```

```cs
// client
var wall = Collision.AddBox(new
{
    center = new Vector3(1533, 2143, 17.1),
    size = new Vector3(6, 0.3, 3),
    rotation = new { x = 0, y = 0, z = 30 },
});
wall.Call("set", new { size = new Vector3(8, 0.3, 3) });
```

- `size` is the width, depth and height in metres along the box's own axes, 0.02 to 200 each; `rotation` is a
  quaternion or angles in degrees.
- A box has `center`, `size`, `rotation`, `valid`, `set({ center, size, rotation })` and `remove()`;
  `Collision.all()` lists yours. They end when the resource stops or the world goes.
- For collision every player has, use a script area with `collision` (see [Areas](areas.md#collision-walls)).

Also: `Vegetation.plantMany(list)` plants many at once, 256 to a question to the game, instead of one a frame. A
dropped stack's `itemClass` is its class GUID, `leftBy` the player who dropped it, and `name` the name it was spawned
with. `PropGhost.editorPreview(options)` is a see-through preview of any of the game's own meshes, as an editor shows
what it is about to place.

Reference: [Prop](/docs/api/prop), [PropGhost](/docs/api/propghost), [PropPlacer](/docs/api/propplacer),
[Prefab](/docs/api/prefab), [Terrain](/docs/api/terrain), [Vegetation](/docs/api/vegetation),
[Collision](/docs/api/collision).
