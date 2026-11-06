# Players

`kcds.players.Player` represents one connected player, and the object stays the same for the session. Values such as
health, position and buffs are what the player's game last reported. Changes you make go to their game and take effect a
moment later.

```js
const { Events } = kcds.core;
const { Player } = kcds.players;

Events.on('playerReady', (player) => {
  console.info(`${player.nickname} (${player.id}) is at ${player.position}`);
});

Events.on('playerCommand', (player, command, args) => {
  if (command === 'heal') {
    player.heal();
  }
  if (command === 'tp' && args[0]) {
    const [target] = Player.find(args[0]);
    if (target) {
      player.teleport(target.position);
    }
  }
});
```

```ts
const { Events } = kcds.core;
const { Player } = kcds.players;

Events.on('playerReady', (player) => {
  console.info(`${player.nickname} (${player.id}) is at ${player.position}`);
});

Events.on('playerCommand', (player, command, args) => {
  if (command === 'heal') {
    player.heal();
  }
  if (command === 'tp' && args[0]) {
    const [target] = Player.find(args[0]);
    if (target) {
      player.teleport(target.position);
    }
  }
});
```

```lua
local Events = kcds.core.Events
local Player = kcds.players.Player

Events.on("playerReady", function(player)
  print(player.nickname .. " (" .. player.id .. ") is at " .. tostring(player.position))
end)

Events.on("playerCommand", function(player, command, args)
  if command == "heal" then
    player:heal()
  end
  if command == "tp" and args[1] then
    local target = Player.find(args[1])[1]
    if target then
      player:teleport(target.position)
    end
  end
end)
```

```cs
Events.On("playerReady", (Player player) =>
{
    Log.Info($"{player.Nickname} ({player.Id}) is at {player.Position}");
});

Events.On("playerCommand", (Player player, string command, string[] args) =>
{
    if (command == "heal")
    {
        player.Heal();
    }
    if (command == "tp" && args.Length > 0)
    {
        var target = Player.Find(args[0]).FirstOrDefault();
        if (target != null)
        {
            player.Teleport(target.Position);
        }
    }
});
```

## Finding players

| Call                                          | Returns                                                     |
| --------------------------------------------- | ----------------------------------------------------------- |
| `Player.all()`                                | everyone connected, including those still loading           |
| `Player.getById(id)`                          | one by network id, or `null`                                |
| `Player.find(text)`                           | what a command argument names: an id, or matching nicknames |
| `Player.inRadius(position, radius, options?)` | players with a body in the world, nearest first             |
| `Player.nearest(position, options?)`          | the closest player with a body in the world                 |

The options are `dimension`, `except` to leave one player out, and `radius` for `nearest`.

An `id` is unique while the server runs and is never reused. `playerIndex` is the connection slot, which is reused.

## Position

- `position`, which moves them when assigned, `rotation`, `velocity` and `gaze`
- `ready`: their body is in the world
- `teleport(position, label?)`: moves them on their next frame
- `spawn(position, rotation)`: used inside `playerSpawning` to choose where they appear when joining

**Drawing things on a body:** in a client script, another player's `position` moves in steps as network updates arrive,
40 to 150 ms apart, while the game draws the body smoothly. For anything attached to the body, ask where the game is
drawing it:

- `renderPosition`: where this machine is drawing the body, or `null` before it is here
- `jointPosition(joint)`: where a joint is being drawn, or `null` if the body lacks it. Use `'Bip01 Head'`, `'Bip01 R
Hand'`, or the game's names, `'Head'` and `'RightHand'`
- `jointRotation(joint)`: how that joint is turned, as a Quaternion, or `null`. With `jointPosition` it is the joint's
  whole pose, so where a prop sits on a hand can be worked out instead of tried

```js
// client: a rope between a guard's hand and a prisoner's neck
const { Player } = kcds.players;

const hand = new Player(data.guardId).jointPosition('Bip01 R Hand');
const neck = new Player(data.prisonerId).jointPosition('Bip01 Neck');
if (hand && neck) {
  console.log(`rope ${hand.distance(neck).toFixed(2)} m`);
}
```

