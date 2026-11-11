# Inventory and items

The server holds each player's inventory, through `kcds.players.Inventory`.

## Naming items

Use an item's class GUID or its name in the game's item table, such as `apple`, `bread` or `arrow_crude`.
`Inventory.classes()` lists them.

`Items.model(item)` gives the mesh a class has on the ground, or `''` if the game's tables give none. Most of these
also work as props, which shows a sword on its own, held still:

```js
const { Items } = kcds.players;

kcds.world.Prop.spawn(Items.model('longSwordDuel'), position, undefined, undefined, 'none');
```

```ts
const { Items } = kcds.players;

kcds.world.Prop.spawn(Items.model('longSwordDuel'), position, undefined, undefined, 'none');
```

```lua
local Items = kcds.players.Items

kcds.world.Prop.spawn(Items.model("longSwordDuel"), position, nil, nil, "none")
```

```cs
Prop.Spawn(Items.Model("longSwordDuel"), position, null, null, "none");
```

## Changing an inventory

Each change is all or nothing, and answers `{ ok, code, revision, items }`. A refused change has `ok: false` and a
`code`, such as `invalidItem`, `invalidAmount`, `insufficientItems`, `inventoryCapacity`, `unknownItem` or
`staleRevision`.

```js
const answer = kcds.players.Inventory.add(player, { item: 'apple', amount: 3 });
if (!answer.ok) {
  console.log(`refused: ${answer.code}`);
}

// Shorter forms:
player.giveItem('bread', 2);
player.takeItem('apple', 1);
```

```ts
const answer = kcds.players.Inventory.add(player, { item: 'apple', amount: 3 });
if (!answer.ok) {
  console.log(`refused: ${answer.code}`);
}

// Shorter forms:
player.giveItem('bread', 2);
player.takeItem('apple', 1);
```

```lua
local answer = kcds.players.Inventory.add(player, { item = "apple", amount = 3 })
if not answer.ok then
  print("refused: " .. answer.code)
end

-- Shorter forms:
player:giveItem("bread", 2)
player:takeItem("apple", 1)
```

```cs
var answer = Inventory.Add(player, new { item = "apple", amount = 3 });
if (!answer["ok"].AsBool())
{
    Log.Info($"refused: {answer["code"]}");
}

// Shorter forms:
player.GiveItem("bread", 2);
await player.TakeItem("apple", 1);
```

| Call                                                    | Effect                                                      |
| ------------------------------------------------------- | ----------------------------------------------------------- |
| `Inventory.add(player, { item, amount, metadata? })`    | adds units to the row with the same properties, or a new row |
| `Inventory.remove(player, { units: [{ id, amount }] })` | takes units out of rows                                     |
| `Inventory.editRow(player, { id, metadata })`           | replaces a row's quality, condition and custom fields       |
| `Inventory.editCustom(player, { id, name?, description?, data? })` | sets a row's own name, description or data; `null` resets one |
| `Inventory.set(player, { items, quickslots? })`         | replaces everything and dresses the body; see [Equipment](equipment.md) |
| `Inventory.transfer(from, to, { units })`               | moves exact units between two players                       |

`metadata` accepts `quality`, `health` or `condition`, `poison`, `poisonCharges`, `onEquipBuffs`, `transient`, and
`custom: { name, description, data }`, where `data` is a plain object, not text.

A row's own `name` and `description` are what its player sees for that row in the inventory and in the item panel,
for every kind of item. Other rows of the same item keep the game's name. A renamed sword or coat is still the same
sword or coat when you equip it, fight with it or wear it.

```js
const { Inventory } = kcds.players;

const [first] = Inventory.get(player).items;
if (first) {
  Inventory.editCustom(player, { id: first.id, name: 'Oathkeeper', description: 'Sworn steel of the Rattay guard.' });
}
```

```ts
const { Inventory } = kcds.players;

const [first] = Inventory.get(player)?.items ?? [];
if (first) {
  Inventory.editCustom(player, { id: first.id, name: 'Oathkeeper', description: 'Sworn steel of the Rattay guard.' });
}
```

