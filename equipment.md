# Equipment

The server holds what a player wears, as part of their inventory. Each row (`InventoryRow`) has `equipped`, the number of units the body
wears or holds. To dress or undress a player, change `equipped` through `kcds.players.Inventory.set`. Their game then dresses the body with
its own equipment code and reports what is really worn.

## Slots

You do not name a slot. The game chooses from the class's armour type, which `Inventory.classes()` gives as `slot`. It is a name from the table
below, or `null` for things the body does not wear, such as weapons, food, glasses, sword sheaths and horse gear.

```js
const slotOf = new Map(kcds.players.Inventory.classes().map((entry) => [entry.name, entry.slot]));

slotOf.get('HoseJoined01_m03_C'); // "leg_trousers"
slotOf.get('TunicShort05_m09_D'); // "body_cloth"
slotOf.get('longSwordDuel'); // null
```

```ts
const slotOf = new Map(kcds.players.Inventory.classes().map((entry) => [entry.name, entry.slot]));

slotOf.get('HoseJoined01_m03_C'); // "leg_trousers"
slotOf.get('TunicShort05_m09_D'); // "body_cloth"
slotOf.get('longSwordDuel'); // null
```

```lua
local slotOf = {}
for _, entry in ipairs(kcds.players.Inventory.classes()) do
  slotOf[entry.name] = entry.slot
end

print(slotOf["HoseJoined01_m03_C"]) -- "leg_trousers"
print(slotOf["TunicShort05_m09_D"]) -- "body_cloth"
print(slotOf["longSwordDuel"]) -- nil
```

```cs
var slotOf = Inventory.Classes().ToDictionary((entry) => entry["name"].AsString()!, (entry) => entry["slot"].AsString());

Log.Info(slotOf["HoseJoined01_m03_C"]); // "leg_trousers"
Log.Info(slotOf["TunicShort05_m09_D"]); // "body_cloth"
Log.Info(slotOf["longSwordDuel"] ?? "null"); // null
```

Most class names start with their armour type: `HoseJoined01_m03_C` is HoseJoined, and `BascinetVisor04_m02_A5` is BascinetVisor. Belts,
rings and necklaces have lower-case names, such as `belt_4slot`, `ring_golden_cross` and `necklace_silver_cross_insignia`.

### Body slots

Slots sharing a place, such as Cloth 1 and Cloth 2, take one piece at a time.

| Slot                  | Place      | Armour types                                                                                  | Needs                      |
| --------------------- | ---------- | --------------------------------------------------------------------------------------------- | -------------------------- |
| `head_coif`           | Coif 1     | CoifCap                                                                                       |                            |
| `head_coif_padded`    | Coif 2     | CoifSmall, CoifLarge, CoifMail                                                                |                            |
| `head_cap`            | Cap 1      | Cap, LeatherCap, F_Hood, F_Bonnet, F_CapAndWimple, F_Hat, F_HoodOpen, F_Veil, F_VeilAndWimple | `head_coif_padded` empty   |
| `head_helmet`         | Cap 2      | KettleHat, SkullCap, BascinetOpen, BascinetVisor                                              | `head_coif_padded` worn    |
| `head_hood`           | Hood       | Hood                                                                                          |                            |
| `collar`              | Collar     | CollarPadded, CollarMail                                                                      |                            |
| `body_cloth`          | Cloth 1    | TunicShort, TunicLong, F_SimpleDress, F_Smock, LeatherApron                                   |                            |
| `body_cloth_padded`   | Cloth 2    | GambesonShort, GambesonLong, Caftan, Pourpoint                                                |                            |
| `body_chainmail`      | Chainmail  | MailShort, MailLong                                                                           | `body_cloth_padded` worn   |
| `body_plate`          | Plate      | Brigandine, Cuirass                                                                           | `body_cloth_padded` worn   |
| `body_coat`           | Coat       | Coat, Waffenrock, Habit                                                                       |                            |
| `sleeves`             | Sleeves    | ArmBrigandine, ArmPlate                                                                       | `body_cloth_padded` worn   |
| `gloves`              | Gloves     | Gloves, Gauntlets                                                                             |                            |
| `leg_trousers`        | Trousers 1 | HoseJoined, HoseLoose, HoseSeparate                                                           |                            |
| `leg_trousers_padded` | Trousers 2 | LegsPadded, LegsChain                                                                         |                            |
| `leg_armor`           | LegArmor   | LegsBrigandine, LegsPlate                                                                     | `leg_trousers_padded` worn |
| `boot`                | Boot       | Shoes, BootsAnkle, BootsKnee, F_Shoes                                                         |                            |
| `spur`                | Spur       | Spurs                                                                                         | `boot` worn                |
| `belt`                | Belt       | Belt                                                                                          |                            |
| `pouch`               | Pouch      | Pouch                                                                                         |                            |
| `ring`                | Ring       | Ring                                                                                          |                            |
| `necklace`            | Necklace   | Necklace                                                                                      |                            |

