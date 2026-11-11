# Events

A resource reacts to the game through events: players joining, chatting and fighting, NPCs dying,
doors opening, as well as events from your own and other resources.

```js
const { Events, Chat } = kcds.core;

Events.on('playerConnect', (player) => {
  console.info(`${player.nickname} is connecting`);
});

Events.on('playerReady', (player) => {
  Chat.sendToPlayer(player, 'Welcome!');
});

Events.on('playerDisconnect', (player) => {
  console.info(`${player.nickname} left`);
});
```

```ts
const { Events, Chat } = kcds.core;

Events.on('playerConnect', (player) => {
  console.info(`${player.nickname} is connecting`);
});

Events.on('playerReady', (player) => {
  Chat.sendToPlayer(player, 'Welcome!');
});

Events.on('playerDisconnect', (player) => {
  console.info(`${player.nickname} left`);
});
```

```lua
local Events, Chat = kcds.core.Events, kcds.core.Chat

Events.on("playerConnect", function(player)
  print(player.nickname .. " is connecting")
end)

Events.on("playerReady", function(player)
  Chat.sendToPlayer(player, "Welcome!")
end)

Events.on("playerDisconnect", function(player)
  print(player.nickname .. " left")
end)
```

```cs
Events.On("playerConnect", (Player player) =>
{
    Log.Info($"{player.Nickname} is connecting");
});

Events.On("playerReady", (Player player) =>
{
    Chat.SendToPlayer(player, "Welcome!");
});

Events.On("playerDisconnect", (Player player) =>
{
    Log.Info($"{player.Nickname} left");
});
```

`Events.on`, `Events.once` and `Events.off` work the same for the game's events and your own, and
`Events.listenerCount(name)` says how many handlers this side has for one. On the server,
`Events.fromClient(name, fn)` hears what players' games send, `Events.onceFromClient` the next one only, and
`Events.offFromClient` removes such a handler. To raise your own events:

| Call                                   | Who hears it                                         |
| -------------------------------------- | ---------------------------------------------------- |
| `Events.emit(name, ...args)`           | All resources on this side. Returns a promise.       |
| `Events.toResource(res, name, ...args)` | One resource, by name.                               |
| `Events.fire(name, ...args)`           | This resource only, through `Events.listen`.         |
| `Events.toServer(name, payload)`       | The server's `Events.fromClient`, sent from a client. |
| `Events.broadcast(name, payload)`      | All players' client scripts, sent from the server.   |
| `player.emit(name, payload)`           | One player's client scripts, sent from the server.   |

[Client resources](client-resources.md) has more on talking between the server and clients.

## A player's arrival

These fire in this order, and `playerDisconnect` fires when they leave.

| Event              | When                                                                                  |
| ------------------ | ------------------------------------------------------------------------------------- |
| `playerConnecting` | Someone asks to join. `request.reject(reason)` turns them away.                       |
| `playerConnect`    | They joined. Their body and inventory exist, and they are still loading.              |
| `playerSpawning`   | Their game waits for a place to appear. Choose it with `player.spawn(position, rotation)`. |
| `playerSpawned`    | They stand in the loaded world.                                                       |
| `playerReady`      | Their game took down its loading screen, so they can see and play.                    |

## Refusing an action

Some events ask before something happens. A handler that returns `false`, not a promise of it,
refuses the action, and it is undone for everyone. All handlers still run, in the order added.

These events ask: `cartEntering`, `craftingCompleting`, `craftingStarting`, `doorInteract`,
`gatheringHarvest`, `horseGearChanging`, `horseMounting`, `npcHarvest`, `npcLoot`, `playerCarrying`,
`playerCarryingItem`, `playerConsuming`, `playerConsumptionEffects`, `playerHarm`, `playerHit`, `playerLoot`,
`playerPerkLearning`, `playerPickpocketStart`, `playerPoisonAbsorbing`, `playerPreparing`,
`playerTravelling`, `playerTreeChop`, `playerTreeFell`, `playerTrunkChop`, `playerTrunkCut`, `playerXpGaining`,
`propPlaceConfirm`, `siegeEngineUse`, `siegeLadderUse`, `stashInteract`, `tradeItem`, `tradeOpen`,
`treeRegrow` and `worldItemPickup`.

