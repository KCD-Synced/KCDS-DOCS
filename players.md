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

In a client script, `LocalPlayer`'s `position`, `rotation`, `velocity`, `gaze`, health, stamina, exhaustion, hunger
and conditions come from the player's own game as it has them now, without the trip to the server and back that
other players' values take. A HUD bar or a turn the script watches follows the game's own.

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

Every player starts in the shared dimension, `World.SHARED_DIMENSION` (4294967295), where everyone sees each other.
Dimensions are unsigned 32-bit numbers (0 to 4294967295), so store them as `BIGINT` in a database, never in a signed
32-bit column such as PostgreSQL's `INTEGER`.

## A lantern in the hand

`player.setLantern(options)` lights the game's own lamp, the one its NPCs carry at night, in the player's hand, with
the game's lantern light (a warm orange, 6 m). It hangs upright from its handle, so it stays with the player on
foot, on a horse and in a cart. In the left hand the body holds it as the game's NPCs do, the arm bent and the lamp
held out; in the right hand it hangs from the hand. A rider carries it in the right hand, since the left holds the
reins, and it goes back to the chosen hand on foot. Every player sees it, later joiners too, until it is set to
`null`, the player leaves or the resource that lit it stops. A new call replaces the lantern.

## Holding a hand at a point

`player.setHandTarget(hand, options)` holds the `'right'` or `'left'` hand at a point: every player's game bends
that player's arm (shoulder, elbow, hand) each frame so the hand's grip, the point held items sit at, is there, on
top of whatever the body plays, walking and running too. The point is on the body's axes in metres (`x` to the
right, `y` forward, `z` up) from the feet, or from a joint such as `'RightArm'` with `relativeTo`. The hand keeps
the turn its animation gives it, so a cup stays upright, unless `rotation` gives one. `pole` is a point the elbow
bends toward, `weight` (0 to 1) how much of the hold is applied, and `blend` the seconds it takes to reach the hold
and to let go of it (0.2 by default). `firstPerson: { position, rotation }` is used instead only in the player's own
first-person view, so an item can sit in view without changing what others see. `null` lets the hand go, and
`player.handTargets` says where both hands are held. Everyone sees it, later joiners too.

```js
// The game's cup hold, with the mug raised in front of the chest, and higher in the player's own view.
player.playAnimation('HoldItemRight', { tags: 'r_cup', loop: true });
player.setHandTarget('right', {
  position: { x: 0.15, y: 0.45, z: 1.35 },
  firstPerson: { position: { x: 0.12, y: 0.5, z: 1.5 } },
});
// Later: let go.
player.setHandTarget('right', null);
```

```ts
// The game's cup hold, with the mug raised in front of the chest, and higher in the player's own view.
player.playAnimation('HoldItemRight', { tags: 'r_cup', loop: true });
player.setHandTarget('right', {
  position: { x: 0.15, y: 0.45, z: 1.35 },
  firstPerson: { position: { x: 0.12, y: 0.5, z: 1.5 } },
});
// Later: let go.
player.setHandTarget('right', null);
```

```lua
-- The game's cup hold, with the mug raised in front of the chest, and higher in the player's own view.
player:playAnimation("HoldItemRight", { tags = "r_cup", loop = true })
player:setHandTarget("right", {
  position = { x = 0.15, y = 0.45, z = 1.35 },
  firstPerson = { position = { x = 0.12, y = 0.5, z = 1.5 } },
})
-- Later: let go.
player:setHandTarget("right", nil)
```

```cs
// The game's cup hold, with the mug raised in front of the chest, and higher in the player's own view.
player.PlayAnimation("HoldItemRight", new { tags = "r_cup", loop = true });
player.SetHandTarget("right", new
{
    position = new { x = 0.15, y = 0.45, z = 1.35 },
    firstPerson = new { position = new { x = 0.12, y = 0.5, z = 1.5 } },
});
// Later: let go.
player.SetHandTarget("right", null);
```

To work out joints yourself, `kcds.characters.Animations.skeleton()` in a client script lists the local body's
joints with their parents and rest pose.

## What the player's game supports

`player.clientNatives` reports what the player's game build gave their client, once their world was ready:
`{ found, combat, riding, inventory, missing }`, how many of the game's functions it found, whether combat between
players works (without it no blow reaches or leaves them and `startFight` refuses them), whether riding works
(without it their client makes no horses and `mount` refuses them), whether the inventory sync works (the server
still keeps their items, but their game does not show the changes), and the names of the functions it did not find.
It is `null` before their client reported it. After a game update, a client that lost functions may not match the
game version; the server log names it too.

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
`knockOut()`, `setHealth(0)` and `revive()` control a knock-out. `player.buffs` does not list the unconsciousness and `addBuff` refuses
it, so a saved buff list cannot hold a player down at their next login. `setKnockoutVision(percent)` sets how much they see
while down (40 % by default; 0 is black) with the game's HUD (compass, health, effects, quick slots) hidden until they
are up again, and the game never ends a knock-out by itself while the server keeps the
player down, and `lootable` sets whether others can search them. While knocked out a player reads `alive` false and
`ableToAct` false, although the game keeps them at 1 health: wait for `alive` (or `knockedOut` false) to know they are
back up.