The body wears one belt and one pouch. A belt with quick slots (`belt_4slot`) uses the `belt` slot, and a pouch (`Pouch_test_4slot`) uses
`pouch`. The apprentice and master badges, `Cap01_mApprentice` and `Cap01_mBlacksmith`, also use `belt`.

Horses have their own slots: `horse_head`, `horse_saddle`, `horse_torso` and `horse_shoe`. See [Horses and dogs](horses-dogs.md).

### A complete harness

The game can dress all of this in one `Inventory.set`: `LegsPadded01_m07_C3`, `GambesonLong01_m09_C3`, `CollarPadded01_m08_C1`,
`CoifSmall02_m07_E1`, `MailLong02_m01_C2`, `Cuirass07_m03_A4`, `ArmPlate03_m02_D1`, `LegsPlate02_m01_B4`, `Gauntlets05_m01_A5`,
`BascinetVisor04_m02_A5`, `Waffenrock08_mPisek_B`, `Spurs01_m01_C`, `belt_4slot`, `ring_golden_cross` and
`necklace_silver_cross_insignia`.

## Pieces that clash

The game would keep only some of a set of clashing pieces. So the server refuses such an `Inventory.set` with `invalidEquipment` before
anything changes. The answer does not say which rows clash.

Only rows with `equipped > 0` count, and each unit is a piece, so `equipped: 2` is two pieces. Weapons, ammunition and classes with `slot`
`null` are not counted.

**One piece per place:**

- two slots sharing a place: a shirt or a gambeson, hose or padded legs, a coif or a padded coif, a cap or a helmet
- two pieces in one slot, such as two shirts or two rings

**Other conflicts:**

- A cap is not worn over a padded coif.
- Plate legs with iron shoes, such as `LegsPlate02_m01_B4`, cover the feet, so shoes cannot go with them, though spurs can.

**Pieces needing another beneath:**

| Piece                            | Needs                                  |
| -------------------------------- | -------------------------------------- |
| a helmet                         | a padded coif                          |
| mail, plate or sleeves           | a gambeson                             |
| plate legs                       | padded legs                            |
| spurs                            | shoes, or plate legs with iron shoes   |

Put both in the same `Inventory.set`. The game dresses in two passes, so row order does not matter. When taking off a lower piece, take off
what depends on it too, such as a helmet with its padded coif.

**Swapping:** to put a piece on an occupied place, change both in one `Inventory.set`, using `equipped: 0` for the old and `equipped: 1` for the
new.

## Weapons and quick slots

Weapons and ammunition use `equipped` too. `quickslots` in `Inventory.set` decides which weapon sits in which quick slot. A belt adds weapon
slots and a pouch adds item slots.

`quickslots.weapons` holds four weapon sets, each `{ primary, secondary }` with row ids or `null`, and `activeWeapon` is the set the player
has selected (from 0), the one their weapon key draws. A body without a belt has only the first set: a weapon put in another one is left
out, and the client log says `the outfit cannot use the slot`.

**Arrows and bolts** are equipped in the set of the bow or crossbow that shoots them, as its `secondary`. That is what the game's own
Equip does with arrows, and only then does the game count them as equipped: the inventory marks the bow and the arrows, and the HUD shows
the bow with the number of arrows. Wearing the arrows (`equipped`) is not enough. The player draws the selected set with their weapon key.

`Player.equip` does this for you: a weapon becomes the weapon of the selected set (the one there stays in the inventory), a bow takes
the arrows the player has on, and arrows go into the set of the bow. `player.equip('bow_b')` and then `player.equip('arrow_normal')`
leave the bow selected with its arrows. With `Inventory.set` you place them yourself, as below.

