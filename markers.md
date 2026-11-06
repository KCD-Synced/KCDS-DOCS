# Markers and blips

- A **marker** is something drawn in the world, such as a decal or shape on the ground. It can also
  tell you when players walk in or out.
- A **blip** is an icon on the compass and map.

The server stores both and tells each player's game when to show, change or hide them.

```js
const { Marker, Blip } = kcds.world;
const { Vector3 } = kcds.math;
const { Events, Chat } = kcds.core;

const shopSpot = new Vector3(100, 200, 10);

const ring = Marker.place('materials/decals/ring', shopSpot, {
  shape: 'cylinder',
  size: 3,
  color: 0x00ff00ff,
  trigger: true,
});

Blip.create({ position: shopSpot, type: 'Shop', label: 'Shop' });

Events.on('markerEnter', (marker, player) => {
  if (marker.id === ring.id) {
    Chat.sendToAll(`${player.nickname} walked in`);
  }
});
```

```lua
local Marker, Blip = kcds.world.Marker, kcds.world.Blip
local Vector3 = kcds.math.Vector3
local Events, Chat = kcds.core.Events, kcds.core.Chat

local shopSpot = Vector3.new(100, 200, 10)

local ring = Marker.place("materials/decals/ring", shopSpot, {
  shape = "cylinder",
  size = 3,
  color = 0x00ff00ff,
  trigger = true,
})

Blip.create({ position = shopSpot, type = "Shop", label = "Shop" })

Events.on("markerEnter", function(marker, player)
  if marker.id == ring.id then
    Chat.sendToAll(player.nickname .. " walked in")
  end
end)
```

```cs
var shopSpot = new Vector3(100, 200, 10);

var ring = Marker.Place("materials/decals/ring", shopSpot, new
{
    shape = "cylinder",
    size = 3,
    color = 0x00ff00ff,
    trigger = true,
});

Blip.Create(new { position = shopSpot, type = "Shop", label = "Shop" });

Events.On("markerEnter", (Marker marker, Player player) =>
{
    if (marker.Id == ring.Id)
    {
        Chat.SendToAll($"{player.Nickname} walked in");
    }
});
```

## Marker

`Marker.place(material, position, options?, dimension?)`. The `material` name has 1 to 256 characters.

| Option     | Accepted values                                                     | Default   |
| ---------- | ------------------------------------------------------------------- | --------- |
| `shape`    | `'decal'`, `'cylinder'`, `'sphere'`, `'chevron'`, `'cube'`, `'plane'` | `'decal'` |
| `rotation` | a unit quaternion or angles                                         |           |
| `size`     | 0.25 to 64 (values outside are limited to this range)               | 2         |
| `height`   | 0.25 to 64                                                          | 2         |
| `spin`     | -720 to 720                                                         | 0         |
| `bob`      | 0 to 4                                                              | 0         |
| `color`    | an integer `0xRRGGBBAA`                                             | 0         |
| `trigger`  | `true` fires `markerEnter` and `markerExit`                         | `false`   |

**Read only:** `id`, `exists`, `material`, `group`, `shape`, `dimension`.

**Read and write:** `size`, `height`, `spin`, `bob`, `color`, `trigger`, `position`, `rotation`.

**Methods:** `setDimension(dimension)`, `destroy()`, and `showOnlyTo(target)` with a player, a list of
players, or `null` for everyone.

**Static:** `Marker.all(dimension?)`, `Marker.getById(id)`, `Marker.removeAll(dimension?)`.

**Events:** `markerPlace(marker)`, `markerRemove(marker)`, `markerEnter(marker, player)`,
`markerExit(marker, player)`.

**The trigger area** is an upright cylinder as wide as the marker. It runs from 1 m below the marker
to its height, or to 2 m for a decal or plane. If the marker is removed or `trigger` is switched off,
no exit event fires.

## Blip

`Blip.create(options)`

| Option      | Accepted values                                     | Default         |
| ----------- | --------------------------------------------------- | --------------- |
| `position`  | required                                            |                 |
| `type`      | a name from `Blip.types()`                          | `'GeneralPoi'`  |
| `markState` | an integer                                          | 0               |
| `label`     | up to 64 bytes of UTF-8; longer text is cut         |                 |
| `compass`   | show on the compass                                 | `true`          |
| `map`       | show on the map                                     | `true`          |
| `dimension` | the blip's dimension                                |                 |
| `player`    | a player or list of players who see it              | everyone        |

**Read only:** `id`, `exists`.

**Read and write:** `type`, `markState`, `label`, `compass`, `map`, `position`.

**Also:** `dimension`, `setDimension(dimension)`, `showOnlyTo(target)`, `destroy()`.

**Static:** `Blip.all(dimension?)`, `Blip.getById(id)`, `Blip.removeAll(dimension?)`, and
`Blip.types()`, which lists icon names such as `Shop`, `Home`, `Pub`, `Camp` and `FastTravel`.

A blip reaches all its players, however far away they are.

Reference: [Marker](/docs/api/marker), [Blip](/docs/api/blip).