`knockOut({ health, keepEffects })` knocks a player out as a fatal blow would, with `health` left (1 by default). Their
injuries, poison and bleeding are cured first unless `keepEffects` is true. `playerDied` follows with the reason
`'knockOut'` and no killer.

Each knock-out is an episode with its own number: `playerDied`'s fourth argument, and `player.knockoutEpisode` while
the player is down (0 when they are up). `revive({ episode })` gets them up only from that knock-out. A timer or a
retry that comes late therefore cannot end a later knock-out. `playerRevived(player, episode)` fires when a revive went
out.

```js
// Knocked out by the guards: up again after a minute, unless something else got them up first.
kcds.core.Events.on('playerDied', (player, killer, reason, episode) => {
  if (reason === 'knockOut') {
    setTimeout(() => player.revive({ episode }), 60000);
  }
});
```

```ts
// Knocked out by the guards: up again after a minute, unless something else got them up first.
kcds.core.Events.on('playerDied', (player, killer, reason, episode) => {
  if (reason === 'knockOut') {
    setTimeout(() => player.revive({ episode }), 60000);
  }
});
```

```lua
-- Knocked out by the guards: up again after a minute, unless something else got them up first.
kcds.core.Events.on("playerDied", function(player, killer, reason, episode)
  if reason == "knockOut" then
    setTimeout(function() return player:revive({ episode = episode }) end, 60000)
  end
end)
```

```cs
// Knocked out by the guards: up again after a minute, unless something else got them up first.
Events.On("playerDied", (Player player, Player killer, string reason, double episode) =>
{
    if (reason == "knockOut")
    {
        Timers.SetTimeout(() => player.Revive(new { episode }), 60000);
    }
});
```

**Harm before it lands:** `playerHarm(player, harm)` asks before an NPC's blow, a fall, fire or a collision takes a
player's health. Return `false` to refuse it, or set `harm.damage` (0 takes none). `harm.reason` and `harm.bodyPart` are
as in `playerDamage`, and `harm.attacker` is the NPC that struck, or null. The player's game waits for the answer (a
round trip to the server), so keep the handler short. While no resource handles `playerHarm` nothing waits. Bleeding,
poison and starvation are never asked about, and another player's blow comes as `playerHit`.

```js
// A safe town: no fall damage, and the town's NPCs strike at half strength.
kcds.core.Events.on('playerHarm', (player, harm) => {
  if (harm.reason === 'fall') {
    return false;
  }
  if (harm.attacker) {
    harm.damage = harm.damage / 2;
  }
});
```

```ts
// A safe town: no fall damage, and the town's NPCs strike at half strength.
kcds.core.Events.on('playerHarm', (player, harm) => {
  if (harm.reason === 'fall') {
    return false;
  }
  if (harm.attacker) {
    harm.damage = harm.damage / 2;
  }
});
```

```lua
-- A safe town: no fall damage, and the town's NPCs strike at half strength.
kcds.core.Events.on("playerHarm", function(player, harm)
  if harm.reason == "fall" then
    return false
  end
  if harm.attacker then
    harm.damage = harm.damage / 2
  end
end)
```

```cs
// A safe town: no fall damage, and the town's NPCs strike at half strength.
Events.On("playerHarm", object? (Player player, JsValue harm) =>
{
    if (harm["reason"].AsString() == "fall")
    {
        return false;
    }
    if (!harm["attacker"].IsNull)
    {
        harm.Set("damage", harm["damage"].AsNumber() / 2);
    }
    return null;
});
```

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
- `crown` puts one of the game's crowns on their head over their own hair, seen by all: `'sigismund'` (Sigismund's
  crown without his hair), `'wreath'` or `'twitchWreath'` (the two wreaths for a man's head that no game item puts
  on); `null` takes it off. It is drawn over a helmet or hood too. `hairCrown = false` draws a hair that carries a
  crown, Sigismund's `m_hair_zikmund`, without it. An item registered with `crown` puts the crown on and takes it
  off when used (see [Inventory and items](/docs/inventory)).
- `clean()` and `soil()` change dirt and blood, and `grime` reads them.
- `setGrimeRate(rate)` scales how fast walking and riding dirty them, as the game does it: 1 is the game's rate, `0`
  none, `3` three times as fast, up to 100. `grimeRate` reads it. It shows no status entry and lasts until they leave.
- `playAnimation(fragment, options)` and `stopAnimation()` play and stop animations.
- On a client, `setCollision(false)` lets the local player walk through that player's body, as through a seated
  player's (a dice table where seated bodies blocked the way); `true` puts it back. It changes this machine only, so
  every client calls it, for example on an event the server emits when someone sits down.

A woman is put on the female skeleton, on her own screen and everyone else's, so she moves, holds weapons and rests her
hands as a woman. She keeps what she wore and held. The game's rule keeping women from alchemy tables, the grindstone,
horses, carrying bodies and pickpocketing does not apply to players. The game's people do not remark on a woman in a
dress as if she wore nothing on her legs (the game counts only hose as covering them); a woman with nothing on gets no
such remark either.

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
