# Dice and pickpocketing

## Dice

`kcds.world.Dice` lets people play on the game's dice screen. The server rolls all the dice and checks
each move against the game's rules. Players can use the game's dice kinds and badges.

### A match between two players

```js
const { Dice } = kcds.world;
const { Events } = kcds.core;
const { Player } = kcds.players;

Events.on('diceTableUse', (player, table, match) => {
  if (match !== 0) {
    return;
  }
  const opponent = Player.nearest(player.position, { radius: 4, except: player });
  if (opponent) {
    Dice.start(player.id, opponent.id, { targetScore: 1500 });
  }
});

Events.on('diceMatchEnd', (match, first, second, winner, reason) => {
  console.info(`Match ${match} ended: ${reason}`);
});
```

```ts
const { Dice } = kcds.world;
const { Events } = kcds.core;
const { Player } = kcds.players;

Events.on('diceTableUse', (player, table, match) => {
  if (match !== 0) {
    return;
  }
  const opponent = Player.nearest(player.position, { radius: 4, except: player });
  if (opponent) {
    Dice.start(player.id, opponent.id, { targetScore: 1500 });
  }
});

Events.on('diceMatchEnd', (match, first, second, winner, reason) => {
  console.info(`Match ${match} ended: ${reason}`);
});
```

```lua
local Dice = kcds.world.Dice
local Events = kcds.core.Events
local Player = kcds.players.Player

Events.on("diceTableUse", function(player, diceTable, match)
  if match ~= 0 then
    return
  end
  local opponent = Player.nearest(player.position, { radius = 4, except = player })
  if opponent then
    Dice.start(player.id, opponent.id, { targetScore = 1500 })
  end
end)

Events.on("diceMatchEnd", function(match, first, second, winner, reason)
  print("Match " .. match .. " ended: " .. reason)
end)
```

```cs
Events.On("diceTableUse", (Player player, JsValue table, double match) =>
{
    if (match != 0)
    {
        return;
    }
    var opponent = Player.Nearest(player.Position, new { radius = 4, except = player });
    if (opponent != null)
    {
        Dice.Start(player.Id, opponent.Id, new { targetScore = 1500 });
    }
});

Events.On("diceMatchEnd", (double match, Player? first, Player? second, double winner, string reason) =>
{
    Log.Info($"Match {match} ended: {reason}");
});
```

`Dice.start(firstId, secondId, options)` requires both players to be within 4 m of each other at a
table. Its options are `targetScore` (default 2000), `firstName`, `secondName`, `firstDice`,
`secondDice`, `firstBadge` and `secondBadge`. To set the points or anything else of a match, also one the shipped dice resource starts when a
second player sits down, change `options` in the `diceMatchStarting` event. Players within 20 m can watch. `Dice.spectate(player, match)` gives a player a camera right above
the dice, from any distance, until the match ends or `Dice.endSpectate(player)`.

A match ends when someone reaches the target score, or with one of these reasons: `gaveUp`, `left`,
`timedOut` (3 minutes without a move), `failed`, or `stopped` (from `Dice.stop(match)`).

To look things up, use `Dice.tables()`, `Dice.kinds()`, `Dice.badges()` and `Dice.gameOf(playerId)`.

### A match against an NPC

`Dice.startAgainstNpc(player, options)` matches a player at a table against an opponent the server
plays. The server makes each throw and turn, and the player's game shows them like another player's.

```js
kcds.world.Dice.startAgainstNpc(player, {
  difficulty: 'hard',
  npcName: 'Gambler',
  targetScore: 1500,
});
```

```ts
kcds.world.Dice.startAgainstNpc(player, {
  difficulty: 'hard',
  npcName: 'Gambler',
  targetScore: 1500,
});
```

```lua
kcds.world.Dice.startAgainstNpc(player, {
  difficulty = "hard",
  npcName = "Gambler",
  targetScore = 1500,
})
```

```cs
Dice.StartAgainstNpc(player, new
{
    difficulty = "hard",
    npcName = "Gambler",
    targetScore = 1500,
});
```

| Option                                | Meaning                                                   |
| ------------------------------------- | --------------------------------------------------------- |
| `difficulty`                          | `'easy'`, `'normal'` (default), `'hard'`, or a skill from 0 to 100 |
| `npc`                                 | an NPC to use; otherwise the server makes a stand-in      |
| `npcName`, `playerName`               | names to show                                             |
| `targetScore`                         | the score to reach                                        |
| `playerDice`, `npcDice`               | the dice each side uses                                   |
| `playerBadge`, `npcBadge`             | badges; the NPC plays passive ones only                   |

A skilled opponent keeps its most valuable dice and takes more risks when far behind. An easy one banks
early and sometimes keeps fewer dice than it could.

