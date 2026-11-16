# Carts and wagons

`kcds.world.Cart` spawns the game's wagons and carts, with horses in harness, for players to ride. The
server stores the blueprint, location, dimension, who sits where and the top speed. The driver's game
does the driving and reports back.

```js
const { Cart } = kcds.world;
const { Vector3 } = kcds.math;
const { Events } = kcds.core;

const wagon = Cart.spawn('wagon_b_covered', new Vector3(100, 200, 10));
wagon.maxSpeed = 5;

// Refuse a second driver.
Events.on('cartEntering', (cart, player, seat) => {
  return seat !== 'driver' || cart.driver === null;
});

Events.on('playerReady', (player) => {
  wagon.seatPlayer(player, 'driver');
});
```

```ts
const { Cart } = kcds.world;
const { Vector3 } = kcds.math;
const { Events } = kcds.core;

const wagon = Cart.spawn('wagon_b_covered', new Vector3(100, 200, 10));
wagon.maxSpeed = 5;

// Refuse a second driver.
Events.on('cartEntering', (cart, player, seat) => {
  return seat !== 'driver' || cart.driver === null;
});

Events.on('playerReady', (player) => {
  wagon.seatPlayer(player, 'driver');
});
```

```lua
local Cart = kcds.world.Cart
local Vector3 = kcds.math.Vector3
local Events = kcds.core.Events

local wagon = Cart.spawn("wagon_b_covered", Vector3.new(100, 200, 10))
wagon.maxSpeed = 5

-- Refuse a second driver.
Events.on("cartEntering", function(cart, player, seat)
  return seat ~= "driver" or cart.driver == nil
end)

Events.on("playerReady", function(player)
  wagon:seatPlayer(player, "driver")
end)
```

```cs
var wagon = Cart.Spawn("wagon_b_covered", new Vector3(100, 200, 10));
wagon.MaxSpeed = 5;

// Refuse a second driver.
Events.On("cartEntering", (Cart cart, Player player, string seat) =>
{
    return seat != "driver" || cart.Driver == null;
});

Events.On("playerReady", (Player player) =>
{
    wagon.SeatPlayer(player, "driver");
});
```

## Blueprints and seats

`Cart.spawn(blueprint?, position, orientation?, dimension?)` uses `wagon_b_covered` by default and
throws on an unknown blueprint. `Cart.blueprints()` lists them: the game's 27 wagons and carts, from
its prefabs in `Prefabs/manmade/vehicles`, plus `cart_a`. The seats depend on the body:

| Blueprint                                    | Horses | Seats                                                                                              |
| -------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------- |
| covered body b, such as `wagon_b_covered`    | 2      | `driver`, `bench`, `rightBack`, `leftBack`, `rightFront`, `leftFront`, `rightMiddle`, `leftMiddle` |
| open body b, such as `wagon_b`               | 2      | `driver`, `bench`, `rightBack`, `leftBack`                                                         |
| body a, such as `wagon_a_chest`              | 2      | `driver`, `bench`                                                                                  |
| body c, such as `wagon_c_empty`              | 2      | `driver`, refused (archived, see below)                                                            |
| `cart_b`                                     | 1      | `driver`, refused (archived, see below)                                                            |
| `cart_a` (one model, no horses)              | 0      | `driver`, `rightBack`, `leftBack`                                                                  |

A loaded wagon, such as `wagon_b_covered_sacks`, only has `driver` and `bench`.

> **Archived: leading on foot.** A c wagon and `cart_b` were driven by a player walking beside their
> horses with the reins in hand. That is archived: their horses offer no **Lead** and their `driver`
> seat is refused, to scripts too. For a horse that walks after a player, use
> [`Horse.follow`](/docs/api/horse) with `rope: true`, which every player sees as a lead rope, such as
> `horse.follow(player, { radius: 2.5, speed: 'walk', rope: true })`.

## Properties

**Read only:** `id`, `blueprint`, `dimension`, `horseCount`, `seats`, `driver`, `position`, `rotation`,
and `pace`, which is `'stand'`, `'walk'`, `'trot'` or `'reverse'`.

**`maxSpeed`** can be read and written. It ranges from 0 to 15, with a default of 7. A value out of
range is ignored.

## Seating players

- `seatPlayer(player, seat)` seats a player and removes them from any other cart. It returns `false`
  if refused, and an unknown seat throws.
- `unseatPlayer(player)` takes a player off.
- `whoSits(seat)` returns a player or `null`, and `seatOf(player)` returns a seat or `null`.

A seat's entrance is at the back for the benches and at the side for the front seats. **Get on** takes the
free seat whose entrance is nearest the player, so getting on from behind takes a bench at the back and
from the front's right side the co-driver's. The player is put at that seat's climb start, turned to it,
and climbs on with the game's clip. Getting off with the use key plays the game's climb down. While
seated, the body is held on its seat, and nobody jumps while on a cart.

**Get on** shows only at a free seat's entrance: to a player within about 2 m of where its climb starts, so
nobody climbs on from the cart's side. `entranceOnly` set to `false` (or `Cart.spawn(..., { entranceOnly: false })`)
offers it to anyone at the cart's body, from any side.

`boardMode` set to `'walk'` (or `Cart.spawn(..., { boardMode: 'walk' })`) has the player walk round the cart to
the seat's climb start instead of being put there; on the way their use key or a movement key calls the
boarding off. `'teleport'` is the default.

