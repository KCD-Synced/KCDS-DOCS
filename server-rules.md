# Server rules

These switches control what the game itself allows players to do on your server.

## The game's interactions

`kcds.players.Interactions` covers the game's single-player prompts. Most start switched off on a
KCDS server until a resource turns them on, for everyone or for one player.

| Name           | What it controls                                          | Starts     |
| -------------- | --------------------------------------------------------- | ---------- |
| `horseWhistle` | the companion key's whistle that calls the player's horse | off        |
| `playerTalk`   | "Talk" and focus chat on another player                   | off        |
| `trespass`     | being inside private and prohibited areas (see below)     | off        |
| `cooking`      | the prompt and use key at the game's fires                | on         |
| `smoking`      | the prompt and use key at smokehouses and drying racks    | on         |
| `npcChatter`   | the barks and lines of NPCs                               | on         |
| `pointing`     | the B key points the arm; see [Animations](animations.md) | on         |
| `npcTalk`      | "Talk" on an NPC, the game's own dialogue                 | always off |
| `npcChat`      | focus chat on an NPC, where the game provokes and insults | always off |

`npcTalk` and `npcChat` cannot be turned on, because the game's NPC conversations crashed players'
games.

```js
kcds.players.Interactions.set('horseWhistle', true);
player.setInteraction('playerTalk', true); // this player only; null returns to the server setting
```

```lua
kcds.players.Interactions.set("horseWhistle", true)
player:setInteraction("playerTalk", true) -- this player only; null returns to the server setting
```

```cs
Interactions.Set("horseWhistle", true);
player.SetInteraction("playerTalk", true); // this player only; Sdk.Null returns to the server setting
```

With `Interactions.set('smoking', false)`, the use key does nothing at smokehouses and drying racks,
so your own hints are the only prompts there. To stop players gutting animals, call
`kcds.characters.Npc.setHarvestTable({})`; see [NPCs](npcs.md).

## Private areas

The game's map marks some areas as private or prohibited, such as houses and the town jail, and warns
a player inside ("You're in a private area!", "You are trespassing"). If you allow a player there, the
warning stays off and the game's AI is told not to react to them.

```js
guard.setTrespassAllowed(true); // this player only
guard.setTrespassAllowed(null); // back to the server setting
kcds.players.Interactions.set('trespass', true); // everyone without their own setting
```

```lua
guard:setTrespassAllowed(true) -- this player only
guard:setTrespassAllowed(null) -- back to the server setting
kcds.players.Interactions.set("trespass", true) -- everyone without their own setting
```

```cs
guard.SetTrespassAllowed(true); // this player only
guard.SetTrespassAllowed(Sdk.Null); // back to the server setting
Interactions.Set("trespass", true); // everyone without their own setting
```

To open a single place, turn this on when a player enters it and off when they leave, using
[Area](/docs/api/area).

To remove only the warning, call `kcds.world.RestrictedAreaMarker.setEnabled(false)`. That affects
everyone, and the game's AI still reacts as usual.

## Camera

```js
kcds.ui.ThirdPerson.setMode('disabled'); // 'allowed' is the default (F5 switches), or 'forced'
kcds.ui.Freecam.setEnabled(false); // the game's photo mode on F1
```

```lua
kcds.ui.ThirdPerson.setMode("disabled") -- "allowed" is the default (F5 switches), or "forced"
kcds.ui.Freecam.setEnabled(false) -- the game's photo mode on F1
```

```cs
ThirdPerson.SetMode("disabled"); // "allowed" is the default (F5 switches), or "forced"
Freecam.SetEnabled(false); // the game's photo mode on F1
```

A scripted shot moves one player's camera to a point and later returns it:

```js
const { Vector3 } = kcds.math;

kcds.ui.Camera.startShot(player, {
  position: new Vector3(100, 200, 25),
  lookAt: new Vector3(110, 210, 12),
  fov: 50,
  seconds: 3,
});

setTimeout(() => kcds.ui.Camera.endShot(player), 8000);
```

```lua
local Vector3 = kcds.math.Vector3

kcds.ui.Camera.startShot(player, {
  position = Vector3.new(100, 200, 25),
  lookAt = Vector3.new(110, 210, 12),
  fov = 50,
  seconds = 3,
})

setTimeout(function() kcds.ui.Camera.endShot(player) end, 8000)
```

```cs
Camera.StartShot(player, new
{
    position = new Vector3(100, 200, 25),
    lookAt = new Vector3(110, 210, 12),
    fov = 50,
    seconds = 3,
});

Timers.SetTimeout(() => Camera.EndShot(player), 8000);
```

## Other switches

| Setting                                      | Effect                                                          |
| -------------------------------------------- | --------------------------------------------------------------- |
| `kcds.world.CombatMarker.setEnabled(false)`  | hides the crossed swords under the compass in a fight           |
| `player.setCombatMarker(false)`              | the same for one player; `null` gives them the server's setting |
| `kcds.players.Needs.setEnabled(true)`        | turns on sleep and food needs, which are off by default         |
| `kcds.players.Pickpocketing.setEnabled(...)` | see [Dice and pickpocketing](activities.md)                     |
| `kcds.players.Inventory.setStolenMarks(...)` | see [Dice and pickpocketing](activities.md)                     |

## Rules for one player

See [Restricting a player](player-restrictions.md) for `player.setPace(...)` (no running or sprinting), `setCuffed`
and `putInPillory`, and [Players](players.md) for `kick` and `setKnockoutVision`.

Reference: [Interactions](/docs/api/interactions), [ThirdPerson](/docs/api/thirdperson),
[Freecam](/docs/api/freecam), [Camera](/docs/api/camera), [CombatMarker](/docs/api/combatmarker),
[RestrictedAreaMarker](/docs/api/restrictedareamarker).
