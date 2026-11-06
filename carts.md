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
| body c, such as `wagon_c_empty` (driver walks) | 2    | `driver`                                                                                           |
| `cart_b` (driver walks)                      | 1      | `driver`                                                                                           |
| `cart_a` (one model, no horses)              | 0      | `driver`, `rightBack`, `leftBack`                                                                  |

A loaded wagon, such as `wagon_b_covered_sacks`, only has `driver` and `bench`.

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

Seating is animated. The player walks to where that seat's climb starts, at the back for the benches
and at the side for the front seats. They climb up with the game's clip and move along the bench. A
driver takes the reins from the front plank during the climb. A player 25 m or more away is first
placed at the start. Getting off with the use key runs the same steps in reverse.

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
returns the same one. It has no chest model. Players search it with the game's Loot prompt at the cart,
and `storage.open(player)` opens it for a player in reach of the cart. The `storage` property returns
it or `null`.

`removeStorage()` or destroying the cart removes the storage and its contents. `storage.destroy()` does
not detach it from the cart.

## Removing and finding carts

`destroy()`, `Cart.all(dimension?)`, `Cart.getById(id)`, `Cart.destroyAll(dimension?)`.

## Events

- `cartSpawn(cart)` and `cartDestroy(cart)`
- `cartEntering(cart, player, seat)`, where returning `false` refuses the seat
- `cartEnter(cart, player, seat)` and `cartExit(cart, player, seat)`

`cart.driverWalks` says whether the driver walks beside the horses rather than sitting at the front.

Reference: [Cart](/docs/api/cart).