```lua
local Inventory = kcds.players.Inventory

local first = Inventory.get(player).items[1]
if first then
  Inventory.editCustom(player, { id = first.id, name = "Oathkeeper", description = "Sworn steel of the Rattay guard." })
end
```

```cs
var items = Inventory.Get(player)["items"];
if (items.Length > 0)
{
    Inventory.EditCustom(player, new { id = items[0]["id"], name = "Oathkeeper", description = "Sworn steel of the Rattay guard." });
}
```

`notify: false` in a request leaves out the "You received" notice.

### Reading

`Inventory.get(player)` returns `{ revision, items, quickslots, ready }`. Each row is `{ id, item, name, amount,
quality, health, condition, equipped, metadata }`. If you pass the `revision` you read with a change, the change is
refused if the inventory has changed in the meantime.

### Items that vanish on leaving

A row with `transient: true` leaves the inventory when its player disconnects, before `playerDisconnect` fires. So a
script saving the inventory at that point never saves it. It always gets its own row, and reads back with
`metadata.transient`. If you save at another time, you can leave such rows out yourself.

```js
const { Inventory } = kcds.players;

// A sack over the prisoner's head, gone if they leave the server.
Inventory.add(prisoner, {
  item: 'BagOverHead_m01',
  amount: 1,
  metadata: { transient: true },
});

kcds.core.Events.on('playerDisconnect', (player) => {
  // The sack is not in what gets saved here.
  console.info(JSON.stringify(Inventory.get(player)?.items));
});
```

```ts
const { Inventory } = kcds.players;

// A sack over the prisoner's head, gone if they leave the server.
Inventory.add(prisoner, {
  item: 'BagOverHead_m01',
  amount: 1,
  metadata: { transient: true },
});

kcds.core.Events.on('playerDisconnect', (player) => {
  // The sack is not in what gets saved here.
  console.info(JSON.stringify(Inventory.get(player)?.items));
});
```

```lua
local Inventory = kcds.players.Inventory

-- A sack over the prisoner's head, gone if they leave the server.
Inventory.add(prisoner, {
  item = "BagOverHead_m01",
  amount = 1,
  metadata = { transient = true },
})

kcds.core.Events.on("playerDisconnect", function(player)
  -- The sack is not in what gets saved here.
  local current = Inventory.get(player)
  print(json.encode(current and current.items))
end)
```

```cs
// A sack over the prisoner's head, gone if they leave the server.
Inventory.Add(prisoner, new
{
    item = "BagOverHead_m01",
    amount = 1,
    metadata = new { transient = true },
});

Events.On("playerDisconnect", (Player player) =>
{
    // The sack is not in what gets saved here.
    Log.Info(Inventory.Get(player)["items"].ToJson());
});
```

### Events

- `playerInventoryReady`: their game now shows the server's stock
- `playerInventoryChanged`: with the reason for the change
- `playerItemUsed`: food, potions, or ammunition spent

### What the player's game changes

Some changes start in the player's game: the player eats, shoots an arrow, drops an item, wears a sword down or picks
something up. Their game sends these to the server in batches, one at a time, and the server answers each batch:

- **Applied**: the server's stock took the changes. Each `playerInventoryChanged` from the batch carries `operation`,
  the batch's id, such as `'3:17'`. It is the same for every change of one batch.
- **Refused**: the server's stock stays as it is, and the player's game is brought back to it.
- **Busy**: the server took too many reports this second. The game sends the same batch again shortly after, so no
  change is lost.

The server applies a batch only once, even when the game sends it twice. A batch from before a reconnect is refused,
because the stock the player's game shows after a reconnect comes from the server again.

