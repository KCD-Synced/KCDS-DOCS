# Felling trees

Players with an axe can chop down the level's trees, and this works without any script. After enough
blows, a tree falls away from the chopper in the same way on all screens and lies as a trunk. Further blows
cut the trunk into items, and later the tree grows back. `kcds.world.Trees` lets scripts change or refuse each
step.

A tree is any mesh under `objects/natural/vegetation/trees/`. Fallen branches and stumps do not count.

## From first blow to regrowth

**A swing counts as a blow** when all of these are true:

- the blade meets the tree, according to the game's weapon trace (`playerWeaponHit`)
- the player holds an axe, meaning a melee weapon named like one, or one of your `axes`
- they stand within `reach` metres, 3.5 by default
- they have not landed a blow in the last `swingSeconds`, 0.4 by default

**The tree falls** after `hits` blows, 8 by default. The standing tree and its collision are hidden in that
dimension, and its mesh falls as a prop, pivoting about its foot: slowly at first, quickly at the end, with a
small bounce. The server moves it, so everyone sees the same pose, and latecomers see it lying down. On a slope
the trunk rests along the ground.

**The trunk is cut** after `trunkHits` axe blows, 6 by default. The trunk is solid until then. It disappears
and gives `drops` to the chopper's inventory, or to the ground with `dropTo: 'ground'`.

**The tree grows back** after `regrowMinutes`, 30 by default, once no player is within `regrowClearance`
metres, 10 by default, so nobody sees it appear in front of them.

## Settings

```js
const { Trees } = kcds.world;

Trees.configure({
  hits: 5,
  trunkHits: 4,
  regrowMinutes: 60,
  drops: [{ item: 'bsmt_woodExotic', amount: 3 }],
});

// For one species, by the start of its mesh path:
Trees.setSpecies('objects/natural/vegetation/trees/normal_trees/quercus', {
  hits: 14,
  height: 22,
});
```

```ts
const { Trees } = kcds.world;

Trees.configure({
  hits: 5,
  trunkHits: 4,
  regrowMinutes: 60,
  drops: [{ item: 'bsmt_woodExotic', amount: 3 }],
});

// For one species, by the start of its mesh path:
Trees.setSpecies('objects/natural/vegetation/trees/normal_trees/quercus', {
  hits: 14,
  height: 22,
});
```

```lua
local Trees = kcds.world.Trees

Trees.configure({
  hits = 5,
  trunkHits = 4,
  regrowMinutes = 60,
  drops = { { item = "bsmt_woodExotic", amount = 3 } },
})

-- For one species, by the start of its mesh path:
Trees.setSpecies("objects/natural/vegetation/trees/normal_trees/quercus", {
  hits = 14,
  height = 22,
})
```

```cs
Trees.Configure(new
{
    hits = 5,
    trunkHits = 4,
    regrowMinutes = 60,
    drops = new[] { new { item = "bsmt_woodExotic", amount = 3 } },
});

// For one species, by the start of its mesh path:
Trees.SetSpecies("objects/natural/vegetation/trees/normal_trees/quercus", new
{
    hits = 14,
    height = 22,
});
```

| Setting           | Default                                    | Meaning                                       |
| ----------------- | ------------------------------------------ | --------------------------------------------- |
| `enabled`         | `true`                                     | whether trees can be chopped                  |
| `hits`            | 8                                          | blows to fell a tree                          |
| `trunkHits`       | 6                                          | blows to cut a trunk                          |
| `regrowMinutes`   | 30                                         | real minutes until regrowth                   |
| `regrowClearance` | 10                                         | metres that must be free of players           |
| `reach`           | 3.5                                        | metres from the chopper's feet to the blow    |
| `swingSeconds`    | 0.4                                        | shortest time between one player's blows      |
| `height`          | 15                                         | trunk length in metres at scale 1             |
| `trunkRadius`     | 0.35                                       | trunk radius in metres, for blows on it       |
| `drops`           | `[{ item: 'bsmt_woodExotic', amount: 2 }]` | items from a trunk                            |
| `dropTo`          | `'inventory'`                              | `'inventory'` or `'ground'`                   |
| `stump`           | `null`                                     | a prop model left where the tree stood        |
| `axes`            | `null`                                     | item classes that chop; `null` means all axes |

The game has no log or firewood item. To use one, register it with `kcds.players.Items.register` and put it
in `drops`.

## Events

Return `false` from a refusable server event to stop that step.