`diceMatchEnd` fires as usual. The NPC's seat holds `null` instead of a player, so pay any bet according
to the winning seat. The server's `dice` resource starts this kind of match with
`/dicenpc [easy|normal|hard]`.

The stand-in sits on the table's free chair with the game's animation. Everyone sees it there, and the
player's game runs it until the match ends.

To seat your own NPC smoothly, walk it to where a free chair's sit-down begins before starting the
match. `Dice.tables()` gives each level table's `chairs`, as `{ position, player, sitFrom }`, where
`player` is `null` for a free chair. Pin the NPC to the player, because only the game running an NPC can
sit it down:

```js
const { Dice } = kcds.world;
const { Npc } = kcds.characters;
const { Player } = kcds.players;

const table = Dice.tables().find((row) => row.chairs.some((chair) => chair.player === null));
if (table) {
  const chair = table.chairs.find((candidate) => candidate.player === null);
  const gambler = Npc.create({ position: table.position });
  gambler.pin(Player.all()[0]);
  gambler.moveTo(chair.sitFrom.position, { speed: 'walk' });
  // When npcIntentDone fires: call gambler.lookAt(chair.sitFrom.lookAt), then start the match.
}
```

```ts
const { Dice } = kcds.world;
const { Npc } = kcds.characters;
const { Player } = kcds.players;

interface Chair {
  player: number | null;
  sitFrom: { position: kcds.math.Vector3; lookAt: kcds.math.Vector3 };
}

const table = Dice.tables().find((row) => row.chairs.some((chair: Chair) => chair.player === null));
const chair: Chair | undefined = table?.chairs.find((candidate: Chair) => candidate.player === null);
if (table && chair) {
  const gambler = Npc.create({ position: table.position });
  gambler.pin(Player.all()[0]);
  gambler.moveTo(chair.sitFrom.position, { speed: 'walk' });
  // When npcIntentDone fires: call gambler.lookAt(chair.sitFrom.lookAt), then start the match.
}
```

```lua
local Dice = kcds.world.Dice
local Npc = kcds.characters.Npc
local Player = kcds.players.Player

local function freeChair(diceTable)
  for _, chair in ipairs(diceTable.chairs) do
    if chair.player == nil then
      return chair
    end
  end
  return nil
end

for _, diceTable in ipairs(Dice.tables()) do
  local chair = freeChair(diceTable)
  if chair then
    local gambler = Npc.create({ position = diceTable.position })
    gambler:pin(Player.all()[1])
    gambler:moveTo(chair.sitFrom.position, { speed = "walk" })
    -- When npcIntentDone fires: call gambler:lookAt(chair.sitFrom.lookAt), then start the match.
    break
  end
end
```

```cs
var table = Dice.Tables().FirstOrDefault((row) => row["chairs"].Any((chair) => chair["player"].IsNull));
if (!table.IsNull)
{
    var chair = table["chairs"].First((candidate) => candidate["player"].IsNull);
    var gambler = Npc.Create(new { position = table["position"] });
    gambler.Pin(Player.All()[0]);
    gambler.MoveTo(chair["sitFrom"]["position"], new { speed = "walk" });
    // When npcIntentDone fires: call gambler.LookAt(chair["sitFrom"]["lookAt"]), then start the match.
}
```

### Sitting down first

A player can take one of a table's two chairs and wait. Their game walks them there and plays the
sit-down. A match between the two seated players plays from their chairs, and both stay seated after it.

```js
const { Dice } = kcds.world;
const { Events, Chat } = kcds.core;
const { Player } = kcds.players;

Events.on('diceTableUse', (player, table, match) => {
  if (match === 0 && Dice.sit(player.id) < 0) {
    Chat.sendToPlayer(player, 'Both chairs are taken.');
  }
});

setInterval(() => {
  for (const table of Dice.tables()) {
    const [first, second] = Dice.sittersAt(table.id).map((id) => {
      return id === null ? null : Player.getById(id);
    });
    const bothSeated = first?.seated && second?.seated;
    if (bothSeated && Dice.gameOf(first.id) === 0 && Dice.gameOf(second.id) === 0) {
      Dice.start(first.id, second.id);
    }
  }
}, 500);
```

```ts
const { Dice } = kcds.world;
const { Events, Chat } = kcds.core;
const { Player } = kcds.players;

Events.on('diceTableUse', (player, table, match) => {
  if (match === 0 && Dice.sit(player.id) < 0) {
    Chat.sendToPlayer(player, 'Both chairs are taken.');
  }
});

setInterval(() => {
  for (const table of Dice.tables()) {
    const [first, second] = Dice.sittersAt(table.id).map((id) => {
      return id === null ? null : Player.getById(id);
    });
    const bothSeated = first?.seated && second?.seated;
    if (bothSeated && Dice.gameOf(first.id) === 0 && Dice.gameOf(second.id) === 0) {
      Dice.start(first.id, second.id);
    }
  }
}, 500);
```