```js
// The bow in the first weapon set with its arrows, selected.
function readyBow(player, bowName, arrowName) {
  const { Inventory } = kcds.players;
  const current = Inventory.get(player);
  if (!current || !current.quickslots) {
    return false;
  }
  const bow = current.items.find((row) => row.name === bowName);
  const arrows = current.items.find((row) => row.name === arrowName);
  if (!bow || !arrows) {
    return false;
  }
  const quickslots = current.quickslots;
  quickslots.weapons[0] = { primary: bow.id, secondary: arrows.id };
  quickslots.activeWeapon = 0;
  return Inventory.set(player, { items: current.items, quickslots, revision: current.revision }).ok;
}

kcds.core.Events.on('playerCommand', (player, command) => {
  if (command === 'archer') {
    player.equip('bow_b');
    kcds.players.Inventory.add(player, { item: 'arrow_normal', amount: 40 });
    readyBow(player, 'bow_b', 'arrow_normal');
  }
});
```

```ts
// The bow in the first weapon set with its arrows, selected.
function readyBow(player: kcds.players.Player, bowName: string, arrowName: string) {
  const { Inventory } = kcds.players;
  const current = Inventory.get(player);
  if (!current || !current.quickslots) {
    return false;
  }
  const bow = current.items.find((row) => row.name === bowName);
  const arrows = current.items.find((row) => row.name === arrowName);
  if (!bow || !arrows) {
    return false;
  }
  const quickslots = current.quickslots;
  quickslots.weapons[0] = { primary: bow.id, secondary: arrows.id };
  quickslots.activeWeapon = 0;
  return Inventory.set(player, { items: current.items, quickslots, revision: current.revision }).ok;
}

kcds.core.Events.on('playerCommand', (player, command) => {
  if (command === 'archer') {
    player.equip('bow_b');
    kcds.players.Inventory.add(player, { item: 'arrow_normal', amount: 40 });
    readyBow(player, 'bow_b', 'arrow_normal');
  }
});
```

```lua
-- The bow in the first weapon set with its arrows, selected.
local function readyBow(player, bowName, arrowName)
  local Inventory = kcds.players.Inventory
  local current = Inventory.get(player)
  if not current or not current.quickslots then
    return false
  end
  local bow, arrows
  for _, row in ipairs(current.items) do
    if row.name == bowName then
      bow = row
    elseif row.name == arrowName then
      arrows = row
    end
  end
  if not bow or not arrows then
    return false
  end
  local weapons = {}
  for index, set in ipairs(current.quickslots.weapons) do
    weapons[index] = set
  end
  weapons[1] = { primary = bow.id, secondary = arrows.id }
  local quickslots = {
    weapons = weapons,
    items = current.quickslots.items,
    activeWeapon = 0,
    activeItem = current.quickslots.activeItem,
  }
  return Inventory.set(player, { items = current.items, quickslots = quickslots, revision = current.revision }).ok
end

kcds.core.Events.on("playerCommand", function(player, command)
  if command == "archer" then
    player:equip("bow_b")
    kcds.players.Inventory.add(player, { item = "arrow_normal", amount = 40 })
    readyBow(player, "bow_b", "arrow_normal")
  end
end)
```

```cs
// The bow in the first weapon set with its arrows, selected.
bool ReadyBow(Player player, string bowName, string arrowName)
{
    var current = Inventory.Get(player);
    if (current.IsNull || current["quickslots"].IsNull)
    {
        return false;
    }
    var rows = current["items"].ToList();
    var bow = rows.FindIndex((row) => row["name"].AsString() == bowName);
    var arrows = rows.FindIndex((row) => row["name"].AsString() == arrowName);
    if (bow < 0 || arrows < 0)
    {
        return false;
    }
    var bowSet = new { primary = rows[bow]["id"], secondary = rows[arrows]["id"] };
    var weapons = current["quickslots"]["weapons"]
        .Select((set, index) => index == 0 ? (object)bowSet : set)
        .ToList();
    var answer = Inventory.Set(player, new
    {
        items = current["items"],
        quickslots = new
        {
            weapons,
            items = current["quickslots"]["items"],
            activeWeapon = 0,
            activeItem = current["quickslots"]["activeItem"],
        },
        revision = current["revision"],
    });
    return answer["ok"].AsBool();
}

Events.On("playerCommand", (Player player, string command, string[] args) =>
{
    if (command == "archer")
    {
        player.Equip("bow_b");
        Inventory.Add(player, new { item = "arrow_normal", amount = 40 });
        ReadyBow(player, "bow_b", "arrow_normal");
    }
});
```