```js
const { Events, Chat } = kcds.core;
const { Vector3 } = kcds.math;

const sacredGrove = new Vector3(500, 500, 30);

Events.on('playerTreeChop', (player, tree, blow) => {
  if (tree.position.distance(sacredGrove) < 30) {
    return false;
  }
  Chat.sendToPlayer(player, `${blow.number} / ${blow.of}`);
});

Events.on('playerTreeFell', (player, tree, fall) => {
  fall.direction = new Vector3(0, 1, 0); // always falls north
});

Events.on('playerTrunkCut', (player, tree, items) => {
  items.push({ item: 'apple', amount: 1 });
});

// Trees return only by day.
Events.on('treeRegrow', () => kcds.world.World.hour >= 6);
```

```ts
const { Events, Chat } = kcds.core;
const { Vector3 } = kcds.math;

const sacredGrove = new Vector3(500, 500, 30);

Events.on('playerTreeChop', (player, tree, blow) => {
  if (tree.position.distance(sacredGrove) < 30) {
    return false;
  }
  Chat.sendToPlayer(player, `${blow.number} / ${blow.of}`);
});

Events.on('playerTreeFell', (player, tree, fall) => {
  fall.direction = new Vector3(0, 1, 0); // always falls north
});

Events.on('playerTrunkCut', (player, tree, items) => {
  items.push({ item: 'apple', amount: 1 });
});

// Trees return only by day.
Events.on('treeRegrow', () => kcds.world.World.hour >= 6);
```

```lua
local Events, Chat = kcds.core.Events, kcds.core.Chat
local Vector3 = kcds.math.Vector3

local sacredGrove = Vector3.new(500, 500, 30)

Events.on("playerTreeChop", function(player, tree, blow)
  if tree.position:distance(sacredGrove) < 30 then
    return false
  end
  Chat.sendToPlayer(player, blow.number .. " / " .. blow.of)
end)

Events.on("playerTreeFell", function(player, tree, fall)
  fall.direction = Vector3.new(0, 1, 0) -- always falls north
end)

Events.on("playerTrunkCut", function(player, tree, items)
  items:push({ item = "apple", amount = 1 })
end)

-- Trees return only by day.
Events.on("treeRegrow", function() return kcds.world.World.hour >= 6 end)
```

```cs
var sacredGrove = new Vector3(500, 500, 30);

Events.On("playerTreeChop", (Player player, JsValue tree, JsValue blow) =>
{
    if (tree["position"].As<Vector3>().Distance(sacredGrove) < 30)
    {
        return false;
    }
    Chat.SendToPlayer(player, $"{blow["number"]} / {blow["of"]}");
    return (bool?)null;
});

Events.On("playerTreeFell", (Player player, JsValue tree, JsValue fall) =>
{
    fall.Set("direction", new Vector3(0, 1, 0)); // always falls north
});

Events.On("playerTrunkCut", (Player player, JsValue tree, JsValue items) =>
{
    items.Call("push", new { item = "apple", amount = 1 });
});

// Trees return only by day.
Events.On("treeRegrow", () => World.Hour >= 6);
```

| Server event      | Arguments           | Refusable                      |
| ----------------- | ------------------- | ------------------------------ |
| `playerTreeChop`  | player, tree, blow  | yes                            |
| `playerTreeFell`  | player, tree, fall  | yes; `fall.direction` can change |
| `treeFelled`      | tree                | no                             |
| `playerTrunkChop` | player, tree, blow  | yes                            |
| `playerTrunkCut`  | player, tree, items | yes; `items` can change        |
| `treeRegrow`      | tree                | yes; delays it by a minute     |
| `treeRegrown`     | tree                | no                             |

Client scripts of players in the tree's dimension hear `treeChop(tree, blow)`, `treeFalling(tree, fall)`,
`treeFelled(tree)`, `trunkCut(tree, items)` and `treeRegrown(tree)`, useful for sounds, dust or a notice:

```js
// client
kcds.core.Events.on('treeFalling', (tree, fall) => {
  kcds.ui.Hud.showNotification(`Timber! (${fall.seconds.toFixed(1)} s)`);
});
```

```ts
// client
kcds.core.Events.on('treeFalling', (tree, fall) => {
  kcds.ui.Hud.showNotification(`Timber! (${fall.seconds.toFixed(1)} s)`);
});
```

```lua
-- client
kcds.core.Events.on("treeFalling", function(tree, fall)
  kcds.ui.Hud.showNotification(string.format("Timber! (%.1f s)", fall.seconds))
end)
```

```cs
// client
Events.On("treeFalling", (JsValue tree, JsValue fall) =>
{
    Hud.ShowNotification($"Timber! ({fall["seconds"].AsNumber():F1} s)");
});
```

## Working with a tree

A tree someone has chopped becomes a `Tree`, with these properties:

