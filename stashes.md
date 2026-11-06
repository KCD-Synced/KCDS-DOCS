# Stashes

`kcds.world.Stash` is a container held by the server, and comes in two kinds:

- A **chest** stands in the world. All players in view see it as the game's own container, but only the
  player who opens it receives its contents.
- A **storage** has no chest. A script opens it for a player anywhere, which suits a bank or a shop's back
  room.

```js
const { Stash } = kcds.world;
const { Vector3 } = kcds.math;

const bank = Stash.createStorage();
bank.addItem({ item: 'apple', amount: 3 });

kcds.core.Events.on('playerReady', (player) => {
  bank.open(player); // opens the transfer screen
});

const chest = Stash.spawn(new Vector3(100, 200, 10));
chest.locked = true;
chest.canBePicked = true;
```

```lua
local Stash = kcds.world.Stash
local Vector3 = kcds.math.Vector3

local bank = Stash.createStorage()
bank:addItem({ item = "apple", amount = 3 })

kcds.core.Events.on("playerReady", function(player)
  bank:open(player) -- opens the transfer screen
end)

local chest = Stash.spawn(Vector3.new(100, 200, 10))
chest.locked = true
chest.canBePicked = true
```

```cs
var bank = Stash.CreateStorage();
bank.AddItem(new { item = "apple", amount = 3 });

Events.On("playerReady", (Player player) =>
{
    bank.Open(player); // opens the transfer screen
});

var chest = Stash.Spawn(new Vector3(100, 200, 10));
chest.Locked = true;
chest.CanBePicked = true;
```

## Creating and finding

| Call                                                 | Result                                                                 |
| ---------------------------------------------------- | ---------------------------------------------------------------------- |
| `Stash.spawn(position, orientation?, dimension?, options?)` | a chest in the world                                            |
| `Stash.createStorage(dimension?)`                    | a storage; nothing appears in the world                                |
| `Stash.find(guid, dimension?)`                       | a chest by GUID, with or without `0x`, or `null`                       |
| `Stash.nearest(position, maxDistance?, dimension?)`  | the nearest chest, skipping storages; within 5 m by default            |
| `Stash.all()`, `Stash.getById(id)`                   | all stashes, or one by id                                              |
| `Stash.destroyAll(dimension?)`                       | removes them and returns how many; bodies of players and NPCs stay     |

A chest stands exactly at the height you give it, so use the ground's height or it will float. Players get
its "Open" prompt within about 3 m with their crosshair on it.

## Using another model

Pass `options.model` to `Stash.spawn` to make a container look like something else. Players see that mesh
with the game's Search prompt, and `stash.model` reads the path.

```js
const { Stash } = kcds.world;
const { Vector3 } = kcds.math;

const cabinet = Stash.spawn(new Vector3(100, 202, 10), {}, undefined, {
  model: 'objects/manmade/common_furniture/cabinets/cabinet_fancy_d.cgf',
});
cabinet.addItem({ item: 'apple', amount: 2 });
```

```lua
local Stash = kcds.world.Stash
local Vector3 = kcds.math.Vector3

local cabinet = Stash.spawn(Vector3.new(100, 202, 10), {}, nil, {
  model = "objects/manmade/common_furniture/cabinets/cabinet_fancy_d.cgf",
})
cabinet:addItem({ item = "apple", amount = 2 })
```

```cs
var cabinet = Stash.Spawn(new Vector3(100, 202, 10), new { }, null, new
{
    model = "objects/manmade/common_furniture/cabinets/cabinet_fancy_d.cgf",
});
cabinet.AddItem(new { item = "apple", amount = 2 });
```

The path must be a `.cgf`, `.cga` or `.cdf` with forward slashes, at most 255 bytes. Anything else throws a
`TypeError`. A mesh without open and close animations does not visibly open. A client that cannot load the
mesh shows the default chest.

## Working with a stash

**Read only:** `guid`, `placedGuid` (of an adopted level chest), `lockedAtBoot`, `isStorage`, `serverLocked`,
`isOpen`, `heldById`, `itemCount`, `position`, `rotation`, and `name`, which is `spawned_stash_<number>`,
`level_chest_<placedGuid>`, or empty for a storage.

**Locks:** you can read and write the following. A locked stash cannot be opened.

| Property      | Values                                                              |
| ------------- | ------------------------------------------------------------------- |
| `locked`      | true or false                                                       |
| `canBePicked` | true or false                                                       |
| `lockLevel`   | 0 (easy) to 1 (hard), or `null` for the game's own                  |
| `lockStyle`   | `'common'` or `'fancy'`, shown by the minigame; `null` for the level's |

**Contents:** `getInventory()` returns `{ revision, items }` or `null`. `addItem(request)`,
`removeItem(request)` and `setInventory(request)` change it.

**Opening:** `open(player)` opens a storage for a player anywhere in their dimension, and a spawned or adopted
chest for a player within 5 m. It returns `false` if the stash is locked or busy, a chest is out of reach, the
player is not available, or a `stashInteract` handler refuses. Dead bodies and saddlebags do not open this way.

**Removing:** `destroy()`.

## While someone is using it

One player at a time can have a container open. Anyone else pressing the key stays put, keeps the same prompt
and sees "Someone else is using this". This is the [text](chat-locales.md) `stash.inUse`, which you can
translate:

```js
kcds.core.Locales.set('czech', { 'stash.inUse': 'Tuhle truhlu právě používá někdo jiný' });
```

```lua
kcds.core.Locales.set("czech", { ["stash.inUse"] = "Tuhle truhlu právě používá někdo jiný" })
```

```cs
Locales.Set("czech", new Dictionary<string, string> { ["stash.inUse"] = "Tuhle truhlu právě používá někdo jiný" });
```