## Dressing or undressing one piece

```js
// Puts one row on or takes it off, leaving the other rows as they are.
function setWorn(player, rowId, worn) {
  const { Inventory } = kcds.players;
  const current = Inventory.get(player);
  if (!current) {
    return false;
  }
  const items = current.items.map((row) => {
    let equipped = row.equipped;
    if (row.id === rowId) {
      equipped = worn ? 1 : 0;
    }
    return { id: row.id, item: row.item, amount: row.amount, metadata: row.metadata, equipped };
  });
  const answer = Inventory.set(player, {
    items,
    quickslots: current.quickslots,
    revision: current.revision,
  });
  return answer.ok;
}
```

```ts
// Puts one row on or takes it off, leaving the other rows as they are.
function setWorn(player: kcds.players.Player, rowId: string, worn: boolean) {
  const { Inventory } = kcds.players;
  const current = Inventory.get(player);
  if (!current) {
    return false;
  }
  const items = current.items.map((row) => {
    let equipped = row.equipped;
    if (row.id === rowId) {
      equipped = worn ? 1 : 0;
    }
    return { id: row.id, item: row.item, amount: row.amount, metadata: row.metadata, equipped };
  });
  const answer = Inventory.set(player, {
    items,
    quickslots: current.quickslots,
    revision: current.revision,
  });
  return answer.ok;
}
```

```lua
-- Puts one row on or takes it off, leaving the other rows as they are.
local function setWorn(player, rowId, worn)
  local Inventory = kcds.players.Inventory
  local current = Inventory.get(player)
  if not current then
    return false
  end
  local items = {}
  for _, row in ipairs(current.items) do
    local equipped = row.equipped
    if row.id == rowId then
      equipped = worn and 1 or 0
    end
    table.insert(items, {
      id = row.id,
      item = row.item,
      amount = row.amount,
      metadata = row.metadata,
      equipped = equipped,
    })
  end
  local answer = Inventory.set(player, {
    items = items,
    quickslots = current.quickslots,
    revision = current.revision,
  })
  return answer.ok
end
```

```cs
// Puts one row on or takes it off, leaving the other rows as they are.
bool SetWorn(Player player, string rowId, bool worn)
{
    var current = Inventory.Get(player);
    if (current.IsNull)
    {
        return false;
    }
    var items = current["items"]
        .Select((row) => new
        {
            id = row["id"],
            item = row["item"],
            amount = row["amount"],
            metadata = row["metadata"],
            equipped = row["id"].AsString() == rowId ? (worn ? 1 : 0) : row["equipped"].AsNumber(),
        })
        .ToList();
    var answer = Inventory.Set(player, new
    {
        items,
        quickslots = current["quickslots"],
        revision = current["revision"],
    });
    return answer["ok"].AsBool();
}
```

- The game moves only rows whose `equipped` changed, and leaves whatever the player changed in the meantime.
- `revision` refuses the change with `staleRevision` if the inventory has changed, so read it again.
- `equipped` is only for equipment and weapons, and can be at most the row's `amount`. Otherwise you get `invalidEquipment`, as with clashing
  pieces, so `setWorn` on an occupied place returns `false`.

**`ok: true` does not yet mean the body wears it,** only that the server accepted the state. When the game has done what it can,
`playerInventoryChanged` fires with the reason `equipment`, and the rows show what is really worn:

```js
kcds.core.Events.on('playerInventoryChanged', (player, change) => {
  if (change.reason !== 'equipment') {
    return;
  }
  const rows = kcds.players.Inventory.get(player)?.items ?? [];
  const worn = rows.filter((row) => row.equipped > 0);
  console.log(`${player.nickname} wears ${worn.map((row) => row.name).join(', ')}`);
});
```

```ts
kcds.core.Events.on('playerInventoryChanged', (player, change) => {
  if (change.reason !== 'equipment') {
    return;
  }
  const rows = kcds.players.Inventory.get(player)?.items ?? [];
  const worn = rows.filter((row) => row.equipped > 0);
  console.log(`${player.nickname} wears ${worn.map((row) => row.name).join(', ')}`);
});
```

