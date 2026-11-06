# NPCs

`kcds.characters.Npc` is a character the server spawns and owns. The server holds its health, inventory and orders. The game of
the nearest player within 150 m simulates its body, including walking, pathfinding and fighting, and all other players see what
that game reports. So everyone sees the same NPC.

A player fighting an NPC takes it over: that player's game simulates it for the fight. The people standing within 30 m of
the fight go to that game too, for 45 seconds after the last blow, and so does the NPC that was fought. So the game that
has the fight also has everyone who saw it, and they react to it as the game's own people do: they scream, run away or
call the guards.

A fight between players alone is no crime to the people who see it: they report nobody and call no guards. As soon as
an NPC is in the fight, struck or striking, they report it as the game does. To punish fights between players, a server
script can watch the players' blows (`playerDamage`) and decide itself.

```js
const { Npc } = kcds.characters;
const { Vector3 } = kcds.math;

const guard = Npc.create({
  soul: 'guard',
  name: 'Gate guard',
  nameplate: true,
  position: new Vector3(-2820, -1320, 104),
});

guard.lookAt(player);
guard.say('Halt! Who goes there?');
guard.moveTo(new Vector3(-2830, -1310, 104), { speed: 'walk' });

kcds.core.Events.on('npcDeath', (npc, killer) => {
  console.info(`${npc.name} died`);
});
```

```lua
local Npc = kcds.characters.Npc
local Vector3 = kcds.math.Vector3

local guard = Npc.create({
  soul = "guard",
  name = "Gate guard",
  nameplate = true,
  position = Vector3.new(-2820, -1320, 104),
})

guard:lookAt(player)
guard:say("Halt! Who goes there?")
guard:moveTo(Vector3.new(-2830, -1310, 104), { speed = "walk" })

kcds.core.Events.on("npcDeath", function(npc, killer)
  print(npc.name .. " died")
end)
```

```cs
var guard = Npc.Create(new
{
    soul = "guard",
    name = "Gate guard",
    nameplate = true,
    position = new Vector3(-2820, -1320, 104),
});

guard.LookAt(player);
guard.Say("Halt! Who goes there?");
guard.MoveTo(new Vector3(-2830, -1310, 104), new { speed = "walk" });

Events.On("npcDeath", (Npc npc, Entity? killer) =>
{
    Log.Info($"{npc.Name} died");
});
```

## Spawning

`Npc.create(options)` requires `position`. The other options are:

- **Body:** `soul` and/or `class`, `appearance`, `outfit`, `wearing`, `faction`
- **Identity:** `name`, `nameplate`
- **Place:** `rotation`, `dimension`, `frozen`, `locomotion`
- **Health:** `health`, `invulnerable`, `lootable`

`soul` is a role or a soul GUID. `class` is `'NPC'`, `'NPC_Female'` or an animal class.

To find and remove NPCs, use `Npc.all(dimension?)`, `Npc.getById(id)`, `Npc.removeAll(dimension?)` and `npc.remove()`.

### Roles

`Npc.roles()` lists kinds of NPC by name: `guard`, `guard_alt`, `knight`, `bandit`, `bandit_alt`, `cuman`, `townsman`,
`townsman_alt`, `townswoman` and `townswoman_alt`. Each is one real soul from the game's tables, so the body already looks right,
and a woman's role has a woman's body. Pass the name as `soul`.

### Nameplates

An NPC can have a player-style name and health bar over its head. Set `nameplate: true`, which is off by default. Then use
`setNameplateText(text)` for a title instead of the name, `setNameplateColor(0xAARRGGBB)` and `setNameplateHealthVisible(false)`.

```js
const captain = kcds.characters.Npc.create({
  soul: 'knight',
  name: 'Hanush',
  nameplate: true,
  position,
});
captain.setNameplateText('Captain of the guard');
captain.setNameplateColor(0xffffd27f);
```