```js
kcds.core.Events.on('playerInventoryChanged', (player, change) => {
  if (change.operation) {
    console.info(`${player.nickname}'s game: ${change.reason} (batch ${change.operation})`);
  }
});
```

```ts
kcds.core.Events.on('playerInventoryChanged', (player, change) => {
  if (change.operation) {
    console.info(`${player.nickname}'s game: ${change.reason} (batch ${change.operation})`);
  }
});
```

```lua
kcds.core.Events.on("playerInventoryChanged", function(player, change)
  if change.operation then
    print(player.nickname .. "'s game: " .. change.reason .. " (batch " .. change.operation .. ")")
  end
end)
```

```cs
Events.On("playerInventoryChanged", (Player player, JsValue change) =>
{
    if (change["operation"].Type == "string")
    {
        Log.Info($"{player.Nickname}'s game: {change["reason"]} (batch {change["operation"]})");
    }
});
```

### Before a player's action changes anything

`Inventory.beforeCommit(handler)` makes the actions of a player's own game wait for your verdict before they move
anything:

| Kind          | The player                                        | Also in the transaction |
| ------------- | ------------------------------------------------- | ----------------------- |
| `pickup`      | takes up an item lying on the ground              | `groundItem`            |
| `drop`        | drops units of an item                            |                         |
| `use`         | eats, drinks or applies an item, or a pot's meal  | `pot` for a meal        |
| `vendor`      | confirms a shop basket                            | `vendor`                |
| `trade`       | confirms a basket on another player's shelf       | `seller`                |
| `stashTake`   | takes from a container                            | `stash`                 |
| `stashPut`    | puts into a container                             | `stash`                 |
| `loot`        | takes from a body                                 | `stash`, `owner`        |
| `horseGear`   | puts tack on a horse or takes it off              | `horse`                 |

Each transaction has an `id`, the `player`, their inventory `revision`, `money` (what the player gets, negative for
what they pay) and `lines`: `{ item, name, amount, metadata, row, from, to }`, with `price` in a basket.

Return `true` or `false`, or a promise of either. You can also call `verdict(true)` or `verdict(false, reason)` later,
for example after asking a web service. Every handler must allow an action, and one that is not decided in 5 seconds
is refused. A refused action changes nothing, and the player's game shows what the server holds again.

A player's actions wait in line and commit one after the other. Each one commits against the inventories as they are
then, so an action that no longer fits is refused by the usual rules. Without a handler, actions commit at once.

```js
const { Inventory } = kcds.players;

// Shop baskets wait until the bank's books took them.
Inventory.beforeCommit((transaction, verdict) => {
  if (transaction.kind !== 'vendor') {
    return true;
  }
  fetch('https://bank.example.com/hold', {
    method: 'POST',
    body: JSON.stringify({ id: transaction.id, money: transaction.money }),
  })
    .then((response) => verdict(response.ok))
    .catch(() => verdict(false, 'bank unreachable'));
});
```

```ts
const { Inventory } = kcds.players;

// Shop baskets wait until the bank's books took them.
Inventory.beforeCommit((transaction, verdict) => {
  if (transaction.kind !== 'vendor') {
    return true;
  }
  fetch('https://bank.example.com/hold', {
    method: 'POST',
    body: JSON.stringify({ id: transaction.id, money: transaction.money }),
  })
    .then((response: { ok: boolean }) => verdict(response.ok))
    .catch(() => verdict(false, 'bank unreachable'));
});
```

```lua
local Inventory = kcds.players.Inventory

-- Players in the stocks can take up, buy and drop nothing.
local inStocks = {}

Inventory.beforeCommit(function(transaction, verdict)
  verdict(not inStocks[transaction.player.id])
end)
```

```cs
// Players in the stocks can take up, buy and drop nothing.
var inStocks = new HashSet<double>();