| Property                | Meaning                                                 |
| ----------------------- | ------------------------------------------------------- |
| `id`                    | `<mesh>@<x>,<y>,<z>`                                    |
| `species`               | its mesh                                                |
| `position`, `dimension` | where it is                                             |
| `hits`, `health`        | its blows                                               |
| `state`                 | `'standing'`, `'falling'`, `'felled'` or `'cut'`        |
| `felledAt`, `regrowAt`  | its times                                               |

Methods: `tree.fell(direction?)` fells it at once, `tree.regrow()` brings it back and `tree.setHits(n)` sets its
blows.

```js
const { Trees } = kcds.world;

// A storm fells the tree a client's ray found.
kcds.core.Events.fromClient('storm', (player, object) => {
  Trees.of(object)?.fell(new kcds.math.Vector3(1, 0, 0));
});

const fallenCount = Trees.felled().length;
Trees.regrowAll(); // restores the whole forest
```

```ts
const { Trees } = kcds.world;

// A storm fells the tree a client's ray found.
kcds.core.Events.fromClient('storm', (player, object) => {
  Trees.of(object)?.fell(new kcds.math.Vector3(1, 0, 0));
});

const fallenCount = Trees.felled().length;
Trees.regrowAll(); // restores the whole forest
```

```lua
local Trees = kcds.world.Trees

-- A storm fells the tree a client's ray found.
kcds.core.Events.fromClient("storm", function(player, object)
  local tree = Trees.of(object)
  if tree then
    tree:fell(kcds.math.Vector3.new(1, 0, 0))
  end
end)

local fallenCount = #Trees.felled()
Trees.regrowAll() -- restores the whole forest
```

```cs
// A storm fells the tree a client's ray found.
Events.FromClient("storm", (Player player, JsValue found) =>
{
    var tree = Trees.Of(found);
    if (!tree.IsNull)
    {
        tree.Call("fell", new Vector3(1, 0, 0));
    }
});

var fallenCount = Trees.Felled().Length;
Trees.RegrowAll(); // restores the whole forest
```

## Keeping trees across restarts

Nothing is saved automatically, so a restart brings back all trees. `Trees.state()` returns plain JSON you can
store, and `Trees.restore()` applies it. Felled trees lie where they lay, cut ones stay hidden, and regrowth times
continue.

```js
const fs = require('node:fs');
const path = require('node:path');

const { Trees } = kcds.world;
const { Events, Runtime } = kcds.core;
const forestFile = path.join(Runtime.serverDir, 'data', 'forest.json');

Events.on('resourceStop', () => {
  fs.mkdirSync(path.dirname(forestFile), { recursive: true });
  fs.writeFileSync(forestFile, JSON.stringify(Trees.state()));
});

Events.on('resourceStart', () => {
  if (fs.existsSync(forestFile)) {
    Trees.restore(JSON.parse(fs.readFileSync(forestFile, 'utf8')));
  }
});
```

```ts
const fs = require('node:fs');
const path = require('node:path');

const { Trees } = kcds.world;
const { Events, Runtime } = kcds.core;
const forestFile = path.join(Runtime.serverDir, 'data', 'forest.json');

Events.on('resourceStop', () => {
  fs.mkdirSync(path.dirname(forestFile), { recursive: true });
  fs.writeFileSync(forestFile, JSON.stringify(Trees.state()));
});

Events.on('resourceStart', () => {
  if (fs.existsSync(forestFile)) {
    Trees.restore(JSON.parse(fs.readFileSync(forestFile, 'utf8')));
  }
});
```

```lua
-- Lua scripts have no files, so a JavaScript resource of yours, "storage", keeps the data: its
-- exports save(name, value) and load(name) write and read it as above.
local Trees = kcds.world.Trees
local Events, Exports = kcds.core.Events, kcds.core.Exports

Events.on("resourceStop", function()
  Exports.get("storage", "save")("forest", Trees.state())
end)

Events.on("resourceStart", function()
  local saved = Exports.get("storage", "load")("forest")
  if saved ~= nil then
    Trees.restore(saved)
  end
end)
```

```cs
// C# scripts have no files, so a JavaScript resource of yours, "storage", keeps the data: its
// exports save(name, value) and load(name) write and read it as above.
Events.On("resourceStop", () =>
{
    Exports.Get("storage", "save").Invoke("forest", Trees.State());
});

Events.On("resourceStart", () =>
{
    var saved = Exports.Get("storage", "load").Invoke("forest");
    if (!saved.IsNull)
    {
        Trees.Restore(saved);
    }
});
```

<!-- only lua cs -->
A Lua or C# resource that keeps its data this way lists `storage` under `requires`, so `storage`
starts first and stops last.
<!-- end -->

`Trees.isTree(mesh)` says whether a mesh path is a tree that can be felled.

Reference: [Trees](/docs/api/trees), [Events](/docs/api/events).