```js
const { Events } = kcds.core;
const { Area } = kcds.world;

// No picking locks in town.
Events.on('doorInteract', (player, door, action) => {
  if (action !== 'lockpick') {
    return;
  }
  const inTown = Area.aroundEntity(player).some((area) => area.labels.includes('town'));
  if (inTown) {
    return false;
  }
});
```

```ts
const { Events } = kcds.core;
const { Area } = kcds.world;

// No picking locks in town.
Events.on('doorInteract', (player, door, action) => {
  if (action !== 'lockpick') {
    return;
  }
  const inTown = Area.aroundEntity(player).some((area) => area.labels.includes('town'));
  if (inTown) {
    return false;
  }
});
```

```lua
local Events = kcds.core.Events
local Area = kcds.world.Area

-- No picking locks in town.
Events.on("doorInteract", function(player, door, action)
  if action ~= "lockpick" then
    return
  end
  for _, area in ipairs(Area.aroundEntity(player)) do
    for _, label in ipairs(area.labels) do
      if label == "town" then
        return false
      end
    end
  end
end)
```

```cs
// No picking locks in town.
Events.On("doorInteract", (Player player, JsValue door, string action) =>
{
    if (action != "lockpick")
    {
        return (bool?)null;
    }
    var inTown = Area.AroundEntity(player).Any((area) => area.Labels.Contains("town"));
    return inTown ? false : null;
});
```

## Chat and commands

`playerChat(player, text)` is an ordinary message. `playerCommand(player, command, args)` is a message
starting with `/`, and goes to no other player. By default, a chat message reaches everyone before
`playerChat` fires. To handle chat yourself, call `Chat.setChatRelay(false)`.

## All events

Each event with its arguments. The [Events](/docs/api/events) reference has its full description and
an example. Server scripts hear these unless marked (client), or (both) for both sides.

### Players: joining and leaving