Inventory.BeforeCommit((JsValue transaction, JsValue verdict) =>
{
    verdict.Invoke(!inStocks.Contains(transaction["player"]["id"].AsNumber()));
});
```

`Inventory.stopBeforeCommit(id)` removes a handler; a resource's handlers also go when it stops.

#### Confirming the transaction afterwards

A transaction's `id` is unique, also across server restarts, so a database outside the server can keep its records
under it. After a transaction settles:

- every `playerInventoryChanged` its commit caused carries the id as `change.transaction`;
- then `playerInventoryTransaction(player, outcome)` fires once, with `{ id, kind, applied, code, reason, revision }`.
  `applied` is true when the action changed the inventories. Otherwise `code` says why: `refused` or `timeout` when
  the handlers refused it (`reason` holds the refusing handler's reason), or the action's own refusal, such as
  `inventoryCapacity`, when it no longer fitted. `revision` is the player's inventory revision afterwards.

So a hold made in `beforeCommit` can be confirmed or rolled back:

```js
const { Events } = kcds.core;

// The bank's hold becomes a booking, or is dropped.
Events.on('playerInventoryTransaction', (player, outcome) => {
  if (outcome.kind !== 'vendor') {
    return;
  }
  const action = outcome.applied ? 'confirm' : 'release';
  fetch(`https://bank.example.com/${action}`, {
    method: 'POST',
    body: JSON.stringify({ id: outcome.id }),
  }).catch(() => console.warn(`bank: ${action} ${outcome.id} failed`));
});
```

```ts
const { Events } = kcds.core;

// The bank's hold becomes a booking, or is dropped.
Events.on('playerInventoryTransaction', (player, outcome) => {
  if (outcome.kind !== 'vendor') {
    return;
  }
  const action = outcome.applied ? 'confirm' : 'release';
  fetch(`https://bank.example.com/${action}`, {
    method: 'POST',
    body: JSON.stringify({ id: outcome.id }),
  }).catch(() => console.warn(`bank: ${action} ${outcome.id} failed`));
});
```

```lua
local Events = kcds.core.Events

-- Each settled transaction in the server log.
Events.on("playerInventoryTransaction", function(player, outcome)
  print(outcome.id .. (outcome.applied and " applied" or " refused (" .. outcome.code .. ")"))
end)
```

```cs
// Each settled transaction in the server log.
Events.On("playerInventoryTransaction", (Player player, JsValue outcome) =>
{
    Log.Info($"{outcome["id"]} {(outcome["applied"].AsBool() ? "applied" : $"refused ({outcome["code"]})")}");
});
```

## Your own item types

`Items.register` creates a type that borrows the look of a miscellaneous item, crafting material or document. It works
with all inventory, stash and ground item calls. When no such item looks right, `model` gives it a mesh of the prop
catalogue instead, such as `model: 'manmade/common_furniture/bucket/bucket_a_a'`: the item lies on the ground as that
mesh, shows it in shops, and `Items.model` returns it. The icon still comes from `visual`, or from `icon`.

```js
const { Items } = kcds.players;

const sealedLetter = Items.register('myres:sealed_letter', {
  name: 'Sealed letter',
  description: 'A letter with the seal of the burgrave.',
  visual: 'crime_letterOfRecognition',
  weight: 0.1,
  usable: true,
  useLabel: 'Break the seal',
});
player.giveItem(sealedLetter);

kcds.core.Events.on('playerCustomItemUse', (player, row, revision) => {
  kcds.core.Chat.sendToPlayer(player, 'The seal breaks.');
});
```

```ts
const { Items } = kcds.players;

const sealedLetter = Items.register('myres:sealed_letter', {
  name: 'Sealed letter',
  description: 'A letter with the seal of the burgrave.',
  visual: 'crime_letterOfRecognition',
  weight: 0.1,
  usable: true,
  useLabel: 'Break the seal',
});
player.giveItem(sealedLetter);

kcds.core.Events.on('playerCustomItemUse', (player, row, revision) => {
  kcds.core.Chat.sendToPlayer(player, 'The seal breaks.');
});
```

```lua
local Items = kcds.players.Items

local sealedLetter = Items.register("myres:sealed_letter", {
  name = "Sealed letter",
  description = "A letter with the seal of the burgrave.",
  visual = "crime_letterOfRecognition",
  weight = 0.1,
  usable = true,
  useLabel = "Break the seal",
})
player:giveItem(sealedLetter)

