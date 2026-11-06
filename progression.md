# Skills, perks and needs

The server has the final say on progression. Each XP gain in a player's game is requested from the
server and granted under your rules. If progress appears that the server never allowed, it is caught,
and `playerProgressionRejected` fires.

XP goes to **tracks**:

- **Stats:** `strength`, `agility`, `vitality`, `speech`
- **Skills:** `stealth`, `horse_riding`, `fencing`, `thievery`, `alchemy`, `craftsmanship`,
  `drinking`, `survival`, `weapon_sword`, `heavy_weapons`, `marksmanship`, `weapon_large`,
  `weapon_unarmed`, `scholarship`, `houndmaster`, `prestige`

## Rules for the whole server

```js
const { Progression } = kcds.players;

Progression.setXpGain('*', 0.5); // half XP on all tracks
Progression.setXpGain('alchemy', 2); // double on this one
Progression.setXpCap('*', 300); // up to 300 XP per minute on each track
Progression.setMaxLevel('weapon_sword', 20);
Progression.lockPerks(['Art of preservation']); // perk names or GUIDs
Progression.setGameXp(false); // only XP your scripts give counts
```

```lua
local Progression = kcds.players.Progression

Progression.setXpGain("*", 0.5) -- half XP on all tracks
Progression.setXpGain("alchemy", 2) -- double on this one
Progression.setXpCap("*", 300) -- up to 300 XP per minute on each track
Progression.setMaxLevel("weapon_sword", 20)
Progression.lockPerks({ "Art of preservation" }) -- perk names or GUIDs
Progression.setGameXp(false) -- only XP your scripts give counts
```

```cs
Progression.SetXpGain("*", 0.5); // half XP on all tracks
Progression.SetXpGain("alchemy", 2); // double on this one
Progression.SetXpCap("*", 300); // up to 300 XP per minute on each track
Progression.SetMaxLevel("weapon_sword", 20);
Progression.LockPerks(new[] { "Art of preservation" }); // perk names or GUIDs
Progression.SetGameXp(false); // only XP your scripts give counts
```

A perk's texts are the server's to change too: `Progression.setPerkText(perk, { name, description, lore })` renames it and
rewrites its description and lore in every player's perk tree, and players who join later read the same. `null` gives one
text back to the game.

```js
const { Progression } = kcds.players;

Progression.setPerkText('Against all odds', {
  name: "Racek's legacy",
  description: 'You fight on when others would fall.',
});
```

```lua
local Progression = kcds.players.Progression

Progression.setPerkText("Against all odds", {
  name = "Racek's legacy",
  description = "You fight on when others would fall.",
})
```

```cs
Progression.SetPerkText("Against all odds", new
{
    name = "Racek's legacy",
    description = "You fight on when others would fall.",
});
```

Two events let you decide case by case. Return `false` from `playerXpGaining` to refuse an XP gain, or
from `playerPerkLearning` to refuse a perk chosen on the perk screen.

## One player

| Goal      | Members                                                                                                           |
| --------- | ----------------------------------------------------------------------------------------------------------------- |
| Read      | `player.level`, `player.levels`, `player.skills`, `player.trackOf(track)`, `player.perks`, `player.hasPerk(perk)` |
| Give      | `addXp(track, xp)`, `setLevel(track, level)`, `addPerk(perk)`, `addPerkPoints(track, count)`                      |
| Take away | `removePerk(perk)`, `resetPerks()`                                                                                |
| Exact XP  | `setXp(track, totalXp)`, `removeXp(track, xp)`                                                                    |
| Recipes   | `learnRecipe(id)`, `forgetRecipe(id)`, `knowsRecipe(id)`; ids from [Recipe](/docs/api/recipe)                     |

`setXp` sets a track's total XP counted from level 0. `removeXp` subtracts from the amount the
player's game last reported. With both, the level moves up or down as needed, without rewards, points
or a notice. `Progression.xpNeeded(track, level)` gives the XP a level costs. Before version 0.3.4,
`setXp` set the XP within the current level.

## Saving progress between visits

Store `player.progression` when a player leaves, and pass it to `loadProgression` when they return:

```js
const { Events } = kcds.core;
const savedProgress = new Map();

Events.on('playerDisconnect', (player) => {
  savedProgress.set(player.nickname, player.progression);
});

Events.on('playerReady', (player) => {
  const snapshot = savedProgress.get(player.nickname);
  if (snapshot) {
    player.loadProgression(snapshot, { notify: false });
  }
});
```

```lua
local Events = kcds.core.Events
local savedProgress = {}

Events.on("playerDisconnect", function(player)
  savedProgress[player.nickname] = player.progression
end)

Events.on("playerReady", function(player)
  local snapshot = savedProgress[player.nickname]
  if snapshot then
    player:loadProgression(snapshot, { notify = false })
  end
end)
```

```cs
var savedProgress = new Dictionary<string, JsValue>();

Events.On("playerDisconnect", (Player player) =>
{
    savedProgress[player.Nickname] = player.Progression;
});

Events.On("playerReady", (Player player) =>
{
    if (savedProgress.TryGetValue(player.Nickname, out var snapshot))
    {
        player.LoadProgression(snapshot, new { notify = false });
    }
});
```

This map is lost when the server stops, so save to your own storage for anything lasting.

## Events and the catalogue

Events: `playerXpGained`, `playerLevelUp`, `playerPerkAdded`, `playerPerkRemoved`.

To look things up: `Progression.tracks()`, `Progression.perks(filter)`,
`Progression.perkNamed(query)`, `Progression.xpNeeded(track, level)`.

## Perks of your own

A resource can add its own perks to the game's perk tree, with their own icons and effects, and
give them with `player.addPerk` like the game's perks: see [Custom perks](custom-perks.md).

## Status effects

`player.addBuff(name)`, `removeBuff`, `hasBuff` and `clearBuffs(tag)` use the game's buffs, which you
can find with `kcds.players.Buffs.find`. You can also make your own: [Custom buffs](custom-buffs.md).

With `kcds.players.Buffs.claim(classes)`, the server takes over whole kinds of effect. For example,
players' games then stop applying `alcohol` or `poison` buffs by themselves, and `playerBuffBlocked`
tells you when one tried.

## Sleep and food

These needs are off by default, so nobody gets tired or hungry. `kcds.players.Needs.setEnabled(true)`
turns the game's needs on for everyone. `setEnabled(false)` turns them off again and fills both.

Also: `Progression.setXpPercent(track, percent)` sets a track's XP rule as a percentage (100 is the game's rate) and
`xpPercent(track)` reads it; `unlockPerks(perks)` allows locked perks again and `lockedPerks()` lists them; `gameXp()`
says whether players earn XP by playing.

Reference: [Progression](/docs/api/progression), [Player](/docs/api/player),
[Buffs](/docs/api/buffs), [Needs](/docs/api/needs).