```lua
local captain = kcds.characters.Npc.create({
  soul = "knight",
  name = "Hanush",
  nameplate = true,
  position = position,
})
captain:setNameplateText("Captain of the guard")
captain:setNameplateColor(0xffffd27f)
```

```cs
var captain = Npc.Create(new
{
    soul = "knight",
    name = "Hanush",
    nameplate = true,
    position,
});
captain.SetNameplateText("Captain of the guard");
captain.SetNameplateColor(0xffffd27f);
```

### Which game runs it

The nearest eligible player's game simulates the NPC. That game must have finished loading the world and started gameplay, so a
connection that only receives world data cannot run NPC AI. If no eligible player holds the NPC, it stays at its last reported
state until one is available.

- `npc.pin(player)` keeps it on one player's game for a scene with them. If they walk away, it goes dormant rather than moving to
  someone else. `npc.pin(null)` releases it.
- `npc.locomotion = 'kinematic'` moves it along its orders at their pace on the ground, without walking animations. `'native'`, the
  default, makes it walk.

## Orders

Orders use the game's own movement and pathfinding.

| Order                               | Effect                                        |
| ----------------------------------- | --------------------------------------------- |
| `moveTo(position, { speed })`       | walks there: `walk` (default), `jog` or `run` |
| `follow(target, { speed, radius })` | trails a player or NPC, 3 m behind by default |
| `flee(from, { speed, radius })`     | runs away, 30 m by default                    |
| `lookAt(target)`                    | turns towards a player, NPC or position       |
| `attack(target, { weapon })`        | fights a living player or NPC within 128 m    |
| `patrol(route, options)`            | walks a route; see below                      |
| `hold()`                            | drops the order and stays still               |
| `teleport(position, rotation)`      | moves it at once                              |

`intent` shows what it was told, and `status` shows progress: `idle`, `running`, then done or failed. `npcIntentDone` fires when an
order ends. `frozen = true` holds it in place but keeps its order.

You can give an order right after `Npc.create`, and it waits for a player's game to simulate the NPC. A new body's game may refuse
a walk at first, so it is retried for a few seconds before `npcIntentDone` reports failure.

