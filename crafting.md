# Crafting, cooking and gathering

## The game's smithies and alchemy tables

Players use the game's own stations and minigames, and the server follows along. In each dimension, one player at a
time can use a station.

```js
const { Events } = kcds.core;

Events.on('craftingStarting', (player, proposal) => player.level >= 5);

Events.on('craftingCompleted', (player, result) => {
  console.info(`${player.nickname} made ${result.recipe}`);
});
```

```ts
const { Events } = kcds.core;

Events.on('craftingStarting', (player, proposal) => player.level >= 5);

Events.on('craftingCompleted', (player, result) => {
  console.info(`${player.nickname} made ${result.recipe}`);
});
```

```lua
local Events = kcds.core.Events

Events.on("craftingStarting", function(player, proposal) return player.level >= 5 end)

Events.on("craftingCompleted", function(player, result)
  print(player.nickname .. " made " .. result.recipe)
end)
```

```cs
Events.On("craftingStarting", (Player player, JsValue proposal) => player.Level >= 5);

Events.On("craftingCompleted", (Player player, JsValue result) =>
{
    Log.Info($"{player.Nickname} made {result["recipe"]}");
});
```

Returning `false` from `craftingStarting` or `craftingCompleting` refuses the step, and a refused product is taken
back out of the inventory. `craftingRefunding` lets you return the materials of an unfinished batch.

**Calls:** `Crafting.stations(kind?)`, `Crafting.session(player)`, `Crafting.occupancy(stationKey)`,
`Crafting.cancel(player)`. Recipes are in [Recipe](/docs/api/recipe), and a player learns one with
`player.learnRecipe(id)`.

**What others see:** other players see and hear the work, including the smith's tools, workpiece, sparks and forge, the
alchemy table's items, and a grindstone's wheel, sparks and sound.

**No fighting at stations:** a player at a smithy, alchemy table or grindstone cannot be fought, as in a dice match.
A blow or arrow to or from them is refused before it takes health, and `playerHit` does not fire. NPCs fighting them
stop when they sit down, and an NPC cannot be ordered to attack them.

## Adding recipes to the game's book and smithy

`Crafting.defineRecipe(definition)` adds an alchemy or smithing recipe to the game's tables. Its fields match one row of
`AlchemyRecipe.xml` or `BlacksmithRecipes.xml`. All clients load it, including later joiners, and `Crafting.recipes`
lists it with `source: 'script'`. It lasts for the rest of the server's life, and registering the same definition again
does nothing.

```js
kcds.world.Crafting.defineRecipe({
  kind: 'smithing',
  id: 'guild_horseshoe',
  name: 'Guild horseshoe',
  icon: 'horseshoeFarmer',
  category: 'Other',
  workpiece: 'horseshoe',
  minSkill: 2,
  description: 'The smiths of the guild shoe their horses this way.',
  ingredients: [{ item: '94d8f5f1-20a7-4840-98ad-1d198d381389', amount: 2 }],
  products: [{ item: '549ab26e-df73-43d6-ac9b-f4f74afec67f', threshold: 40 }],
});
```

```ts
kcds.world.Crafting.defineRecipe({
  kind: 'smithing',
  id: 'guild_horseshoe',
  name: 'Guild horseshoe',
  icon: 'horseshoeFarmer',
  category: 'Other',
  workpiece: 'horseshoe',
  minSkill: 2,
  description: 'The smiths of the guild shoe their horses this way.',
  ingredients: [{ item: '94d8f5f1-20a7-4840-98ad-1d198d381389', amount: 2 }],
  products: [{ item: '549ab26e-df73-43d6-ac9b-f4f74afec67f', threshold: 40 }],
});
```

```lua
kcds.world.Crafting.defineRecipe({
  kind = "smithing",
  id = "guild_horseshoe",
  name = "Guild horseshoe",
  icon = "horseshoeFarmer",
  category = "Other",
  workpiece = "horseshoe",
  minSkill = 2,
  description = "The smiths of the guild shoe their horses this way.",
  ingredients = { { item = "94d8f5f1-20a7-4840-98ad-1d198d381389", amount = 2 } },
  products = { { item = "549ab26e-df73-43d6-ac9b-f4f74afec67f", threshold = 40 } },
})
```

