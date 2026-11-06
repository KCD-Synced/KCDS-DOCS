# Siege

Three types work together in a siege: engines, ladders and piles of stones. The server decides what
happens to them, including an engine's cycle, crew, ammunition, health and the result of each shot.
Players' games draw them and report where a stone landed.

All three types have the same basics: the static `spawn`, `all(dimension?)`, `getById(id)` and
`destroyAll(dimension?)`, and the instance method `destroy()`.

```js
const { SiegeEngine } = kcds.world;
const { Vector3 } = kcds.math;
const { Events } = kcds.core;

const trebuchet = SiegeEngine.spawn('trebuchet', new Vector3(100, 200, 10));
trebuchet.ammo = 5;
trebuchet.load();

Events.on('siegeEngineReady', (engine) => {
  engine.fire(new Vector3(200, 200, 10));
});

Events.on('siegeImpact', (engine, where, attacker) => {
  console.info(`Stone landed at ${where.x}, ${where.y}`);
});
```

```ts
const { SiegeEngine } = kcds.world;
const { Vector3 } = kcds.math;
const { Events } = kcds.core;

const trebuchet = SiegeEngine.spawn('trebuchet', new Vector3(100, 200, 10));
trebuchet.ammo = 5;
trebuchet.load();

Events.on('siegeEngineReady', (engine) => {
  engine.fire(new Vector3(200, 200, 10));
});

Events.on('siegeImpact', (engine, where, attacker) => {
  console.info(`Stone landed at ${where.x}, ${where.y}`);
});
```

```lua
local SiegeEngine = kcds.world.SiegeEngine
local Vector3 = kcds.math.Vector3
local Events = kcds.core.Events

local trebuchet = SiegeEngine.spawn("trebuchet", Vector3.new(100, 200, 10))
trebuchet.ammo = 5
trebuchet:load()

Events.on("siegeEngineReady", function(engine)
  engine:fire(Vector3.new(200, 200, 10))
end)

Events.on("siegeImpact", function(engine, where, attacker)
  print("Stone landed at " .. where.x .. ", " .. where.y)
end)
```

```cs
var trebuchet = SiegeEngine.Spawn("trebuchet", new Vector3(100, 200, 10));
trebuchet.Ammo = 5;
trebuchet.Load();

Events.On("siegeEngineReady", (SiegeEngine engine) =>
{
    engine.Fire(new Vector3(200, 200, 10));
});

Events.On("siegeImpact", (SiegeEngine engine, Vector3 where, Player? attacker) =>
{
    Log.Info($"Stone landed at {where.X}, {where.Y}");
});
```

## Engines

`SiegeEngine.spawn(kind, position, orientation?, dimension?)`

| Kind          | Range         | Crew places |
| ------------- | ------------- | ----------- |
| `'trebuchet'` | 25 to 250 m   | 2           |
| `'cannon'`    | 10 to 200 m   | none        |

**Read only:** `cycle` (`idle`, `drawing`, `loading`, `ready`, `firing`), `loaded`, `position`,
`rotation`, `minRange`, `maxRange`, `reloadTime`, `wrecked`, `crewSeats`, `crew`, `operator`.

**Read and write:**

| Property      | Notes                                       |
| ------------- | ------------------------------------------- |
| `speed`       | 0.1 to 20                                   |
| `damage`      |                                             |
| `blastRadius` | 0 to 100                                    |
| `usable`      |                                             |
| `range`       | limited to what the engine can reach        |
| `maxHealth`   | also sets `health`                          |
| `health`      | 0 wrecks the engine                         |
| `ammo`        | a negative number means it never runs out   |
| `crewMinimum` | 0 to `crewSeats`                            |

**Methods:**

- `load()`, `fire(target, attacker?)` and `aim(target)` return `{ accepted, reason, seconds }`.
- `inReach(target)` returns `''` when the target can be hit, otherwise the reason.
- `showAimLine(enabled)`, `aimLine()` and `repair()`.
- `assignGunner(player)` gives a player the place. `null` frees it, and it returns `false` if the
  player has left.

**Events:** `siegeEngineSpawn`, `siegeEngineReady`, `siegeFire(engine, attacker, target)`,
`siegeImpact(engine, position, attacker)`, `siegeEngineDamage(engine, amount, attacker)`,
`siegeEngineWreck(engine, attacker)`, `siegeEngineDestroy`, and `siegeEngineUse(player, engine,
action)`. For `siegeEngineUse`, the action is `operate`, `leave`, `load` or `fire`, and returning
`false` refuses it.

## Ladders

`SiegeLadder.spawn(kind, position, orientation?, dimension?)`

| Kind    | Height  |
| ------- | ------- |
| `siege` | 9.75 m  |
| `tall`  | 5.5 m   |
| `short` | 3.25 m  |
| `low`   | 2 m     |

`SiegeLadder.describe(kind)` returns `{ kind, model, height, lean }`, or `null` for an unknown kind.

**Read only:** `pose` (`standing`, `pushed`, `down`, `raising`, `broken`), `height`, `climbable`,
`pusher`, `position`, `rotation`. **Read and write:** `usable`.

**Methods:** `push(player)` knocks over a standing ladder in 4.5 s and `raise(player)` puts up a fallen
one in 5.7 s. Both return `{ accepted, reason, seconds }`. There are also `break()` and `repair()`.

**Events:** `siegeLadderSpawn`, `siegeLadderUse(player, ladder, action)` with `push` or `raise`,
`siegeLadderFall(ladder, pusher)`, `siegeLadderRaise(ladder)`, `siegeLadderDestroy`.

## Stone piles

`StonePile.spawn(position, orientation?, dimension?)`

**Read only:** `position`, `rotation`.

**Read and write:**

| Property      | Accepted values                       | Default |
| ------------- | ------------------------------------- | ------- |
| `stones`      | a negative number becomes -1, an endless pile |         |
| `usable`      |                                       |         |
| `damage`      | 0 to 1000                             | 60      |
| `blastRadius` | 0 to 10                               | 1.5     |

A dropped stone hurts everyone within `blastRadius`, with full damage at the point of impact and a
quarter at the edge.

**Events:** `stonePileSpawn`, `stonePileThrow(player, pile)`, `stoneImpact(pile, position, player)`,
`stonePileDestroy`.

Reference: [SiegeEngine](/docs/api/siegeengine), [SiegeLadder](/docs/api/siegeladder),
[StonePile](/docs/api/stonepile).