A walking NPC (`moveTo`, `flee` and a patrol's legs) steps around a player or another NPC standing in its way, passing about
1.5 m beside it, and then walks on to where it was going.

## Patrol routes

```js
const { PatrolRoute } = kcds.characters;
const { Vector3 } = kcds.math;

const wallWalk = PatrolRoute.create({
  name: 'walls',
  points: [new Vector3(0, 0, 10), new Vector3(30, 0, 10), new Vector3(30, 30, 10)],
  speed: 'walk',
  waitSeconds: 5,
});
guard.patrol(wallWalk);

kcds.core.Events.on('patrolWaypoint', (npc, patrol) => {
  console.info(`${npc.name} at ${patrol.waypointIndex}`);
});
```

```lua
local PatrolRoute = kcds.characters.PatrolRoute
local Vector3 = kcds.math.Vector3

local wallWalk = PatrolRoute.create({
  name = "walls",
  points = { Vector3.new(0, 0, 10), Vector3.new(30, 0, 10), Vector3.new(30, 30, 10) },
  speed = "walk",
  waitSeconds = 5,
})
guard:patrol(wallWalk)

kcds.core.Events.on("patrolWaypoint", function(npc, patrol)
  print(npc.name .. " at " .. patrol.waypointIndex)
end)
```

```cs
var guard = Npc.Create(new { soul = "guard", position = new Vector3(0, 0, 10) });

var wallWalk = PatrolRoute.Create(new
{
    name = "walls",
    points = new[] { new Vector3(0, 0, 10), new Vector3(30, 0, 10), new Vector3(30, 30, 10) },
    speed = "walk",
    waitSeconds = 5,
});
guard.Patrol(wallWalk);

Events.On("patrolWaypoint", (Npc npc, JsValue patrol) =>
{
    Log.Info($"{npc.Name} at {patrol["waypointIndex"]}");
});
```

An NPC copies the route when ordered, so later changes only affect new patrols. `npc.routeState` shows where it is on the route.

## Health and fighting

**Basics:** `health` (0 to 100, where 0 kills), `alive`, `invulnerable` and `fightsBack`. Also `damage(amount, attacker?)`,
`kill(killer?)`, `revive(health?)`, and `setCombatSkill({ defense, damage })`, which sets how well it parries and how hard it
hits. Events are `npcDamage`, `npcDeath` and `npcRevive`.

Three switches control behaviour. Each is true by default, and each is also an option of `Npc.create`. Animals have them too.

| Switch       | When false                                                                                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `brain`      | the game's brain does not run it; no thinking, perceiving, fighting or revenge (`fightsBack` reads false), `attack` is refused, and only your orders move it |
| `targetable` | players cannot lock on to it                                                                                                                                 |
| `attackable` | players cannot fight it: no lock-on, and their blows, shots and horses' hits land nowhere, so it does not react and the game sees no attack to punish        |

**A shopkeeper or quest giver:** use `attackable: false`, since the NPC still talks, walks and trades. Another NPC can still hit it,
so add `invulnerable` if that must not hurt it.

**A mannequin:** given to `Npc.create`, `brain: false` means the brain never starts, and all clients get the NPC with it off, so it
does not take a step or turn first. With `interactable: false` too, it is a mannequin from its first frame, suitable for a preview or
character-creation model. To make it unhurtable by blows, shots, falls and `damage()`, add `invulnerable`. Set to false later on an
NPC whose brain runs, the brain stops at once, the same as one made without it, and `moveTo` still walks it where you send it.

```js
const statue = kcds.characters.Npc.create({
  soul: 'townswoman',
  position: new kcds.math.Vector3(100, 200, 25),
  brain: false,
  targetable: false,
  invulnerable: true,
});

statue.brain = true; // it thinks again
```

```lua
local statue = kcds.characters.Npc.create({
  soul = "townswoman",
  position = kcds.math.Vector3.new(100, 200, 25),
  brain = false,
  targetable = false,
  invulnerable = true,
})

statue.brain = true -- it thinks again
```

```cs
var statue = Npc.Create(new
{
    soul = "townswoman",
    position = new Vector3(100, 200, 25),
    brain = false,
    targetable = false,
    invulnerable = true,
});

statue.Brain = true; // it thinks again
```

## Looks, loot and interaction

- `setAppearance(...)`, `setOutfit(preset)` and `wear(itemClasses)` change its look.
- `getInventory()` shows what it carries. Players can search a dead, `lootable` NPC, where `npcLoot` can refuse and `npcLooted` follows.
- `playAnimation(fragment, { tags })` plays a Mannequin fragment, as for players, and `playAnimation('', { clip })` plays a clip.
- `say(text)` speaks.
- With `interactable`, pressing the use key on it fires `npcInteract`.

## Animals

The server can populate the world with animals from the game's spawn points around players.

```js
const { Npc } = kcds.characters;

Npc.setAnimalPopulation({ enabled: true });
Npc.setAnimalSpecies('Wolf', 50); // half as many wolves
Npc.setHarvestTable({ Boar: [{ item: 'boarMeat', amount: 2 }] });
```

```lua
local Npc = kcds.characters.Npc

Npc.setAnimalPopulation({ enabled = true })
Npc.setAnimalSpecies("Wolf", 50) -- half as many wolves
Npc.setHarvestTable({ Boar = { { item = "boarMeat", amount = 2 } } })
```

```cs
Npc.SetAnimalPopulation(new { enabled = true });
Npc.SetAnimalSpecies("Wolf", 50); // half as many wolves
// A dictionary keeps its keys as written ("Boar"); an anonymous object's names start lower-case.
Npc.SetHarvestTable(new Dictionary<string, object> { ["Boar"] = new[] { new { item = "boarMeat", amount = 2 } } });
```

The classes are `Boar`, `CattleBull`, `CattleCow`, `Dog`, `Hare`, `Pig`, `Raven`, `RedDeerDoe`, `RedDeerStag`, `RoeDeerBuck`,
`RoeDeerHind`, `SheepEwe`, `SheepRam`, `WildDog` and `Wolf`. `Npc.animals()` lists the spawned animals as `Npc` objects.

### Carcasses

`npcHarvest` and `npcHarvested` follow a player skinning a carcass. A population animal is not `lootable`, so its carcass offers Gut
but not Search. Set `npc.lootable = true` on one players should search.

A dead animal with `lootable = false` and nothing to harvest offers neither prompt. That happens if its class is not in the table,
or it was already harvested. `Npc.setHarvestTable({})` turns off the game's gutting for all animals. Any hints your server puts on
the carcass are then the only prompts there.

An `npcHarvest` handler can return its own yield for this harvest, as `{ items: [{ item, amount }] }`. It can have up to 16 rows,
checked like the table's. The class must still be in the table, and `[]` is enough.

```js
kcds.core.Events.on('npcHarvest', (player, npc) => {
  if (npc.actorClass !== 'Boar' || player.getStat('survival') < 10) {
    return;
  }
  return {
    items: [
      { item: 'boarMeat', amount: 4 },
      { item: 'boarKidney', amount: 1 },
    ],
  };
});
```

```lua
kcds.core.Events.on("npcHarvest", function(player, npc)
  if npc.actorClass ~= "Boar" or player:getStat("survival") < 10 then
    return
  end
  return {
    items = {
      { item = "boarMeat", amount = 4 },
      { item = "boarKidney", amount = 1 },
    },
  }
end)
```

```cs
Events.On("npcHarvest", (Player player, Npc npc) =>
{
    if (npc.ActorClass != "Boar" || player.GetStat("survival") < 10)
    {
        return null;
    }
    return (object)new
    {
        items = new[]
        {
            new { item = "boarMeat", amount = 4 },
            new { item = "boarKidney", amount = 1 },
        },
    };
});
```

## The level's people

People the level places are server NPCs, the same for all players. They appear around players at their level positions and go when
nobody is near. They walk a relaxed round of neighbouring places, a patrol that a script can replace with any order. They do not
follow the level's daily schedules, and people a quest stages, such as an army or siege, are left out.
The level keeps many of its people at a storage spot, where its schedule picks them up, such as a line of guards in a
castle courtyard or a town street's crowd. Since these people do not follow the schedule, each such crowd (people of one
group standing within 2 m of each other) gets at most two of them. The level's stand-ins, such as a replacement guard
for one who is away, are left out. For the level's full population with its daily routines, use
[the game's own people](#the-games-own-people-levelpeople). `Npc.adopt` still takes any of them.
What they say on their own, such as a greeting to a passer-by or a line in a fight, every nearby player sees in the
same bubble over their head and hears in their own voice.
Players cannot talk to them: they show no Talk, Chat or Reply prompt and raise no `npcInteract`. A script that adopts one can
turn it on with `Npc.adopt(guid).interactable = true`.

```js
const { Npc } = kcds.characters;

Npc.setPopulation({ maxPeople: 24, activationRadius: 60, despawnRadius: 90 });
Npc.setPopulation({ share: 50 }); // half of the level's people
Npc.setPopulation({ enabled: false }); // only your own NPCs
```

```lua
local Npc = kcds.characters.Npc

Npc.setPopulation({ maxPeople = 24, activationRadius = 60, despawnRadius = 90 })
Npc.setPopulation({ share = 50 }) -- half of the level's people
Npc.setPopulation({ enabled = false }) -- only your own NPCs
```

```cs
Npc.SetPopulation(new { maxPeople = 24, activationRadius = 60, despawnRadius = 90 });
Npc.SetPopulation(new { share = 50 }); // half of the level's people
Npc.SetPopulation(new { enabled = false }); // only your own NPCs
```

`share` is the percent of the level's people who get bodies, from 0 to 100 (100 by default): with 50, half of them.
Which ones is fixed by each placement, so every player and every start sees the same people, and a smaller share only
takes people away. It counts the server's own people, not the game's own (`levelPeople`).

The same settings can go in `people-population.json` in the resources folder, for example `{ "share": 50 }`. `Npc.placements()` lists the level's people, and
`Npc.adopt(guid, options)` gives one to a script, after which it stays when players leave.

### The game's own people (`levelPeople`)

With `"levelPeople": true` in `server.json`, the people are the level's own instead: each person is the game's own character
with their daily routine (work, meals, sleep) and their own lines. One player's game runs each person near players, normally the
nearest player's, and every other player sees the same person in the same place doing the same thing. When that player leaves or
moves away, another player's game takes the person over where they are. They are server NPCs like the others: `Npc.all()` lists
them, they fight back, and players cannot talk to them. Within 150 m of a player a person is shared (up to 128 at once, which
`Npc.setPopulation` changes); farther away each game keeps them to itself until a player comes near. People a quest stages stay
off, as with the default people.

### Where the people spend the night

`npcHouses` in `server.json` says whether the level's people own houses:

- `true`, the default: they do. The game's own people (`levelPeople`) go home at night by their own
  daily routines, as in the game.
- `false`: they own none. At night each of them walks to the nearest of the level's exits, the
  points where its roads leave the map (8 on Trosecko, 13 on Kutnohorsko, the game's own), and goes
  away there. Until the morning none of the level's people comes; then they stand at their places
  again. People a script adopted (`Npc.adopt`) stay.

The night runs from 21:00 to 6:00 of the world's clock (`World.hour`). `people-night.json` in the
resources folder changes it, for example `{ "leaveHour": 22, "returnHour": 5 }` (hours from 0 up to
24; a file the server cannot use is logged and changes nothing).

### Lines the people say

Now and then one of the level's people near a player says a line to them in the game's speech bubble, and every player there
sees the same bubble. The game's own people (`levelPeople`) also keep their own barks and reactions. The lines are yours, in
`people-lines.json` in the resources folder:

```json
{
  "everySeconds": 60,
  "radius": 8,
  "seconds": 5,
  "lines": ["Good day, {player}.", "Mind the mud on the road."],
  "byName": { "Guard": ["Move along.", "Keep your nose clean, {player}."] }
}
```

- `everySeconds` (5 to 3600, 60 by default): about how often a player hears a line, from half to one and a half times it. A
  person who spoke waits as long before speaking again.
- `radius` (1 to 50 m, 8 by default): how near the person must be. The nearest one who may speak does.
- `seconds` (1 to 30, 5 by default): how long the bubble shows.
- `lines`: up to 512 lines of up to 240 bytes; `{player}` is the player's nickname. `byName` gives the people of a name lines of
  their own instead. People a script adopted (`Npc.adopt`) say none of these.
- `"enabled": false` turns the lines off. Without the file, or without lines, nobody speaks them. A file the server cannot use
  is logged and changes nothing.

## Bodies for one player

For a body only one player sees, such as a preview or a menu guide, see `LocalNpc` in
[Local NPCs and the local player](local-figures.md).

Also: `Npc.setAnimalPopulationEnabled(false)` turns the game's wild animals off (`animalPopulation` reads the settings),
`Npc.animalSpots(level)` lists the game's animal spawn points, and `Npc.harvestTable()` the skinning table. On an NPC,
`combatSkill` is `{ defense, damage }`, `harvestable` says whether its carcass can be skinned now, `lootOf()` lists the
game's loot candidates for an animal without rolling any (the script fills the carcass), and `pinnedTo` is the player
it is pinned to. `PatrolRoute.getByName(name)` finds a route. In a client script, `localNpc.setPosition(position,
rotation)` moves a local NPC.

Reference: [Npc](/docs/api/npc), [PatrolRoute](/docs/api/patrolroute), [LocalNpc](/docs/api/localnpc).