kcds.core.Events.on("playerCustomItemUse", function(player, row, revision)
  kcds.core.Chat.sendToPlayer(player, "The seal breaks.")
end)
```

```cs
var sealedLetter = Items.Register("myres:sealed_letter", new
{
    name = "Sealed letter",
    description = "A letter with the seal of the burgrave.",
    visual = "crime_letterOfRecognition",
    weight = 0.1,
    usable = true,
    useLabel = "Break the seal",
});
player.GiveItem(sealedLetter);

Events.On("playerCustomItemUse", (Player player, JsValue row, double revision) =>
{
    Chat.SendToPlayer(player, "The seal breaks.");
});
```

## Changing the game's items

`Items.override(item, { weight, price, max, ...stats })` changes an item for all players, where `max` is the most units
a player can hold. `Items.clearOverride(item)` restores the game's values. To apply the same when the server starts, use
`resources/item-config.json`:

```json
{
  "arrow_crude": { "max": 40, "price": 20 },
  "bread": { "weight": 0.2 }
}
```

`Inventory.overrideStats(item, stats)` rewrites a class's combat numbers for everyone. A new set replaces the previous
one entirely. `Inventory.itemStats(item)` reads them, or returns `null` for a class that is unchanged.

```js
const { Inventory } = kcds.players;

Inventory.overrideStats('longSwordDuel', { attack: 210, strReq: 12 });
Inventory.overrideStats('SkullCap02_m02_C2', { defenseStab: 150, noise: 0.1 });
Inventory.overrideStats('longSwordDuel', null); // back to the game's numbers
```

```ts
const { Inventory } = kcds.players;

Inventory.overrideStats('longSwordDuel', { attack: 210, strReq: 12 });
Inventory.overrideStats('SkullCap02_m02_C2', { defenseStab: 150, noise: 0.1 });
Inventory.overrideStats('longSwordDuel', null); // back to the game's numbers
```

```lua
local Inventory = kcds.players.Inventory

Inventory.overrideStats("longSwordDuel", { attack = 210, strReq = 12 })
Inventory.overrideStats("SkullCap02_m02_C2", { defenseStab = 150, noise = 0.1 })
Inventory.overrideStats("longSwordDuel", null) -- back to the game's numbers
```

```cs
Inventory.OverrideStats("longSwordDuel", new { attack = 210, strReq = 12 });
Inventory.OverrideStats("SkullCap02_m02_C2", new { defenseStab = 150, noise = 0.1 });
Inventory.OverrideStats("longSwordDuel", null); // back to the game's numbers
```

Other calls: `Inventory.setWearRates({ default, armor, ... })` makes items wear faster or slower,
`Inventory.renameItem(item, name)` renames an item for everyone, and `Inventory.itemPrice` prices one unit as the game
does.

## Items on the ground

`kcds.world.GroundItem` is a stack in the world that all players see and can pick up. Items players drop and arrows
that land become ground items too.

```js
const { GroundItem } = kcds.world;

GroundItem.spawn('apple', player.position, {}, 5);
GroundItem.spawn('bread', player.position, {}, 1, undefined, { name: 'harvest_table' });

const breadStack = GroundItem.all().find((stack) => stack.name === 'harvest_table');

kcds.core.Events.on('groundItemPickup', (stack, player) => {
  console.info(`${player.nickname} took it`);
});
```

```ts
const { GroundItem } = kcds.world;

GroundItem.spawn('apple', player.position, {}, 5);
GroundItem.spawn('bread', player.position, {}, 1, undefined, { name: 'harvest_table' });

const breadStack = GroundItem.all().find((stack) => stack.name === 'harvest_table');

kcds.core.Events.on('groundItemPickup', (stack, player) => {
  console.info(`${player.nickname} took it`);
});
```

```lua
local GroundItem = kcds.world.GroundItem

GroundItem.spawn("apple", player.position, {}, 5)
GroundItem.spawn("bread", player.position, {}, 1, nil, { name = "harvest_table" })

