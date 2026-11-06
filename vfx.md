# Particle effects

`kcds.world.Vfx` plays the game's particle effects in two ways:

- **Spawned:** a lasting entity. All players in view have it until you destroy it, and changes are sent
  only when they happen.
- **Burst:** plays once for nearby players, and the server keeps no record of it.

```js
const { Vfx } = kcds.world;
const { Vector3 } = kcds.math;

const available = Vfx.list('WH_Particels.other');

const glow = Vfx.spawn('WH_Particels.other.magic', new Vector3(100, 200, 10), { scale: 2 });
glow.strength = 0.5; // rebuilds the effect

Vfx.burst('WH_Particels.other.gravel', new Vector3(100, 200, 10), { durationMs: 1500 });
```

```ts
const { Vfx } = kcds.world;
const { Vector3 } = kcds.math;

const available = Vfx.list('WH_Particels.other');

const glow = Vfx.spawn('WH_Particels.other.magic', new Vector3(100, 200, 10), { scale: 2 });
glow.strength = 0.5; // rebuilds the effect

Vfx.burst('WH_Particels.other.gravel', new Vector3(100, 200, 10), { durationMs: 1500 });
```

```lua
local Vfx = kcds.world.Vfx
local Vector3 = kcds.math.Vector3

local available = Vfx.list("WH_Particels.other")

local glow = Vfx.spawn("WH_Particels.other.magic", Vector3.new(100, 200, 10), { scale = 2 })
glow.strength = 0.5 -- rebuilds the effect

Vfx.burst("WH_Particels.other.gravel", Vector3.new(100, 200, 10), { durationMs = 1500 })
```

```cs
var available = Vfx.List("WH_Particels.other");

var glow = Vfx.Spawn("WH_Particels.other.magic", new Vector3(100, 200, 10), new { scale = 2 });
glow.Strength = 0.5; // rebuilds the effect

Vfx.Burst("WH_Particels.other.gravel", new Vector3(100, 200, 10), new { durationMs = 1500 });
```

## Effect names

Names have at least three parts separated by dots: `Library.group.name`. `Vfx.list(prefix?)` lists the
583 effects in the game's particle libraries, such as `WH_Particels`, `cinematics`, `collisions` and
`WH_vranik`. It works the same in client scripts.

An unknown name makes `spawn`, `burst` and `burstFrom` throw a `RangeError` on both sides, such as
`Unknown Vfx effect WH_Particels.fires.campfire_a`.

## Spawning

`Vfx.spawn(effect, position, options?, dimension?)`. Numbers out of range are moved to the nearest
allowed value.

| Option        | Accepted values                  | Default |
| ------------- | -------------------------------- | ------- |
| `rotation`    | a quaternion or angles           |         |
| `scale`       | 0.01 to 32                       | 1       |
| `countScale`  | 0.01 to 16                       | 1       |
| `speedScale`  | 0 to 16                          | 1       |
| `timeScale`   | 0.01 to 16                       | 1       |
| `strength`    | -1 to 1                          | -1      |
| `pulsePeriod` | 0 to 600                         | 0       |
| `prime`       | `true` starts it already running |         |
| `entity`      | a Player or Npc it attaches to   |         |
| `joint`       | a joint on that body, `'RightHand'` for example |         |

With `entity`, the effect follows the body or joint until it is destroyed or the player leaves. Then
`position` is an offset of up to 8 m. This is how you make a flame on a torch or smoke from a pipe.
Players who join later also see it.

**Read only:** `id`, `effect`, `library`, `group`, `dimension`, `position`, `rotation`.

**Read and write:** `scale`, `countScale`, `speedScale`, `timeScale`, `strength`, `pulsePeriod`. A
new `scale` resizes the running effect, so a fire can grow step by step. Any other change rebuilds the
effect for all players, so it starts again.

**Other members:** `destroy()`, `Vfx.all(dimension?)`, `Vfx.getById(id)`, and
`Vfx.destroyAll(dimension?)`, which returns how many it removed.

**Events:** `vfxSpawn(vfx)`, `vfxDestroy(vfx)`.

Effects have no physical body, so players walk through them. Up to version 0.3.4.1, an effect's entity
did have one, and players could bump into it.

## Bursts

- `Vfx.burst(effect, position, options?, dimension?)` plays the effect once for players in that
  dimension within 250 m, and returns how many there were. It takes the options above, plus
  `durationMs` from 0 to 60000, default 2000.
- `Vfx.burstFrom(effect, entity, options?)` does the same on an entity or its `id`, and the effect
  follows it. `options.offset` moves it by up to 8 m. `options.joint` attaches it to a joint of a player
  or NPC, such as `'RightHand'`, `'LeftHand'` or `'head'`, and the offset then counts from that joint.

## Blood effects

The `cinematics.blood.*` effects (`blood_blunt_a`, `blood_stab_a`, `blood_splash_small`,
`blood_leaking` and others) are the game's close-up wounds. They are at most 15 cm across and last well
under a second, except `blood_leaking`, which keeps dripping. On the ground a few metres away they are
barely visible and leave nothing behind.

Attach them to a body instead, and use a decal for blood that stays on the ground:

```js
const { Vfx, Marker } = kcds.world;
const { Vector3 } = kcds.math;

const wound = Vfx.spawn('cinematics.blood.blood_leaking', new Vector3(0, 0, 0), {
  entity: player,
  joint: 'Head',
});
const pool = Marker.place('materials/decals/decal_blood_a', player.position);
```

```ts
const { Vfx, Marker } = kcds.world;
const { Vector3 } = kcds.math;

const wound = Vfx.spawn('cinematics.blood.blood_leaking', new Vector3(0, 0, 0), {
  entity: player,
  joint: 'Head',
});
const pool = Marker.place('materials/decals/decal_blood_a', player.position);
```

```lua
local Vfx, Marker = kcds.world.Vfx, kcds.world.Marker
local Vector3 = kcds.math.Vector3

local wound = Vfx.spawn("cinematics.blood.blood_leaking", Vector3.new(0, 0, 0), {
  entity = player,
  joint = "Head",
})
local pool = Marker.place("materials/decals/decal_blood_a", player.position)
```

```cs
var wound = Vfx.Spawn("cinematics.blood.blood_leaking", new Vector3(0, 0, 0), new
{
    entity = player,
    joint = "Head",
});
var pool = Marker.Place("materials/decals/decal_blood_a", player.Position);
```

In a client script, `Vfx.isPlaying(effect)` says whether an effect your resource placed still plays, and `Vfx.stopAll()`
stops them all.

Reference: [Vfx](/docs/api/vfx).