```ts
// client: a rope between a guard's hand and a prisoner's neck
const { Player } = kcds.players;

const hand = new Player(data.guardId).jointPosition('Bip01 R Hand');
const neck = new Player(data.prisonerId).jointPosition('Bip01 Neck');
if (hand && neck) {
  console.log(`rope ${hand.distance(neck).toFixed(2)} m`);
}
```

```lua
-- client: a rope between a guard's hand and a prisoner's neck
local Player = kcds.players.Player

local hand = Player.new(data.guardId):jointPosition("Bip01 R Hand")
local neck = Player.new(data.prisonerId):jointPosition("Bip01 Neck")
if hand and neck then
  print(string.format("rope %.2f m", hand:distance(neck)))
end
```

```cs
// client: a rope between a guard's hand and a prisoner's neck
var hand = new Player(data["guardId"]).JointPosition("Bip01 R Hand");
var neck = new Player(data["prisonerId"]).JointPosition("Bip01 Neck");
if (hand != null && neck != null)
{
    Log.Info($"rope {hand.Distance(neck):F2} m");
}
```

## Dimensions

Players and entities see each other only in the same dimension. Dimension 4294967295 is the shared world everyone starts in;
`setDimension(4294967295)` brings a player back to it. Use others, such as 0, 1 or 1000 + a player's id, for
instances, interiors, minigames or an admin's own space.

```js
player.setDimension(1000 + player.id); // a dimension of their own
player.showOnlyTo(admin); // one other player can still see them
```

```ts
player.setDimension(1000 + player.id); // a dimension of their own
player.showOnlyTo(admin); // one other player can still see them
```

```lua
player:setDimension(1000 + player.id) -- a dimension of their own
player:showOnlyTo(admin) -- one other player can still see them
```

```cs
player.SetDimension(1000 + player.Id); // a dimension of their own
player.ShowOnlyTo(admin); // one other player can still see them
```

Most spawn calls, such as `Npc.create`, `Prop.spawn` and `Vfx.spawn`, also take a `dimension`.

## Health and condition

**Read:** `health`, `maxHealth`, `healthPercent`, `stamina`, `exhaust`, `hunger`, `bleeding`, `drunkenness`, `poisoning`,
`alive`, `knockedOut`, `ableToAct`, `injuries`. Also `staminaCap` (the stamina they can reach while healthy, lowered
by injuries), `maxExhaust`, `maxHunger`, `sleeping` (non-zero while asleep), `consciousness` (0 while knocked out),
`derivedStat(name)` for a computed stat of [DerivedPlayerStat](/docs/api/derivedplayerstat), and `scaledStats` and
`scaledSkills`, the stats and skills as the game's relative values.

**The body now:** `inAir`, `crouched`, `guardUp` (weapon or fists raised), `visorDown` (helmet visor closed),
`bodyStance` (the stance tag, with `playerStanceChanged` when it changes), `fightZone` (the combat direction they aim at),
`physicsProfile`, `heldWeapon` (the weapon in the right hand) and `heldItem` (what they carry in their arms). On a client,
`movePace` and `moveDirection` are the tags this machine's animation chose.

**Change:** `setStat`, `addStat` and `fillStat` with a [PlayerStat](/docs/api/playerstat) name, `setHealth`, `setHunger`,
and `heal()`, the game's own cure.

**Knock-outs:** a fatal blow knocks a player out and fires `playerDied`, and `revive()` gets them up. Only `knockedOut`,
`setHealth(0)` and `revive()` control a knock-out. `player.buffs` does not list the unconsciousness and `addBuff` refuses
it, so a saved buff list cannot hold a player down at their next login. `setKnockoutVision(percent)` sets how much they see
while down (40 % by default; 0 is black) with the game's HUD (compass, health, effects, quick slots) hidden until they
are up again, and the game never ends a knock-out by itself while the server keeps the
player down, and `lootable` sets whether others can search them. While knocked out a player reads `alive` false and
`ableToAct` false, although the game keeps them at 1 health: wait for `alive` (or `knockedOut` false) to know they are
back up.

