# Actions and their phases

Many things a player does change the world: taking an item, opening a door, striking someone. Each one goes through
up to three phases, and a resource can step in at each of them.

- **Before:** an event or a handler that runs before anything changes. It can refuse the action, and some can
  change it, such as the damage of a blow.
- **Commit:** the server makes the change and checks it against the state as it is then. An action that no longer
  fits is refused by the usual rules.
- **After:** an event that says what happened. It cannot undo anything.

Most "before" events decide at once: return `false` to refuse. `Inventory.beforeCommit` can also wait for an answer
from a web service or a database (see [Inventory and items](inventory.md)).

## Which action has which phase

| Action | Before | After |
| --- | --- | --- |
| Taking, dropping or using an item, a shop or player basket, a container, looting a body, tack on a horse | `Inventory.beforeCommit` (may wait, at most 5 s) | `playerInventoryChanged`, `playerItemUsed`, `vendorTrade`, `tradeDone`, `playerLooted`, `groundItemPickup` |
| Eating or drinking | `playerConsuming`, then `playerConsumptionEffects` | `playerConsumed` |
| What the player's own game changed (wear, ammunition, pickups) | | `playerInventoryChanged` with `operation` |
| Opening a player trade, confirming a basket | `tradeOpen`, `tradeItem` | `tradeDone`, `tradeClosed` |
| A price in a haggle | `vendorHaggle` | `vendorTrade` |
| Opening, closing, unlocking or lockpicking a door | `doorInteract` | |
| Opening a stash | `stashInteract` | |
| Picking up an item the level placed | `worldItemPickup` | |
| Taking from or giving to another player's inventory view | `playerInventoryTake`, `playerInventoryGive` | `playerInventoryChanged` |
| Taking from a body | `playerLoot`, `npcLoot` | `playerLooted` |
| Another player's blow or shot on a player | `playerHit` (refuse, or change the damage) | `playerDamage` |
| Any hit on an NPC | `npcHit` (refuse, or change the damage) | `npcDamage`, `npcDeath` |
| A knock-out | | `playerDied` with the episode |
| A revive | `revive({ episode })` ends only that knock-out | `playerRevived` |
| A fight a resource starts | | `playerFightStarted`, `playerFightEnded` |
| An animation a resource plays | | `playerAnimationEnd`, `npcAnimationEnd` (once per request) |
| An NPC's walk, follow or patrol leg | | `npcIntentDone` |

## Ids that make retries safe

- **Inventory batches:** each batch of changes from a player's game has an `operation`, such as `'3:17'`. The server
  applies a batch once, even when the game sends it again.
- **Inventory transactions:** every transaction `Inventory.beforeCommit` sees has its own `id`.
- **Knock-outs:** each knock-out has an episode number. `revive({ episode })` refuses to end a later knock-out, so a
  late timer or a second revive does no harm.
- **Animations:** `playerAnimationEnd` and `npcAnimationEnd` fire once per request.

## An example

A door that only guards may open, and a log line for every item a player's game spent:

```js
const { Events } = kcds.core;

Events.on('doorInteract', (player) => {
  return player.state.get('guard') === true;
});

Events.on('playerItemUsed', (player, item, kind, amount) => {
  console.info(`${player.nickname} used ${amount} ${item} (${kind})`);
});
```

```ts
const { Events } = kcds.core;

Events.on('doorInteract', (player) => {
  return player.state.get('guard') === true;
});

Events.on('playerItemUsed', (player, item, kind, amount) => {
  console.info(`${player.nickname} used ${amount} ${item} (${kind})`);
});
```

```lua
local Events = kcds.core.Events

Events.on("doorInteract", function(player)
  return player.state:get("guard") == true
end)

Events.on("playerItemUsed", function(player, item, kind, amount)
  print(player.nickname .. " used " .. tostring(amount) .. " " .. tostring(item) .. " (" .. tostring(kind) .. ")")
end)
```

```cs
Events.On("doorInteract", (Player player) =>
{
    return player.State.Get("guard").AsBool();
});

Events.On("playerItemUsed", (Player player, JsValue item, JsValue kind, JsValue amount) =>
{
    Log.Info($"{player.Nickname} used {amount} {item} ({kind})");
});
```

Reference: [Events](events.md), [Inventory and items](inventory.md), [Players](players.md), [NPCs](npcs.md).