## Hints

A cart shows the game's own hints: **Drive** and **Let go** on its horses, **Get on** on its body, and
**Search** (U) on its body while it has a storage. `Cart.setDefaultHints({ drive, letGo, getOn, search })`
changes their texts on every cart, and `cart.setHints(...)` on one. A text left out stays, and `null` hides
that hint. `World.addInteraction({ entity: cart, ... })` puts a script's own hint on the cart's body, shown
beside the cart's.

```js
const { Cart, World } = kcds.world;

Cart.setDefaultHints({ drive: 'Řídit', letGo: 'Pustit opratě', getOn: 'Nastoupit', search: 'Otevřít náklad' });

const wagon = Cart.spawn('wagon_b', new kcds.math.Vector3(100, 200, 10));
wagon.setHints({ getOn: null });
World.addInteraction({ entity: wagon, text: 'Board' }, (player) => {
  wagon.seatPlayer(player, 'bench');
});
```

```ts
const { Cart, World } = kcds.world;

Cart.setDefaultHints({ drive: 'Řídit', letGo: 'Pustit opratě', getOn: 'Nastoupit', search: 'Otevřít náklad' });

const wagon = Cart.spawn('wagon_b', new kcds.math.Vector3(100, 200, 10));
wagon.setHints({ getOn: null });
World.addInteraction({ entity: wagon, text: 'Board' }, (player) => {
  wagon.seatPlayer(player, 'bench');
});
```

```lua
local Cart, World = kcds.world.Cart, kcds.world.World

Cart.setDefaultHints({ drive = "Řídit", letGo = "Pustit opratě", getOn = "Nastoupit", search = "Otevřít náklad" })

local wagon = Cart.spawn("wagon_b", kcds.math.Vector3.new(100, 200, 10))
wagon:setHints({ getOn = nil })
World.addInteraction({ entity = wagon, text = "Board" }, function(player)
  wagon:seatPlayer(player, "bench")
end)
```

```cs
Cart.SetDefaultHints(new { drive = "Řídit", letGo = "Pustit opratě", getOn = "Nastoupit", search = "Otevřít náklad" });

var wagon = Cart.Spawn("wagon_b", new Vector3(100, 200, 10));
wagon.SetHints(new { getOn = (string?)null });
World.AddInteraction(new { entity = wagon, text = "Board" }, (Player player) =>
{
    wagon.SeatPlayer(player, "bench");
});
```

## Moving, loads and storage

`teleport(position, orientation?)` moves the cart with everyone on it.

`load` reads and sets what the cart visibly carries, from the game's own loaded carts. It is one of the
cart's `loads`, or `''` for nothing, which is the default. Any other value throws.

| Blueprint | Loads                                  |
| --------- | -------------------------------------- |
| `cart_b`  | `goods`, `goods2`, `sacks`             |
| `cart_a`  | `shadyGoods`, `shadyGoods2`, `branches` |
| others    | none                                   |

`addStorage()` gives the cart a storage and returns it as a [Stash](/docs/api/stash). Calling it again
returns the same one. It has no chest model. Players open it with the **Search** hint (U) on the cart's
body, and `storage.open(player)` opens it for a player in reach of the cart. The `storage` property
returns it or `null`.

`attachProp({ model, position, rotation, scale })` puts a prop on the cart that rides with it, such as logs
on its bed: `position` and `rotation` are an offset from the cart's root, its middle on the ground. The prop
has no physics, so it never holds the cart back, and it goes with the cart.

`removeStorage()` or destroying the cart removes the storage and its contents. `storage.destroy()` does
not detach it from the cart.

## Your own horses in the shafts

A cart can be drawn by real horses instead of its own. `cart.harness(horses)` puts them into the shafts of a
cart without horses: one `Horse` for a cart, an array of two for a wagon (`horseCount`). Each must be alive,
unridden, in no other cart, in the cart's world and within 15 m of it. While harnessed a horse leaves the
world (`horse.harnessedTo` names the cart) and the cart's horse wears its look. `cartHarness(cart, horses)`
comes first and may refuse it with `false`; `harness` returns `false` when it cannot.

`cart.harnessed` says whether horses are in its shafts, its own or harnessed ones, and `cart.horses` lists the
harnessed `Horse` entities, the left one first on a wagon. `cart.unharness()` takes them out: they stand
beside the cart, back in their own world, and the cart has no horses. It returns `false` when no horses are
harnessed or someone sits in the driver seat. `cartUnharness(cart, horses)` follows, also when the cart or one
of the horses is removed. A cart without horses cannot be driven, but its other seats can still be used.

## Removing and finding carts

`destroy()`, `Cart.all(dimension?)`, `Cart.getById(id)`, `Cart.destroyAll(dimension?)`.

## Events

- `cartSpawn(cart)` and `cartDestroy(cart)`
- `cartHarness(cart, horses)`, where returning `false` refuses the horses, and `cartUnharness(cart, horses)`
- `cartEntering(cart, player, seat)`, where returning `false` refuses the seat
- `cartEnter(cart, player, seat)` and `cartExit(cart, player, seat)`

`cart.driverWalks` (archived) says whether the blueprint is one whose driver walked beside the horses.

Reference: [Cart](/docs/api/cart).
