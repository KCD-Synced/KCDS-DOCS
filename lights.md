# Lights

`kcds.world.Light` adds point lights to the world, such as a lamp by a door or a dim glow in a crypt.

- **From the server:** everyone sees the light, including players who join later. It stays until you
  destroy it or its resource stops.
- **From a client script:** only that player sees it.

```js
const { Light } = kcds.world;
const { Vector3 } = kcds.math;

const lamp = Light.spawn(new Vector3(2293.2, 2026.0, 106.5), {
  color: { x: 1, y: 0.55, z: 0.2 },
  intensity: 2,
  radius: 8,
});

lamp.radius = 12; // all players see the new size
lamp.castShadows = true;
```

```ts
const { Light } = kcds.world;
const { Vector3 } = kcds.math;

const lamp = Light.spawn(new Vector3(2293.2, 2026.0, 106.5), {
  color: { x: 1, y: 0.55, z: 0.2 },
  intensity: 2,
  radius: 8,
});

lamp.radius = 12; // all players see the new size
lamp.castShadows = true;
```

```lua
local Light = kcds.world.Light
local Vector3 = kcds.math.Vector3

local lamp = Light.spawn(Vector3.new(2293.2, 2026.0, 106.5), {
  color = { x = 1, y = 0.55, z = 0.2 },
  intensity = 2,
  radius = 8,
})

lamp.radius = 12 -- all players see the new size
lamp.castShadows = true
```

```cs
var lamp = Light.Spawn(new Vector3(2293.2, 2026.0, 106.5), new
{
    color = new { x = 1, y = 0.55, z = 0.2 },
    intensity = 2,
    radius = 8,
});

lamp.Radius = 12; // all players see the new size
lamp.CastShadows = true;
```

## Creating a light

`Light.spawn(position, options?)` takes a `Vector3` or `{ x, y, z }` in metres. All options are
optional:

| Option        | Accepted values                                | Default                        |
| ------------- | ---------------------------------------------- | ------------------------------ |
| `color`       | `{ x, y, z }` as red, green, blue, each 0 to 1 | warm orange, `{ 1, 0.6, 0.3 }` |
| `intensity`   | 0 to 100, a multiplier for the colour          | 1                              |
| `radius`      | 0.1 to 100 metres                              | 5                              |
| `castShadows` | `true` or `false`                              | `false`                        |

Whether shadows really appear also depends on the player's game settings.

A number out of range throws `RangeError` and a value of the wrong type throws `TypeError`. In both
cases nothing is created or changed.

## Changing a light

You can read and assign `position`, `color`, `intensity`, `radius` and `castShadows`. Assignments go
through the same checks, and only real changes are sent to players. `destroy()` removes the light and
can safely be called twice. On the server, `new kcds.world.Light(id)` gives you a light by network id.

## Lights from a world export

A [world export](level-editing.md) can list lights as `"lights": [{ position, color, intensity,
radius, castShadows }]`. These belong to the server and last until it shuts down.
`WorldResource#lights` gives the ones that still exist.

## Lights in a client script

A client-side light lasts until you call `destroy()`, the resource stops or the world changes. Check
`valid` to see if it is still shining. A removed light keeps its last values for reading, and writing
to it has no effect.

```js
// client
const { Light } = kcds.world;
const { Vector3 } = kcds.math;

const feet = kcds.players.LocalPlayer.position;
const headLamp = Light.spawn(new Vector3(feet.x, feet.y, feet.z + 2), { radius: 4 });
```

```ts
// client
const { Light } = kcds.world;
const { Vector3 } = kcds.math;

const me = kcds.players.LocalPlayer;
if (me) {
  const feet = me.position;
  const headLamp = Light.spawn(new Vector3(feet.x, feet.y, feet.z + 2), { radius: 4 });
}
```

```lua
-- client
local Light = kcds.world.Light
local Vector3 = kcds.math.Vector3

local feet = kcds.players.LocalPlayer.position
local headLamp = Light.spawn(Vector3.new(feet.x, feet.y, feet.z + 2), { radius = 4 })
```

```cs
// client
var feet = LocalPlayer.Current!.Position;
var headLamp = Light.Spawn(new Vector3(feet.X, feet.Y, feet.Z + 2), new { radius = 4 });
```

### Room lights

The game lights its rooms with ambient lights: such a light sets the ambient light within its radius to its colour
times its intensity, without shine or shadows. In a client script, `ambient: true` makes one (it stays one: read
`ambient`). An intensity below 1 darkens what it covers and above 1 brightens it, so one in the middle of a building,
as wide as the building and warm, takes out the cold sky light a room open to the sky gets; the room's lamps then
light it. World Architect's **Room light** is one.

```js
// client
const room = kcds.world.Light.spawn(new kcds.math.Vector3(3066, 665, 58), {
  ambient: true,
  radius: 25,
  color: { x: 1, y: 0.75, z: 0.5 },
  intensity: 1.5,
});
```

```ts
// client
const room = kcds.world.Light.spawn(new kcds.math.Vector3(3066, 665, 58), {
  ambient: true,
  radius: 25,
  color: { x: 1, y: 0.75, z: 0.5 },
  intensity: 1.5,
});
```

```lua
-- client
local room = kcds.world.Light.spawn(kcds.math.Vector3.new(3066, 665, 58), { ambient = true, radius = 25, color = { x = 1, y = 0.75, z = 0.5 }, intensity = 1.5 })
```

```cs
// client
var room = Light.Spawn(new Vector3(3066, 665, 58), new { ambient = true, radius = 25, color = new { x = 1, y = 0.75, z = 0.5 }, intensity = 1.5 });
```

## Limit

The game shows at most 192 lights, counting both server and client ones. `Light.spawn` in a client
script throws once that client's scripts have 192 lights. If the game has no room for a light, its
`valid` becomes false.

Reference: [Light](/docs/api/light).