## Looks and body

- `setAppearance({ gender, body, head, hair, beard })` takes names from [Appearances](/docs/api/appearances);
  `Appearances.beards(head, gender)` lists the beards a face can have.
- `setOutfit(preset)` dresses them in one of the game's outfits.
- `setVisible(false)` hides the body, and `setDisguise(model)` draws it as a prop (`disguise` reads it).
- `Player.outfitSets(gender)` lists the game's outfits.
- `wearItem(item)` puts clothing on them for a while without it being in their inventory, such as a sack or shackles,
  and everyone sees it; `removeWorn(item)` takes it off and `wornItems` lists it.
- `setPhysicalProp({ kind, segments, length })` puts a rope, whip or chain in their right hand with the game's rope
  physics, seen by all; `null` takes it away and `physicalProp` reads it.
- `setPoseAnimation({ keys, blend, loop })` plays a key animation (joint turns over time) on top of the body's own
  animation, seen by all, until it is set to `null`; `poseAnimation` reads it.
- `clean()` and `soil()` change dirt and blood, and `grime` reads them.
- `setGrimeRate(rate)` scales how fast walking and riding dirty them, as the game does it: 1 is the game's rate, `0`
  none, `3` three times as fast, up to 100. `grimeRate` reads it. It shows no status entry and lasts until they leave.
- `playAnimation(fragment, options)` and `stopAnimation()` play and stop animations.
- On a client, `setCollision(false)` lets the local player walk through that player's body, as through a seated
  player's (a dice table where seated bodies blocked the way); `true` puts it back. It changes this machine only, so
  every client calls it, for example on an event the server emits when someone sits down.

A woman is put on the female skeleton, on her own screen and everyone else's, so she moves, holds weapons and rests her
hands as a woman. She keeps what she wore and held. The game's rule keeping women from alchemy tables, the grindstone,
horses, carrying bodies and pickpocketing does not apply to players.

## Removing players

- `kick(reason)` ends their session with "You were kicked from the server: ...".
- `disconnect(text, { quit })` lets them go without the word "kicked". The main menu shows `text` as written, or with
  `quit: true` their game closes. `playerDisconnect` follows, where you can save them. On the client, `kcds.core.Game.quit()`
  closes the game in the same way.
- `ban(reason)` bans them from this server by their identifiers except the address, on the list KCDS-Admin keeps. They leave
  with "You are banned from this server: <reason>." and are refused when they return.

`player.ip` (or `getIP()`) is the address their connection came from, on the server only.

For players who are not online, use `Bans.add(identifiers, reason)`. `Bans.remove(idOrIdentifier)` unbans, and `Bans.find` and
`Bans.list` read the list. A script's ban never ends by itself, so for a three-day ban, store the end date and call
`Bans.remove` when it comes.

## Restricting, carrying and attaching

Cuffs, freezing, the pillory, the pace and ending fights are on [Restricting a player](player-restrictions.md).
Carrying bodies and items, and keeping one player next to another, are on
[Carrying and attaching players](carrying.md).

## Items, skills and buffs

- `giveItem`, `takeItem`, `getInventory()` and `dropAll()`: see [Inventory](inventory.md)
- `setItemLimit(item, max)` and `itemLimit(item)`: the most units of an item this player may hold, over the server's
  item config (`null` clears it)
- `setEquippedWeightPercent(percent)` and `equippedWeightPercent`: how much of what they wear counts toward their load
- `showItemNotice(item, amount, received)`: the game's "received" or "given away" notice alone, the inventory unchanged
- `addXp`, `setLevel`, `addPerk` and others: see [Progression](progression.md)
- `addBuff`, `removeBuff`, `hasBuff`: see [Custom buffs](custom-buffs.md)

Reference: [Player](/docs/api/player), [Bans](/docs/api/bans), [Game](/docs/api/game).