```cs
Crafting.DefineRecipe(new
{
    kind = "smithing",
    id = "guild_horseshoe",
    name = "Guild horseshoe",
    icon = "horseshoeFarmer",
    category = "Other",
    workpiece = "horseshoe",
    minSkill = 2,
    description = "The smiths of the guild shoe their horses this way.",
    ingredients = new[] { new { item = "94d8f5f1-20a7-4840-98ad-1d198d381389", amount = 2 } },
    products = new[] { new { item = "549ab26e-df73-43d6-ac9b-f4f74afec67f", threshold = 40 } },
});
```

**Items** are the game's item class GUIDs. Custom items from `Items.register` cannot be used, and a product is never a
quest item.

**Alchemy** needs a `base` and 1 to 28 `steps`. The first step identifies the recipe. Each step has a condition in the
game's syntax, joined by `AND`:

- `PotionBase()`
- `IngredientCount('<item>') == n` or `>= n`
- `IsMilled('<item>', 0 | 1)`
- `WeakBoilingTime('<item>', t)`, `StrongBoilingTime('<item>', t)`
- `IsBaseDistilled(0 | 1)`, `IsResultMilled(0 | 1)`

**Smithing** needs a `category` and a `workpiece`. `perk` is the perk that teaches it, and without one all smiths know it.

**Pictures** are base64 PNGs with 8 bits per channel, not interlaced, up to 4096x4096. They are fitted to size with the
aspect ratio kept and the rest transparent.

| Field                    | Used for                                                                    |
| ------------------------ | --------------------------------------------------------------------------- |
| `iconImage`              | the 64 x 64 item icon; without it, `icon` names a game item icon            |
| `productImage`           | alchemy book, 256 x 256 next to the description                             |
| `recipeImage`            | alchemy book, 1024 x 1024                                                   |
| `bookPicturesFrom`       | instead of those two, the `RecipeId` of a game alchemy recipe to borrow from |

Without any, the book shows the game's placeholder squares in their place. Clients receive pictures in parts after the
recipe, taking about a second for the large one.

### Alchemy recipes in the player's book

The book at an alchemy table lists a recipe only for players who know it through a perk, as the game does for its own
recipes. A recipe of yours needs a perk of its own and a row linking that perk to the recipe's id. Both are table patches
in your resource's `replace/` folder:

```text
my-alchemy/
  replace/Libs/Tables/rpg/perk__my_alchemy.xml          the perk
  replace/Libs/Tables/rpg/perk_recipe__my_alchemy.xml   the perk teaches recipe 910001
```

The perk is written like the game's own recipe perks: hidden from the perk screen (`visibility="0"`) and never given by
a level (`autolearnable="false"`), with a GUID of your own:

```xml
<?xml version="1.0" encoding="us-ascii"?>
<database xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="barbora" xsi:noNamespaceSchemaLocation="../database.xsd">
	<perks version="1">
		<perk autolearnable="false" perk_id="6b1f3c2e-951a-4d7e-9a51-c0de00910001" perk_name="Alch recipe - sage tonic" visibility="0" />
	</perks>
</database>
```

```xml
<?xml version="1.0" encoding="us-ascii"?>
<database xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="barbora" xsi:noNamespaceSchemaLocation="../database.xsd">
	<perk_recipes version="1">
		<perk_recipe perk_id="6b1f3c2e-951a-4d7e-9a51-c0de00910001" recipe_id="910001" />
	</perk_recipes>
</database>
```

When the resource starts, the server log says `recipe perks 6b1f3c2e-... -> 910001 added from perk_recipe__my_alchemy.xml`,
and the recipe's `perks` lists the perk. Players' games load both files with the game's own tables at their next start,
as they load [custom perks](custom-perks.md): a player who joins before that sees "This server has perks of its own.
Restart the game to have them." From then on, `player.learnRecipe(id)` or `Crafting.grantKnowledge(player, 910001,
536870911)` gives the perk, and the recipe is in their book with its name, usage text and steps. `Crafting.knowledge`
reports it whole. The same works for a row of your own `AlchemyRecipe__<name>.xml`.

**Errors:** it throws if the definition is malformed or names an unknown item, if the game or another resource has
the id, or if the id is already yours with other contents. The recipe's table row with its texts can be at most 40 KiB, which
28 long step texts can exceed.

## Your own recipes and stations

For custom crafting, the server does everything: it checks and takes materials, runs the progress and grants the products.

```js
const { Crafting } = kcds.world;