```lua
kcds.core.Events.on("playerInventoryChanged", function(player, change)
  if change.reason ~= "equipment" then
    return
  end
  local current = kcds.players.Inventory.get(player)
  local worn = {}
  for _, row in ipairs(current and current.items or {}) do
    if row.equipped > 0 then
      table.insert(worn, row.name)
    end
  end
  print(player.nickname .. " wears " .. table.concat(worn, ", "))
end)
```

```cs
Events.On("playerInventoryChanged", (Player player, JsValue change) =>
{
    if (change["reason"].AsString() != "equipment")
    {
        return;
    }
    var rows = Inventory.Get(player)["items"];
    var worn = rows.Where((row) => row["equipped"].AsNumber() > 0).Select((row) => row["name"]);
    Log.Info($"{player.Nickname} wears {string.Join(", ", worn)}");
});
```

## Reading what someone wears

- `Inventory.get(player).items` with `equipped > 0` gives the rows worn or held, with quality and condition.
- `player.equipment` gives the classes the body wears, as 32 hex digits.
- `player.rightHandItem` and `leftHandItem` give the class in each hand, or `''`.

Client scripts see the same for each player their game shows.

## Restoring what a player wore

Save the inventory as `Inventory.get` gives it on disconnect, and return it with `Inventory.set` when they come back. Their game dresses them by
`equipped` and restores the quick slots. A player's joining name comes with their client, so wait for `playerReady`.

```js
const { Inventory } = kcds.players;
const { Events } = kcds.core;
const savedOutfits = new Map();

Events.on('playerDisconnect', (player) => {
  const current = Inventory.get(player);
  if (!current) {
    return;
  }
  const items = current.items.map((row) => ({
    id: row.id,
    item: row.item,
    amount: row.amount,
    metadata: row.metadata,
    equipped: row.equipped,
  }));
  savedOutfits.set(player.nickname, { items, quickslots: current.quickslots });
});

Events.on('playerReady', (player) => {
  const outfit = savedOutfits.get(player.nickname);
  if (outfit) {
    Inventory.set(player, outfit);
  }
});
```

```ts
const { Inventory } = kcds.players;
const { Events } = kcds.core;
const savedOutfits = new Map();

Events.on('playerDisconnect', (player) => {
  const current = Inventory.get(player);
  if (!current) {
    return;
  }
  const items = current.items.map((row) => ({
    id: row.id,
    item: row.item,
    amount: row.amount,
    metadata: row.metadata,
    equipped: row.equipped,
  }));
  savedOutfits.set(player.nickname, { items, quickslots: current.quickslots });
});

Events.on('playerReady', (player) => {
  const outfit = savedOutfits.get(player.nickname);
  if (outfit) {
    Inventory.set(player, outfit);
  }
});
```

```lua
local Inventory = kcds.players.Inventory
local Events = kcds.core.Events
local savedOutfits = {}

Events.on("playerDisconnect", function(player)
  local current = Inventory.get(player)
  if not current then
    return
  end
  local items = {}
  for _, row in ipairs(current.items) do
    table.insert(items, {
      id = row.id,
      item = row.item,
      amount = row.amount,
      metadata = row.metadata,
      equipped = row.equipped,
    })
  end
  savedOutfits[player.nickname] = { items = items, quickslots = current.quickslots }
end)

Events.on("playerReady", function(player)
  local outfit = savedOutfits[player.nickname]
  if outfit then
    Inventory.set(player, outfit)
  end
end)
```

```cs
var savedOutfits = new Dictionary<string, object>();

Events.On("playerDisconnect", (Player player) =>
{
    var current = Inventory.Get(player);
    if (current.IsNull)
    {
        return;
    }
    var items = current["items"]
        .Select((row) => new
        {
            id = row["id"],
            item = row["item"],
            amount = row["amount"],
            metadata = row["metadata"],
            equipped = row["equipped"],
        })
        .ToList();
    savedOutfits[player.Nickname] = new { items, quickslots = current["quickslots"] };
});

Events.On("playerReady", (Player player) =>
{
    if (savedOutfits.TryGetValue(player.Nickname, out var outfit))
    {
        Inventory.Set(player, outfit);
    }
});
```

This example only keeps it in memory. A real server should save it in a database.

Reference: [Inventory](/docs/api/inventory).