local breadStack
for _, stack in ipairs(GroundItem.all()) do
  if stack.name == "harvest_table" then
    breadStack = stack
  end
end

kcds.core.Events.on("groundItemPickup", function(stack, player)
  print(player.nickname .. " took it")
end)
```

```cs
GroundItem.Spawn("apple", player.Position, new { }, 5);
GroundItem.Spawn("bread", player.Position, new { }, 1, null, new { name = "harvest_table" });

var breadStack = GroundItem.All().FirstOrDefault((stack) => stack.Name == "harvest_table");

Events.On("groundItemPickup", (GroundItem stack, Player player) =>
{
    Log.Info($"{player.Nickname} took it");
});
```

### Landing

A stack starts where it was spawned or dropped, and the game's physics lets it fall and roll. When the first player's
game with a view of it sees it stop, it stays at that spot for everyone: `resting` turns true, `position` is where it
lies, and `groundItemResting` fires. To wait for landing, listen for that event rather than polling the position:

```js
const { GroundItem } = kcds.world;

const droppedApple = GroundItem.spawn('apple', { x: 1650, y: 2151, z: 40 });

kcds.core.Events.on('groundItemResting', (landed) => {
  if (landed === droppedApple) {
    console.info(`the apple landed at ${landed.position.z.toFixed(2)}`);
  }
});
```

```ts
const { GroundItem } = kcds.world;

const droppedApple = GroundItem.spawn('apple', { x: 1650, y: 2151, z: 40 });

kcds.core.Events.on('groundItemResting', (landed) => {
  if (landed === droppedApple) {
    console.info(`the apple landed at ${landed.position.z.toFixed(2)}`);
  }
});
```

```lua
local GroundItem = kcds.world.GroundItem

local droppedApple = GroundItem.spawn("apple", { x = 1650, y = 2151, z = 40 })

kcds.core.Events.on("groundItemResting", function(landed)
  if landed == droppedApple then
    print(string.format("the apple landed at %.2f", landed.position.z))
  end
end)
```

```cs
var droppedApple = GroundItem.Spawn("apple", new { x = 1650, y = 2151, z = 40 });

Events.On("groundItemResting", (GroundItem landed) =>
{
    if (landed == droppedApple)
    {
        Log.Info($"the apple landed at {landed.Position.Z:F2}");
    }
});
```

A landed arrow or bolt is at rest from the start. If it sticks in a target, wall or tree, it stays there for all
players, where the shooter's game saw it, and any player can pull it out once.

### For one player only

A stack can be shown to a single player, whose game alone lays it and who alone can take it, such as an item a shop
previews for a customer. `showOnlyTo(null)` shows it to everyone again. If that player leaves, nobody sees the stack
until it is shown to someone else.

```js
const preview = kcds.world.GroundItem.spawn('longSwordDuel', counter, {}, 1, undefined, {
  visibleTo: player,
});
```

```ts
const preview = kcds.world.GroundItem.spawn('longSwordDuel', counter, {}, 1, undefined, {
  visibleTo: player,
});
```

```lua
local preview = kcds.world.GroundItem.spawn("longSwordDuel", counter, {}, 1, nil, {
  visibleTo = player,
})
```

```cs
var preview = GroundItem.Spawn("longSwordDuel", counter, new { }, 1, null, new
{
    visibleTo = player,
});
```

To refuse a pick-up of the level's own items, use `worldItemPickup`; see [Time, weather and the world](world.md).

## In the player's game

On the client, `kcds.players.Inventory.whenUsed(item, handler)` takes over an item's actions on the inventory screen.
The handler receives the use, and the server decides what it does.

Also: `Inventory.getWearRates()` reads the wear rates set, `Items.setEquippedWeightPercent(percent)` sets how much of
what players wear counts toward their load (100 by default; `player.setEquippedWeightPercent` for one player), and in
a client script `Inventory.stopWhenUsed(id)` removes a `whenUsed` handler.

Reference: [Inventory](/docs/api/inventory), [Items](/docs/api/items), [GroundItem](/docs/api/grounditem).