Crafting.addRecipe({
  id: 'smoked_boar',
  name: 'Smoked boar',
  ingredients: [{ item: 'boarMeat', amount: 2 }],
  products: [{ item: 'boarMeatSmoked', amount: 2 }],
  duration: 8,
});

const smokehouse = Crafting.addStation({
  position: new kcds.math.Vector3(100, 200, 10),
  recipes: ['smoked_boar'],
  text: 'Smoke meat',
});

kcds.core.Events.on('craftingStationUsed', (player, { station }) => {
  if (station === smokehouse) {
    Crafting.craft(player, station, 'smoked_boar');
  }
});
```

```ts
const { Crafting } = kcds.world;

Crafting.addRecipe({
  id: 'smoked_boar',
  name: 'Smoked boar',
  ingredients: [{ item: 'boarMeat', amount: 2 }],
  products: [{ item: 'boarMeatSmoked', amount: 2 }],
  duration: 8,
});

const smokehouse = Crafting.addStation({
  position: new kcds.math.Vector3(100, 200, 10),
  recipes: ['smoked_boar'],
  text: 'Smoke meat',
});

kcds.core.Events.on('craftingStationUsed', (player, { station }) => {
  if (station === smokehouse) {
    Crafting.craft(player, station, 'smoked_boar');
  }
});
```

```lua
local Crafting = kcds.world.Crafting

Crafting.addRecipe({
  id = "smoked_boar",
  name = "Smoked boar",
  ingredients = { { item = "boarMeat", amount = 2 } },
  products = { { item = "boarMeatSmoked", amount = 2 } },
  duration = 8,
})

local smokehouse = Crafting.addStation({
  position = kcds.math.Vector3.new(100, 200, 10),
  recipes = { "smoked_boar" },
  text = "Smoke meat",
})

kcds.core.Events.on("craftingStationUsed", function(player, use)
  if use.station == smokehouse then
    Crafting.craft(player, use.station, "smoked_boar")
  end
end)
```

```cs
Crafting.AddRecipe(new
{
    id = "smoked_boar",
    name = "Smoked boar",
    ingredients = new[] { new { item = "boarMeat", amount = 2 } },
    products = new[] { new { item = "boarMeatSmoked", amount = 2 } },
    duration = 8,
});

var smokehouse = Crafting.AddStation(new
{
    position = new Vector3(100, 200, 10),
    recipes = new[] { "smoked_boar" },
    text = "Smoke meat",
});

Events.On("craftingStationUsed", (Player player, JsValue use) =>
{
    if (use["station"].AsString() == smokehouse)
    {
        Crafting.Craft(player, smokehouse, "smoked_boar");
    }
});
```

- A station shows the game's "press [key] to ..." hint on a prop or a place. `animation` is what the crafter plays.
- With `autoProgress: false`, progress moves only through `Crafting.setProgress` and `addProgress`, for your own
  minigame.
- With `learn: true`, or a `sketch` usable item type, only players who learnt the recipe can craft it. See
  `Crafting.learnRecipe(player, id)`, `knowsRecipe` and `learnedRecipes`.

## Stew pots

`kcds.world.CookPot` covers the stew pots on the game's fireplaces. They hold stock per dimension: the meal (`goulash`,
`lentil` or `soup`), the meals left, whether eating is allowed, and any poison.

```js
const { CookPot } = kcds.world;

for (const pot of CookPot.all()) {
  CookPot.refill(pot.guid, 6);
}
```

```ts
const { CookPot } = kcds.world;

for (const pot of CookPot.all()) {
  CookPot.refill(pot.guid, 6);
}
```

```lua
local CookPot = kcds.world.CookPot

for _, pot in ipairs(CookPot.all()) do
  CookPot.refill(pot.guid, 6)
end
```

```cs
foreach (var pot in CookPot.All())
{
    CookPot.Refill(pot["guid"], 6);
}
```

Players can refill an empty pot from their inventory (`CookPot.topUpFrom`) or poison one (`CookPot.poison`).
`CookPot.setFire(guid, lit)` lights or puts out a pot's fire for all players and holds it that way, and `null` returns
control to each game. A pot's `fireLit` reads it.

## Herbs

`kcds.world.Gathering` handles the level's plants. Players pick them, and the server checks each pick and gives the herb
and 5 Survival XP. Picked plants grow back on the world clock.

```js
const { Gathering } = kcds.world;