| Event                  | Arguments        | When                                                     |
| ---------------------- | ---------------- | -------------------------------------------------------- |
| `playerConnecting`     | `request`        | Someone asks to join, before their connection exists.    |
| `playerConnect`        | `player`         | A player joined and is still loading.                    |
| `playerSpawning`       | `player`         | Their game waits for a place to appear.                  |
| `playerSpawned`        | `player`         | They stand in the loaded world.                          |
| `playerReady` (both)   | `player`         | Their loading screen is down.                            |
| `playerResourcesReady` | `player`         | Their game started the client resources.                 |
| `playerDisconnect`     | `player`         | A player left.                                           |
| `playerQueued`         | `view`           | A player joined a full server's queue.                   |
| `playerUnqueued`       | `view, value`    | A player left the queue.                                 |
| `loadingProgress` (client) | `stage, percent` | How far the game has loaded ([Web views](web-views.md)). |
| `playerTeleported`     | `player, position` | A player a script moved now stands there (their game reported the body within 3 m). |
| `playerStanceChanged`  | `player, stance, previous` | Their game chose another stance for the body (`player.bodyStance`). |
| `playerLobbyChanged`   | `player, held`   | `player.setLobby` held them out of the world or let them in ([Waiting in a lobby](players-connecting.md#waiting-in-a-lobby)). |
| `playerTravelling`     | `player, trip`   | They began the game's travel to another map; return `false` to keep it an ordinary trip ([Several maps](world.md#several-maps-on-one-server)). |
| `playerMapChanged`     | `player, map, from` | `player.setMap` or a travel brought them to another map, before `playerSpawned`. |

### Players: chat and commands

| Event            | Arguments                | When                                                 |
| ---------------- | ------------------------ | ---------------------------------------------------- |
| `consoleCommand` | `command, parts`         | A server console command the server does not handle. |
| `playerChat`     | `player, text`           | A player sent a chat message.                        |
| `playerCommand`  | `player, command, parts` | A player typed a message starting with /.            |

### Players: fighting, damage and knock-outs

| Event                   | Arguments                                    | When                                                |
| ----------------------- | -------------------------------------------- | --------------------------------------------------- |
| `playerAttack`          | `player, opponent`                           | The server accepted a melee swing at another player. |
| `playerCombatCancelled` | `player, opponent`                           | `player.stopFight` ended melee combat.              |
| `playerDamage`          | `player, attacker, amount, bodyPart, reason` | A player's game took health off them.               |
| `playerDied`            | `player, killer, reason`                     | A player was knocked out.                           |
| `playerHarm`            | `player, harm`                               | An NPC's blow, a fall, fire or a collision is about to take health. |
| `playerHit`             | `player, attacker, hit`                      | Another player's blow or shot is about to take health. |
| `playerMissileHit`      | `player, hit`                                | A player's arrow or bolt came to rest, with what it struck. |
| `playerInjured`         | `player, limb`                               | A limb was injured.                                 |
| `playerInjuryHealed`    | `player, limb`                               | A limb's injury healed.                             |
| `playerWeaponHit`       | `player, hit`                                | A swing struck the world and not a character.       |
| `playerWhistle`         | `player`                                     | A player whistled for their horse.                  |

### Players: carrying, cuffs and the pillory

| Event                   | Arguments             | When                                                  |
| ----------------------- | --------------------- | ----------------------------------------------------- |
| `playerCarry`           | `carrier, carried`    | A player lifted a knocked-out player.                 |
| `playerCarrying`        | `carrier, carried`    | A player is about to lift a knocked-out player.       |
| `playerCarryingItem`    | `player, item, stack` | A player is about to pick up a carryable ground item. |
| `playerCarryItem`       | `player, item, stack` | A player now carries an item in their arms.           |
| `playerCuffedChange`    | `player, cuffed`      | `player.setCuffed` cuffed or uncuffed a player.       |
| `playerPilloryReleased` | `player, reason`      | A player left the pillory.                            |
| `playerPutDown`         | `carrier, carried`    | A carried player was put down.                        |
| `playerPutDownItem`     | `player, item, stack` | A player stopped carrying an item and set it down.    |

### Players: inventory, food and items

| Event                      | Arguments                     | When                                                  |
| -------------------------- | ----------------------------- | ----------------------------------------------------- |
| `playerConsumed`           | `player, consumed`            | A player ate or drank something.                      |
| `playerConsuming`          | `player, consumption`         | A player is about to eat or drink.                    |
| `playerConsumptionEffects` | `player, consumption`         | Follows `playerConsuming`, and can stop the game's effects. |
| `playerCustomItemUse`      | `player, item, revision`      | A player used a custom usable item.                   |
| `playerInventoryChanged`   | `player, change`              | Rows in their inventory changed.                      |
| `playerInventoryGive`      | `player, owner, line`         | A player is about to put an item into another player's inventory through a view of it. Return false to refuse. |
| `playerInventoryGiven`     | `player, owner, items`        | A player put items into another player's inventory.   |
| `playerInventoryTransaction` | `player, outcome`           | An action under `Inventory.beforeCommit` settled: applied or refused, with its id. |
| `playerInventoryTake`      | `player, owner, line`         | A player is about to take an item from another player's inventory through a view of it. Return false to refuse. |
| `playerInventoryTaken`     | `player, owner, items`        | A player took items from another player's inventory.  |
| `playerInventoryReady`     | `player`                      | Their game shows the inventory the server holds.      |
| `playerItemUsed`           | `player, item, kind, amount`  | A player used up units of an item.                    |
| `playerLoot`               | `player, victim, line`        | A player is about to take from a knocked-out body.    |
| `playerLooted`             | `player, victim, items`       | A player took items from a knocked-out body.          |
| `playerPoisonAbsorbing`    | `player, consumption, poison` | A player is about to get a poison they consumed.      |
| `playerPreparing`          | `player, preparing`           | A player is about to cook, smoke or dry food.         |
| `playerPrepared`           | `player, preparation`         | A preparation is over, with `items` the game converted. |

### Players: progression and buffs

| Event                       | Arguments                          | When                                             |
| --------------------------- | ---------------------------------- | ------------------------------------------------ |
| `playerBuffAdded`           | `player, buff, source`             | A status effect landed on a player.              |
| `playerBuffBlocked`         | `player, buff`                     | The server's rules kept a status effect off.     |
| `playerBuffRemoved`         | `player, buff, reason`             | A status effect left a player.                   |
| `playerCustomBuffApply`     | `player, buff`                     | An instance of a registered buff landed.         |
| `playerCustomBuffRemove`    | `player, buff, reason`             | An instance of a registered buff left.           |
| `playerCustomBuffTick`      | `player, buff`                     | A registered buff's tick.                        |
| `playerLevelUp`             | `player, track, level, perkPoints` | A player reached a new level on a track.         |
| `playerPerkAdded`           | `player, perk, source`             | A player got a perk.                             |
| `playerPerkLearning`        | `player, perk`                     | A player chooses a perk.                         |
| `playerPerkRemoved`         | `player, perk, source`             | A player lost a perk.                            |
| `playerProgressionRejected` | `player, reason`                   | Their game reported XP or perks the server did not grant. |
| `playerStatsChanged`        | `player, changes`                  | Stats in their report changed.                   |
| `playerXpGained`            | `player, track, xp`                | A player gained XP on a track.                   |
| `playerXpGaining`           | `player, track, xp, source`        | Their game is about to grant XP.                 |

### Players: pickpocketing and animations

| Event                    | Arguments                  | When                                               |
| ------------------------ | -------------------------- | -------------------------------------------------- |
| `playerAnimationEnd`     | `player, fragment, reason` | An animation from `player.playAnimation` ended.    |
| `playerPickpocketCaught` | `thief, victim`            | A victim caught their pickpocket.                  |
| `playerPickpocketed`     | `thief, victim, items`     | A pickpocketing ended, with the items taken.       |
| `playerPickpocketStart`  | `thief, victim`            | A player tries to pickpocket another.              |
| `propPickedUp`           | `player, prop`             | An animation took a prop into the hand.            |
| `propPlaced`             | `player, prop`             | An animation put a prop down.                      |

### NPCs and patrols

| Event                 | Arguments                                 | When                                             |
| --------------------- | ----------------------------------------- | ------------------------------------------------ |
| `npcAnimationEnd`     | `npc, fragment, reason`                   | An animation from `npc.playAnimation` ended.     |
| `npcDamage`           | `npc, attacker, amount, bodyPart, reason` | An NPC lost health.                              |
| `npcDeath`            | `npc, killer`                             | An NPC died.                                     |
| `npcDestroy`          | `npc`                                     | An NPC is being removed.                         |
| `npcHarvest`          | `player, npc`                             | A player is about to get a carcass's harvest.    |
| `npcHarvested`        | `npc, player`                             | A player got a carcass's harvest.                |
| `npcIntentDone`       | `npc, status`                             | An NPC finished a moveTo, a follow or a patrol leg. |
| `npcInteract`         | `npc, player`                             | A player pressed use on an interactable NPC.     |
| `npcInventoryChanged` | `npc, change`                             | A spawned NPC's stock changed.                   |
| `npcLoot`             | `player, npc, line`                       | A player is about to take from a dead NPC.       |
| `npcLooted`           | `player, npc, items`                      | A player took items from a dead NPC.             |
| `npcRevive`           | `npc`                                     | A dead NPC was revived.                          |
| `npcSimulatorChange`  | `npc, simulator, previous`                | Another player's game now simulates the NPC.     |
| `npcSpawn`            | `npc`                                     | An NPC was spawned.                              |
| `patrolFinished`      | `npc, patrol`                             | A patrol ended: done, cancelled or given up.     |
| `patrolWaypoint`      | `npc, patrol`                             | An NPC reached a waypoint, or a leg failed.      |

### Horses and dogs

| Event               | Arguments                         | When                                        |
| ------------------- | --------------------------------- | ------------------------------------------- |
| `dogDestroy`        | `dog`                             | A dog is being removed.                     |
| `dogModeChanged`    | `dog, mode`                       | A dog's mode changed.                       |
| `dogOwnerChanged`   | `dog, owner`                      | A dog got a new owner, or none.             |
| `dogSpawn`          | `dog`                             | A dog was spawned.                          |
| `horseDamage`       | `horse, attacker, amount, reason` | A horse lost health.                        |
| `horseDeath`        | `horse, killer, reason`           | A horse died.                               |
| `horseDestroy`      | `horse`                           | A horse is being removed.                   |
| `horseDismount`     | `horse, player`                   | A rider left the saddle.                    |
| `horseGearChanged`  | `horse, player, gear`             | A horse's tack changed on all clients.      |
| `horseGearChanging` | `horse, player, gear`             | An owner changed the tack, before the server takes it. |
| `horseIntentDone`   | `horse, status`                   | A horse finished a moveTo or patrol order.  |
| `horseKnockedOut`   | `horse, attacker, reason`         | A horse was knocked out.                    |
| `horseMount`        | `horse, player`                   | A player sat in the saddle.                 |
| `horseMounting`     | `horse, player`                   | A player mounts, before the server accepts it. |
| `horseOwnerChanged` | `horse, owner`                    | A horse got a new owner, or none.           |
| `horseRevive`       | `horse`                           | A dead horse was revived.                   |
| `horseSpawn`        | `horse`                           | A horse was spawned.                        |
| `horseWake`         | `horse`                           | A knocked-out horse got up.                 |

### Doors, gates and containers

| Event                   | Arguments                       | When                                                 |
| ----------------------- | ------------------------------- | ---------------------------------------------------- |
| `doorDiscovered`        | `door, player`                  | The server met a level door for the first time in this run. |
| `doorInteract`          | `player, door, action, keySide` | A player is about to open, close, unlock or lockpick a door. |
| `gateBarChanged`        | `gate, barred`                  | A gate's bar was put in or taken out.                |
| `gateChanged`           | `gate, state`                   | A gate moved to another state.                       |
| `graveDug`              | `player, grave`                 | A player dug out a grave.                            |
| `stashClose`            | `stash, player, reason`         | A player closed a stash.                             |
| `stashDestroy`          | `stash`                         | A stash is being removed.                            |
| `stashInteract`         | `player, stash, action`         | A player is about to open a stash.                   |
| `stashInventoryChanged` | `stash, player, change`         | A script changed a stash's contents.                 |
| `stashOpen`             | `stash, player`                 | A player opened a stash.                             |
| `stashSpawn`            | `stash`                         | A chest, container or searchable body was created.   |
| `stashUnlocked`         | `stash, player, method`         | A player unlocked a locked stash.                    |

### Carts

| Event          | Arguments            | When                                                 |
| -------------- | -------------------- | ---------------------------------------------------- |
| `cartDestroy`  | `cart`               | A cart is being removed, with everyone let off first. |
| `cartEnter`    | `cart, player, seat` | A player took a seat.                                |
| `cartEntering` | `cart, player, seat` | A player is about to take a seat.                    |
| `cartExit`     | `cart, player, seat` | A player got off.                                    |
| `cartSpawn`    | `cart`               | A cart or wagon was spawned.                         |

### Props, effects and the ground

| Event               | Arguments                           | When                                          |
| ------------------- | ----------------------------------- | --------------------------------------------- |
| `groundItemDestroy` | `stack`                             | A ground item is being removed.               |
| `groundItemPickup`  | `stack, player`                     | A player picked up a ground item.             |
| `groundItemResting` | `stack`                             | A ground item came to rest.                   |
| `groundItemSpawn`   | `stack`                             | A stack of items appeared on the ground.      |
| `propDestroy`       | `prop`                              | A prop is being removed.                      |
| `propHit`           | `prop, speed, position`             | A rigid prop struck something, at `speed` m/s. |
| `propPlaceCancel`   | `player, session`                   | A player cancelled a placement.               |
| `propPlaceConfirm`  | `player, session, placement`        | A player confirmed a placement.               |
| `propPlaceEnd`      | `player, session`                   | A placement session ended.                    |
| `propPlaceMove`     | `player, session, placement, valid` | A player moved a `PropPlacer` preview.        |
| `propPlaceRejected` | `player, session, placement`        | A placement was refused.                      |
| `propRest`          | `prop, position, rotation`          | A rigid prop came to rest.                    |
| `propSpawn`         | `prop`                              | A prop was spawned.                           |
| `vfxDestroy`        | `vfx`                               | A visual effect is being removed.             |
| `vfxSpawn`          | `vfx`                               | A visual effect was spawned.                  |
| `worldItemPickup`   | `player, item`                      | A player is about to pick up an item the level placed. |

### Markers and areas

| Event          | Arguments                         | When                                  |
| -------------- | --------------------------------- | ------------------------------------- |
| `areaEnter`    | `area, entity, matchingDimension` | A player, horse or NPC entered an area. |
| `areaExit`     | `area, entity, matchingDimension` | A player, horse or NPC left an area.  |
| `markerEnter`  | `marker, player`                  | A player walked into a trigger marker. |
| `markerExit`   | `marker, player`                  | A player left a trigger marker.       |
| `markerPlace`  | `marker`                          | A marker was placed.                  |
| `markerRemove` | `marker`                          | A marker is being removed.            |

### Crafting and gathering

| Event                   | Arguments                 | When                                                |
| ----------------------- | ------------------------- | --------------------------------------------------- |
| `craftingCompleted`     | `player, details`         | A craft gave its product.                           |
| `craftingCompleting`    | `player, proposal`        | A craft is about to give its product.               |
| `craftingEnded`         | `player, details`         | A session ended: 'success', 'failed' or 'cancelled'. |
| `craftingRecipeLearned` | `player, details`         | A player learned a custom recipe.                   |
| `craftingRefunding`     | `player, details, refund` | An unfinished craft is settling.                    |
| `craftingStarted`       | `player, details`         | A player started crafting.                          |
| `craftingStarting`      | `player, proposal`        | A player is about to start crafting.                |
| `craftingStationUsed`   | `player, details`         | A player used a station with more than one recipe.  |
| `gatheringHarvest`      | `player, proposal`        | A player picked a plant and is about to get it.     |
| `gatheringHarvested`    | `player, details`         | A player got what they picked.                      |

### Trees

| Event                 | Arguments             | When                                                |
| --------------------- | --------------------- | --------------------------------------------------- |
| `playerTreeChop`      | `player, tree, blow`  | An axe blow on a standing tree counts.              |
| `playerTreeFell`      | `player, tree, fall`  | A last blow fells a tree.                           |
| `playerTrunkChop`     | `player, tree, blow`  | An axe blow on a felled trunk counts.               |
| `playerTrunkCut`      | `player, tree, items` | A last blow cuts a trunk into items.                |
| `treeChop` (client)   | `tree, blow`          | Any player's counted blow on a standing tree here.  |
| `treeFalling` (client) | `tree, fall`         | A tree here starts to fall.                         |
| `treeFelled` (both)   | `tree`                | A felled trunk came to rest.                        |
| `treeRegrow`          | `tree`                | A tree is about to grow back.                       |
| `treeRegrown` (both)  | `tree`                | A felled tree stands again.                         |
| `trunkCut` (client)   | `tree, items`         | A felled trunk here was cut into items.             |

### Dice

| Event            | Arguments                                                       | When                       |
| ---------------- | --------------------------------------------------------------- | -------------------------- |
| `diceBadge`      | `event`                                                         | A badge took effect on a throw. |
| `diceMatchEnd`   | `match, first, second, winner, reason, firstScore, secondScore` | A dice match ended.        |
| `diceMatchStart` | `match, first, second, target`                                  | A dice match began.        |
| `diceMatchStarting` | `first, second, table, options`                              | A dice match is about to begin; change `options` to set it. |
| `diceMove`       | `event`                                                         | A player held dice and continued or passed. |
| `diceRoll`       | `event`                                                         | Dice were thrown.          |
| `diceSettled`    | `event`                                                         | The thrown dice lie still. |
| `diceTableUse`   | `player, table, match`                                          | A player used a dice table. |
| `diceTurnEnd`    | `event`                                                         | A turn ended.              |
| `diceTurnStart`  | `event`                                                         | A player's turn began.     |

### Trade, shops and conversations

| Event                 | Arguments                                        | When                                       |
| --------------------- | ------------------------------------------------ | ------------------------------------------ |
| `dialogueChoice`      | `session, player, option`                        | A player chose an option from `Dialogue.open`. |
| `dialogueClosed`      | `session, player, reason`                        | A conversation ended.                      |
| `tradeClosed`         | `seller, player, reason`                         | A trade ended.                             |
| `tradeDone`           | `seller, buyer, boughtLines, soldLines, balance` | A basket went through.                     |
| `tradeItem`           | `seller, buyer, boughtLines, soldLines, balance` | The buyer confirmed a basket.              |
| `tradeOpen`           | `seller, buyer`                                  | A trade is about to open.                  |
| `vendorClosed` (both) | `vendor, player, reason`                         | A shop screen closed.                      |
| `vendorHaggle`        | `vendorId, player, basket, offered`              | A player offered a price in a vendor's haggle. |
| `vendorOpened` (client) | `session, npc`                                 | A server vendor's shop screen opened.      |
| `vendorTrade`         | `vendorId, player, bought, sold, balance`        | A player traded with a vendor.             |

### Board and table games

| Event                 | Arguments                      | When                                                        |
| --------------------- | ------------------------------ | ----------------------------------------------------------- |
| `tabletopSit`         | `table, player, seat`          | A player is about to sit at a table. Return false to refuse. |
| `tabletopSeated`      | `table, player, seat`          | A player sat down: start a game once every seat is taken.   |
| `tabletopLeave`       | `table, player, seat, reason`  | A player left a seat: `stood`, `disconnect` or `removed`.   |
| `tabletopStart`       | `table`                        | A game started (`table.start`).                             |
| `tabletopTurn`        | `table, seat, player`          | The turn went to a seat.                                    |
| `tabletopTurnTimeout` | `table, seat, player`          | A turn's time ran out. Return false to keep the turn a while longer. |
| `tabletopPieceSelect` | `table, player, piece`         | A seated player picked a piece. Return false to refuse.     |
| `tabletopMoveRequest` | `table, player, piece, cell`   | A seated player asks to move their piece to a cell. Return false to refuse. |
| `tabletopCellClick`   | `table, player, cell`          | A seated player clicked a cell.                             |
| `tabletopForfeit`     | `table, player, seat, reason`  | A player left a game in play and lost it.                   |
| `tabletopEnd`         | `table, winner, reason`        | A game ended; the winner has taken the pot.                 |

### Voice chat

| Event                             | Arguments         | When                                               |
| --------------------------------- | ----------------- | -------------------------------------------------- |
| `voiceTalkingChanged`             | `player, talking` | A player started or stopped talking.               |
| `voiceTierRequest`                | `player, tier`    | A player asked to talk in another tier. Return false to refuse. |
| `localVoiceTransmitting` (client) | `transmitting`    | This player's voice started or stopped going out.  |

### Quests

| Event                         | Arguments                | When                                     |
| ----------------------------- | ------------------------ | ---------------------------------------- |
| `questTrackingChanged` (both) | `quest, player, tracked` | A player started or stopped tracking a quest. |

### Siege

| Event                | Arguments                    | When                                       |
| -------------------- | ---------------------------- | ------------------------------------------ |
| `siegeEngineDamage`  | `engine, amount, attacker`   | A siege engine took damage.                |
| `siegeEngineDestroy` | `engine`                     | A siege engine is being removed.           |
| `siegeEngineReady`   | `engine`                     | A siege engine finished loading.           |
| `siegeEngineSpawn`   | `engine`                     | A siege engine was spawned.                |
| `siegeEngineUse`     | `player, engine, action`     | A player is about to operate an engine.    |
| `siegeEngineWreck`   | `engine, attacker`           | A siege engine was wrecked.                |
| `siegeFire`          | `engine, attacker, target`   | A siege engine fired.                      |
| `siegeImpact`        | `engine, position, attacker` | A siege engine's shot landed.              |
| `siegeLadderDestroy` | `ladder`                     | A siege ladder is being removed.           |
| `siegeLadderFall`    | `ladder, pusher`             | A ladder was pushed away and fell.         |
| `siegeLadderRaise`   | `ladder`                     | A ladder was raised against a wall.        |
| `siegeLadderSpawn`   | `ladder`                     | A siege ladder was spawned.                |
| `siegeLadderUse`     | `player, ladder, action`     | A player is about to push away or raise a ladder. |
| `stoneImpact`        | `pile, position, player`     | A thrown stone landed, hurting anyone near. |
| `stonePileDestroy`   | `pile`                       | A stone pile is being removed.             |
| `stonePileSpawn`     | `pile`                       | A stone pile was spawned.                  |
| `stonePileThrow`     | `player, pile`               | A player threw a stone from a pile.        |

### World

| Event                 | Arguments                   | When                                  |
| --------------------- | --------------------------- | ------------------------------------- |
| `worldDayChange`      | `day`                       | The world's clock reached a new day.  |
| `worldResourcesReady` |                             | All world resources have loaded.      |
| `worldWeatherChange`  | `preset, previous, seconds` | `World.setWeather` started changing the weather. |

### Resources and state

| Event                  | Arguments                      | When                                         |
| ---------------------- | ------------------------------ | -------------------------------------------- |
| `entityStateChange`    | `entity, key, value, previous` | A value in an entity's shared state changed. |
| `resourceStart` (both) | `name`                         | A resource's scripts have run.               |
| `resourceStop` (both)  | `name`                         | A resource is stopping.                      |

### The camera

| Event                    | Arguments        | When                                  |
| ------------------------ | ---------------- | ------------------------------------- |
| `cameraShotEnded` (both) | `player, reason` | A `Camera.startShot` ended by itself. |
| `cameraPathFinished` (client) |             | A `Camera.playPath` reached its last key; the view holds there until `Camera.endShot`. |

### The player's own game (client)

| Event                              | Arguments                  | When                                              |
| ---------------------------------- | -------------------------- | ------------------------------------------------- |
| `actionHintsChanged` (client)      |                            | The HUD's hints changed.                          |
| `bookClosed` (client)              | `id, reason`               | A book closed.                                    |
| `bookOpened` (client)              | `id`                       | A `Book` opened.                                  |
| `bookTextChanged` (client)         | `id, text, reason`         | The text in an open book changed.                 |
| `bookWritingEnded` (client)        | `id, text`                 | The player stopped writing in a book.             |
| `characterCreatorChanged` (client) | `appearance`               | The character creator's figure shows another look. |
| `characterCreatorClosed` (client)  | `appearance, reason`       | The character creator closed.                     |
| `chatKey` (client)                 |                            | The player pressed the chat key while a resource hid the game's chat (`Chat.setUIVisible(false)`). |
| `chatMessage` (client)             | `message`                  | A chat message arrived.                           |
| `cursorVisibilityChanged` (client) | `visible`                  | The mouse pointer showed or went away.            |
| `followStarted` (client)           | `target`                   | The local player's scripted follow started.       |
| `followStopped` (client)           | `target, reason`           | The local player's follow ended.                  |
| `gameResolutionChanged` (client)   | `width, height`            | The game's frame changed size.                    |
| `localPlayerKnockedOut` (client)   |                            | The local player was knocked out (`LocalPlayer.knockedOut`). |
| `localPlayerRevived` (client)      |                            | The local player, knocked out, was revived.       |
| `locationDiscovered` (client)      | `locationId`               | The player discovered a map location.             |
| `mapClosed` (client)               |                            | The local player closed the map.                  |
| `mapOpened` (client)               |                            | The local player opened the map.                  |
| `mapWaypointCleared` (client)      | `position, mapId`          | The local player removed their waypoint.          |
| `mapWaypointSet` (client)          | `position, mapId, moved`   | The local player set or moved their waypoint.     |
| `monologueSuppressed` (client)     | `text, textKey`            | KCDS silenced one of the character's remarks.     |
| `nativeMenuClosed` (client)        |                            | The game's Esc menu closed.                       |
| `nativeMenuOpened` (client)        |                            | The game's Esc menu opened.                       |
| `nativeScreenClosed` (client)      | `name`                     | One of the game's own screens went away (`NativeUI.openScreens`). |
| `nativeScreenOpened` (client)      | `name`                     | One of the game's own screens opened, such as the inventory (`ApseInventoryList`); [the names](native-ui.md#the-games-screens). |
| `noclipAllowedChanged` (client)    | `allowed`                  | The server allowed or forbade NoClip.             |
| `noclipChanged` (client)           | `enabled, reason`          | NoClip went on or off.                            |
| `poiDiscovered` (client)           | `poiId`                    | The player discovered a point of interest.        |
| `progressionLevelUp` (client)      | `track, level, perkPoints` | The local player reached a new level.             |
| `progressionPerkAdded` (client)    | `perk`                     | The local player got a perk.                      |
| `progressionPerkRefused` (client)  | `perk, reason`             | The server refused a perk they chose.             |
| `progressionPerkRemoved` (client)  | `perk`                     | The local player lost a perk.                     |
| `progressionXpGained` (client)     | `track, xp`                | The local player gained XP.                       |
| `storyCutsceneSuppressed` (client) | `cutscene`                 | KCDS stopped a quest's story cutscene.            |
| `tutorialHintClosed` (client)      | `id, reason`               | A tutorial hint closed.                           |
| `weaponHit` (client)               | `hit`                      | The local player's weapon struck the world.       |