```lua
local Dice = kcds.world.Dice
local Events, Chat = kcds.core.Events, kcds.core.Chat
local Player = kcds.players.Player

Events.on("diceTableUse", function(player, diceTable, match)
  if match == 0 and Dice.sit(player.id) < 0 then
    Chat.sendToPlayer(player, "Both chairs are taken.")
  end
end)

setInterval(function()
  for _, diceTable in ipairs(Dice.tables()) do
    local sitters = Dice.sittersAt(diceTable.id)
    local first = sitters[1] and Player.getById(sitters[1])
    local second = sitters[2] and Player.getById(sitters[2])
    local bothSeated = first and first.seated and second and second.seated
    if bothSeated and Dice.gameOf(first.id) == 0 and Dice.gameOf(second.id) == 0 then
      Dice.start(first.id, second.id)
    end
  end
end, 500)
```

```cs
Events.On("diceTableUse", (Player player, JsValue table, double match) =>
{
    if (match == 0 && Dice.Sit(player.Id) < 0)
    {
        Chat.SendToPlayer(player, "Both chairs are taken.");
    }
});

Timers.SetInterval(() =>
{
    foreach (var table in Dice.Tables())
    {
        var sitters = Dice.SittersAt(table["id"])
            .Select((id) => id.IsNull ? null : Player.GetById(id))
            .ToArray();
        var first = sitters.ElementAtOrDefault(0);
        var second = sitters.ElementAtOrDefault(1);
        var bothSeated = first?.Seated == true && second?.Seated == true;
        if (bothSeated && Dice.GameOf(first!.Id) == 0 && Dice.GameOf(second!.Id) == 0)
        {
            Dice.Start(first.Id, second.Id);
        }
    }
}, 500);
```

- `Dice.sit(playerId)` returns the chair, 0 or 1, that the player sits on or already holds. It returns
  -1 if they are playing, not at a table, or both chairs are taken.
- `Dice.sittersAt(tableId)` lists who holds each chair, with `null` for a free one. A player walking to a
  chair already holds it, and `player.seated` turns true when they sit.
- A chair frees up when its player stands up or leaves. The walk and sit-down can take up to 10 seconds
  first.

The `resources/dice` resource that comes with the server works this way. <!-- only examples -->
Using a table (E, "Play dice")
sits you down, and the match begins when a second player sits on the other chair.

Also: `Dice.setNames(match, first, second)` changes the names both games show, `Dice.badgesOf(match)`,
`Dice.badgeUses(match)` and `Dice.diceOf(match)` read each seat's badge, badge uses left and dice, and
`Dice.watchers(match)` lists who watches a match.

Reference: [Dice](/docs/api/dice).

## Pickpocketing

`kcds.players.Pickpocketing` lets players rob each other using the game's own pickpocketing. The
thief's game runs the minigame, and the server decides the result.

```js
const { Pickpocketing } = kcds.players;

Pickpocketing.setEnabled(true);
Pickpocketing.setVictimNotice(true, '{thief} had a hand in your pocket!');

kcds.core.Events.on('playerPickpocketStart', (thief, victim) => {
  return thief.dimension === victim.dimension;
});
```

```ts
const { Pickpocketing } = kcds.players;

Pickpocketing.setEnabled(true);
Pickpocketing.setVictimNotice(true, '{thief} had a hand in your pocket!');

kcds.core.Events.on('playerPickpocketStart', (thief, victim) => {
  return thief.dimension === victim.dimension;
});
```

```lua
local Pickpocketing = kcds.players.Pickpocketing

Pickpocketing.setEnabled(true)
Pickpocketing.setVictimNotice(true, "{thief} had a hand in your pocket!")

kcds.core.Events.on("playerPickpocketStart", function(thief, victim)
  return thief.dimension == victim.dimension
end)
```

```cs
Pickpocketing.SetEnabled(true);
Pickpocketing.SetVictimNotice(true, "{thief} had a hand in your pocket!");

Events.On("playerPickpocketStart", (Player thief, Player victim) =>
{
    return thief.Dimension == victim.Dimension;
});
```

Returning `false` from `playerPickpocketStart` refuses the attempt. The other events are
`playerPickpocketCaught` and `playerPickpocketed`. `kcds.players.Inventory.setStolenMarks(enabled)`
turns the game's stolen marks on or off, and `Inventory.stolenMarks()` reads it. `Pickpocketing.victimNotice()` and
`Pickpocketing.victimNoticeText()` read whether a player who catches a thief is told, and with which text.

Reference: [Pickpocketing](/docs/api/pickpocketing).