Gathering.setRegrowth(6, 12); // grows back in 6 to 12 game hours

kcds.core.Events.on('gatheringHarvest', (player, pick) => player.dimension === 4294967295); // false refuses outside the shared dimension
```

```ts
const { Gathering } = kcds.world;

Gathering.setRegrowth(6, 12); // grows back in 6 to 12 game hours

kcds.core.Events.on('gatheringHarvest', (player, pick) => player.dimension === 4294967295); // false refuses outside the shared dimension
```

```lua
local Gathering = kcds.world.Gathering

Gathering.setRegrowth(6, 12) -- grows back in 6 to 12 game hours

-- false refuses outside the shared dimension
kcds.core.Events.on("gatheringHarvest", function(player, pick) return player.dimension == 4294967295 end)
```

```cs
Gathering.SetRegrowth(6, 12); // grows back in 6 to 12 game hours

// false refuses outside the shared dimension
Events.On("gatheringHarvest", (Player player, JsValue pick) => player.Dimension == 4294967295);
```

The server knows all the plants in its level, so it can answer with no player nearby:

| Call                                 | Effect                                                              |
| ------------------------------------ | ------------------------------------------------------------------- |
| `Gathering.herbsWithin(position, radius)` | lists pickable herbs in a sphere                               |
| `Gathering.clearHerbs(position, radius)` | removes them until `Gathering.respawn` or `regrowEverywhere`    |
| `area.herbsWithin()`, `area.clearHerbs()`, `area.regrowHerbs()` | the same for an area's shape             |

Each call takes a dimension, by default the shared one. For example, within 15 m of the inn yard at Zelejov the server
finds 16 dandelions, and one looks like this:

```json
{
  "kind": 22,
  "item": "a11cc7f6-b499-4003-aef1-938e87b30a2e",
  "position": { "x": 1650.13, "y": 2167.94, "z": 34.8 }
}
```

To make a garden where nothing grows wild:

```js
const garden = kcds.world.Area.getById('garden');

garden.clearHerbs();
console.info(`${garden.herbsWithin().length} herbs left`); // 0
```

```ts
const garden = kcds.world.Area.getById('garden');

if (garden) {
  garden.clearHerbs();
  console.info(`${garden.herbsWithin().length} herbs left`); // 0
}
```

```lua
local garden = kcds.world.Area.getById("garden")

garden:clearHerbs()
print(#garden:herbsWithin() .. " herbs left") -- 0
```

```cs
var garden = Area.GetById("garden")!;

garden.ClearHerbs();
Log.Info($"{garden.HerbsWithin().Length} herbs left"); // 0
```

A removed plant is gone for all players in that dimension, including later joiners, until a respawn or server restart.
Changing regrowth timing does not bring it back. The server knows the herbs and mushrooms of Trosecko and Kutnohorsko,
but not herbs a resource places itself.

### Performance

The server reads its level's plants once, at startup. Kutnohorsko's 2.4 million plants take about a third of a second
and 40 MB of the script host's memory, and players' ticks are not slowed. A query's cost grows with the plants inside:

| Sphere on Kutnohorsko | Plants   | Time on the script thread |
| --------------------- | -------- | ------------------------- |
| 50 m                  |          | about 0.2 ms              |
| 150 m                 | ~3,000   | about 2 ms                |
| 500 m                 | ~145,000 | about 100 ms              |

So use small spheres in handlers that run often.

Also: `Crafting.knowledge(player)` lists the alchemy recipes a player knows, each with a mask of the steps they know,
and `Crafting.grantKnowledge(player, recipeId, mask)` teaches one (the whole mask), forgets it (0) or teaches part of it
for this session. `Crafting.removeRecipe(id)` and `Crafting.removeStation(id)` remove your own recipes and stations.
`CookPot.setFood(guid, food, dimension)` changes the meal a pot serves, and `CookPot.poisons()` lists the poisons a pot
takes. `Gathering.setNaturalSpawns(false, kind)` turns the world's natural plants of a kind off for all players (all kinds
without one), `naturalSpawnsOff()` lists them, and `regrowTime()` and `resetRegrowth()` read and reset what
`setRegrowth` set.

Reference: [Crafting](/docs/api/crafting), [Recipe](/docs/api/recipe), [CookPot](/docs/api/cookpot),
[Gathering](/docs/api/gathering).