When a script calls `open(player)` in this case, no notice shows, and it returns `false`.

## Events

`stashSpawn(stash)`, `stashDestroy(stash)`, `stashOpen(stash, player)`, `stashClose(stash, player, reason)`,
`stashUnlocked`, `stashInventoryChanged(stash, player, change)`, and `stashInteract(player, stash, 'open')`,
where returning `false` refuses it.

## The level's own containers

Sacks, barrels, wardrobes and chests placed by the level belong to the game, and each player's game fills them
as the level says.

- `Stash.levelChests(position?, radius?)` lists them, by default within 20 m, as `{ placedGuid, position,
  model, lockedAtBoot, canBePicked, lockLevel }`.
- `Stash.adopt(placedGuid)` hands one to the server. It starts empty and holds what the server gives it, keeps
  the level's lock until a script changes it, and opens like a server chest for all players.
- `destroy()` hands it back to the game.

```js
const { Stash } = kcds.world;

// This chest opens only with its own key.
const [closest] = Stash.levelChests(player.position, 3);
const chest = closest && Stash.adopt(closest.placedGuid);
if (chest) {
  chest.keyClass = kcds.players.Items.effective('key_custom')?.id ?? '';
  chest.canBePicked = false;
  chest.locked = true;
}
```

```lua
local Stash = kcds.world.Stash

-- This chest opens only with its own key.
local closest = Stash.levelChests(player.position, 3)[1]
local chest = closest and Stash.adopt(closest.placedGuid)
if chest then
  local key = kcds.players.Items.effective("key_custom")
  chest.keyClass = key and key.id or ""
  chest.canBePicked = false
  chest.locked = true
end
```

```cs
// This chest opens only with its own key.
var closest = Stash.LevelChests(player.Position, 3).FirstOrDefault();
var chest = closest.IsNull ? null : Stash.Adopt(closest["placedGuid"]);
if (chest != null)
{
    chest.KeyClass = Items.Effective("key_custom")["id"].AsString() ?? "";
    chest.CanBePicked = false;
    chest.Locked = true;
}
```

### Beehives

The server runs the level's beehives with no script needed. All players see the same honey, and what one takes
is gone for the others. A hive holds 10 honey and refills to that an hour after it was last topped up.

A hive becomes a server chest once a player first comes within 60 m. It then appears in `Stash.all()` and
`Stash.find(placedGuid)`, and scripts can change its stock like any other chest's. The level's other containers
stay with the game.

## Horse saddlebags

Each server horse has saddlebags held by the server. `horse.storage` is a `Stash` with the usual `getInventory`,
`addItem`, `removeItem`, `setInventory` and `stashInventoryChanged`.

```js
const satchelHorse = kcds.characters.Horse.spawn(position);
satchelHorse.giveTo(player);
satchelHorse.storage.addItem({ item: 'apple', amount: 3 });
```

```lua
local satchelHorse = kcds.characters.Horse.spawn(position)
satchelHorse:giveTo(player)
satchelHorse.storage:addItem({ item = "apple", amount = 3 })
```

```cs
var satchelHorse = Horse.Spawn(position);
satchelHorse.GiveTo(player);
satchelHorse.Storage.AddItem(new { item = "apple", amount = 3 });
```

- Only the owner sees the contents, on the Horse tab of the game's inventory (Move to horse).
- The server checks each move. It must be the owner, within 15 m of the horse, and within
  `horse.maxCarryWeight`. Without that setting, the game's own limit applies, and the server does not check it.
- The bags belong to the horse. `horse.storage.destroy()` does not remove them, and they go when the horse is
  destroyed. They have no lock, and `open(player)` returns `false`.
- A new owner from `giveTo` gets the contents, and the previous owner's game stops showing them.

## Another player's inventory

`Stash.playerInventory(player, { put })` is a view of a connected player's own inventory, for admins and
the like. Open it for another player with `open(viewer)`, from anywhere: their game shows the game's own
transfer screen with everything the player carries and wears.

```js
// /inv <id>: an admin goes through a player's pockets and kit.
kcds.core.Events.on('playerCommand', (admin, command, args) => {
  const target = kcds.players.Player.getById(Number(args[0]));
  if (command === 'inv' && admin.state.get('role') === 'admin' && target) {
    kcds.world.Stash.playerInventory(target, { put: true }).open(admin);
  }
});
```

```lua
kcds.core.Events.on("playerCommand", function(admin, command, args)
  local target = kcds.players.Player.getById(tonumber(args[1]))
  if command == "inv" and admin.state:get("role") == "admin" and target then
    kcds.world.Stash.playerInventory(target, { put = true }):open(admin)
  end
end)
```

```cs
Events.On("playerCommand", (Player admin, string command, string[] args) =>
{
    var target = Player.GetById(int.Parse(args[0]));
    if (command == "inv" && admin.State.Get("role").AsString() == "admin" && target != null)
    {
        Stash.PlayerInventory(target, new { put = true }).Open(admin);
    }
});
```

- What the viewer takes leaves the player, worn things too: the player's game takes off the armour,
  clothes or weapon taken. `playerInventoryTake(viewer, owner, { item, row, amount, worn })` can refuse
  each take, and `playerInventoryTaken(viewer, owner, items)` follows.
- With `put: true` the viewer can also give the player things from their own inventory
  (`playerInventoryGive`, which can refuse, then `playerInventoryGiven`). Without it, nothing goes in.
- What the player picks up, drops or uses meanwhile shows to the viewer at once.
- The player's keyring is not shown. Nobody opens a view of their own inventory, and one viewer has a
  view open at a time.
- The view goes when the player leaves or the resource stops, and `destroy()` removes it sooner.
  `stash.inventoryOf` is the player whose inventory it shows.

Reference: [Stash](/docs/api/stash).
