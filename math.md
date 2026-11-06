# Math types

`kcds.math` has `Vector2`, `Vector3`, `Vector4`, `Quaternion` and `Color`, on both the server and the
client. All of them run inside your resource, so reading or converting a value never contacts the
server.

## Vectors

Each vector has its components plus:

| Kind                 | Members                                                  |
| -------------------- | -------------------------------------------------------- |
| Changes the vector   | `add`, `sub`, `mul`, `div`, `normalize`, `lerp`, `set`   |
| Leaves it unchanged  | `dot`, `distance`, `length`, `lengthSquared`, `clone`    |
| Text                 | `toString`, `toJSON`                                     |
| Static               | `zero`, `one`                                            |

`Vector3` also has `cross` and the static directions `up`, `forward` and `right`.

Methods that change the vector return it, which lets you chain them. Call `clone()` first if you
still need the original. Dividing by zero throws.

```js
const { Vector3 } = kcds.math;

const gate = new Vector3(10, 20, 3);
const aboveGate = gate.clone().add(new Vector3(0, 0, 3));
const height = gate.distance(aboveGate); // 3
```

```lua
local Vector3 = kcds.math.Vector3

local gate = Vector3.new(10, 20, 3)
local aboveGate = gate:clone():add(Vector3.new(0, 0, 3))
local height = gate:distance(aboveGate) -- 3
```

```cs
var gate = new Vector3(10, 20, 3);
var aboveGate = gate.Clone().Add(new Vector3(0, 0, 3));
var height = gate.Distance(aboveGate); // 3
```

**World directions:** in the game world, Z points up and a character faces +Y. The static helpers
`Vector3.up()` and `Vector3.forward()` use another common convention, +Y and +Z, so write world
directions out yourself, for example `new Vector3(0, 0, 1)` for up.

## Quaternions

The constructor order is `(w, x, y, z)`. Available members are `mul`, `normalize`, `conjugate`,
`inverse`, `slerp`, `dot`, `rotateVector`, `toEuler`, `set`, `clone`, and the static `identity`,
`fromEuler` and `fromAxisAngle`.

**Units:** quaternion methods work in radians. Euler angles given to spawn and teleport calls are in
degrees.

```js
const { Quaternion, Vector3 } = kcds.math;

const quarterTurn = Quaternion.fromAxisAngle(new Vector3(0, 0, 1), Math.PI / 2);
const turned = quarterTurn.rotateVector(new Vector3(0, 1, 0));
```

```lua
local Quaternion, Vector3 = kcds.math.Quaternion, kcds.math.Vector3

local quarterTurn = Quaternion.fromAxisAngle(Vector3.new(0, 0, 1), math.pi / 2)
local turned = quarterTurn:rotateVector(Vector3.new(0, 1, 0))
```

```cs
var quarterTurn = Quaternion.FromAxisAngle(new Vector3(0, 0, 1), Math.PI / 2);
var turned = quarterTurn.RotateVector(new Vector3(0, 1, 0));
```

To get a point three metres in front of a player, rotate a forward offset by the player's rotation:

```js
const { Vector3 } = kcds.math;

const ahead = player.position.clone().add(player.rotation.rotateVector(new Vector3(0, 3, 0)));
```

```lua
local Vector3 = kcds.math.Vector3

local ahead = player.position:clone():add(player.rotation:rotateVector(Vector3.new(0, 3, 0)))
```

```cs
var ahead = player.Position.Clone().Add(player.Rotation.RotateVector(new Vector3(0, 3, 0)));
```

## Colours

A `Color` stores red, green, blue and alpha as numbers from 0 to 1.

- **Create:** `Color.fromRGB` takes bytes from 0 to 255. `Color.fromHex` takes six or eight hex
  digits, with or without `#`, and returns white for anything else.
- **Use:** `set`, `lerp`, `clone`, `toString` and `toJSON`. `toHex()` gives the hex form, and
  `toHex(true)` includes alpha.
- **Presets:** `white`, `black`, `red`, `green`, `blue`, `yellow`, `cyan`, `magenta`, `transparent`.

```js
const { Color } = kcds.math;

const banner = Color.fromRGB(20, 100, 220, 128);
const css = banner.toHex(true);
```

```lua
local Color = kcds.math.Color

local banner = Color.fromRGB(20, 100, 220, 128)
local css = banner:toHex(true)
```

```cs
var banner = Color.FromRGB(20, 100, 220, 128);
var css = banner.ToHex(true);
```

Reference: [Vector3](/docs/api/vector3), [Quaternion](/docs/api/quaternion),
[Color](/docs/api/color).
