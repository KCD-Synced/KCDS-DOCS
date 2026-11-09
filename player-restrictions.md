# Restricting a player

What a server can stop a player from doing, one player at a time. The rules for everyone are in
[Server rules](server-rules.md).

| Member                                        | Effect                                                                                                                                   |
| --------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `setCuffed(true)`                             | hands behind the back; they can only walk and look, and only the upper body takes the pose                                               |
| `noclipAllowed = false`                       | no flying with `NoClip`; true by default and again on rejoining, so set it in `playerJoin` ([Free camera](client-camera.md#free-camera)) |
| `frozen = true`                               | held in place; see below                                                                                                                 |
| `putInPillory(name?)`, `releaseFromPillory()` | the pillory                                                                                                                              |
| `setPace(mode)`                               | forbids running or sprinting                                                                                                             |
| `startFight(opponent)`                        | starts a fight between two players at once; see below; `fights` lists the opponents                                                    |
| `stopFight(opponent)`                         | ends a fight                                                                                                                             |
| `makePeace()`                                 | ends every fight they are in; `opponents` lists who they fought in the last 15 s                                                         |
| `setInteraction(name, allowed)`               | the game's interactions for this player; see [Server rules](server-rules.md); `interactionAllowed(name)` reads one                       |
| `trespassAllowed`, `combatMarker`             | read: whether they may be in private areas now, and whether they see the combat marker                                                   |
| `setLockable(false)` (client)                 | combat targeting on this machine cannot pick that player, as for someone lying wounded                                                   |

**`frozen`** works like `Npc.frozen`. Their game lets through only looking around and menus, and a modified game that moves
the body anyway is put back within a second. A script's teleport moves the spot where they are held. It ends when they leave
the server.

**Locking on:** a player struck in melee by another player while locked on nobody locks on the striker, as the game
locks on an NPC that attacks. `stopFight` and `makePeace` drop the lock again.

**Starting a fight:** `startFight(opponent)` starts melee combat between two players at once: both games lock on the
other, as the game's own lock does, and `playerFightStarted(player, opponent)` fires. The fight lasts until
`stopFight`, a knock-out of either or a disconnect, and `playerFightEnded(player, opponent, reason)` tells which
(`'stopped'`, `'knockedOut'` or `'left'`). `player.fights` lists the opponents of the fights a script started. Ask both
players first, for example with a duel offer the other accepts: the fight does not wait for them. It returns false
for a player who is gone, not playing or knocked out, the same player, another dimension, more than 15 m apart, or a
pair that fights already.

```js
// server: a duel both players agreed to starts at once
const { Events, Chat } = kcds.core;
const offers = new Map();

Events.fromClient('duel', (player, target) => {
  const opponent = kcds.players.Player.getById(Number(target));
  if (!opponent) {
    return;
  }
  if (offers.get(opponent.id) === player.id) {
    offers.delete(opponent.id);
    player.startFight(opponent);
    return;
  }
  offers.set(player.id, opponent.id);
});

Events.on('playerFightEnded', (player, opponent, reason) => {
  if (reason === 'knockedOut') {
    const winner = player.knockedOut ? opponent : player;
    Chat.sendToAll(`${winner.nickname} won the duel`);
  }
});
```

```ts
// server: a duel both players agreed to starts at once
const { Events, Chat } = kcds.core;
const offers = new Map<number, number>();

Events.fromClient('duel', (player, target) => {
  const opponent = kcds.players.Player.getById(Number(target));
  if (!opponent) {
    return;
  }
  if (offers.get(opponent.id) === player.id) {
    offers.delete(opponent.id);
    player.startFight(opponent);
    return;
  }
  offers.set(player.id, opponent.id);
});

Events.on('playerFightEnded', (player, opponent, reason) => {
  if (reason === 'knockedOut') {
    const winner = player.knockedOut ? opponent : player;
    Chat.sendToAll(`${winner.nickname} won the duel`);
  }
});
```

```lua
-- server: a duel both players agreed to starts at once
local Events, Chat = kcds.core.Events, kcds.core.Chat
local offers = {}

Events.fromClient("duel", function(player, target)
  local opponent = kcds.players.Player.getById(tonumber(target))
  if not opponent then
    return
  end
  if offers[opponent.id] == player.id then
    offers[opponent.id] = nil
    player:startFight(opponent)
    return
  end
  offers[player.id] = opponent.id
end)

Events.on("playerFightEnded", function(player, opponent, reason)
  if reason == "knockedOut" then
    local winner = player.knockedOut and opponent or player
    Chat.sendToAll(winner.nickname .. " won the duel")
  end
end)
```

```cs
// server: a duel both players agreed to starts at once
var offers = new Dictionary<double, double>();

Events.FromClient("duel", (Player player, JsValue target) =>
{
    var opponent = Player.GetById(target.AsNumber());
    if (opponent == null)
    {
        return;
    }
    if (offers.TryGetValue(opponent.Id, out var asked) && asked == player.Id)
    {
        offers.Remove(opponent.Id);
        player.StartFight(opponent);
        return;
    }
    offers[player.Id] = opponent.Id;
});

Events.On("playerFightEnded", (Player player, Player opponent, string reason) =>
{
    if (reason == "knockedOut")
    {
        var winner = player.KnockedOut ? opponent : player;
        Chat.SendToAll($"{winner.Nickname} won the duel");
    }
});
```

**Making peace:** `makePeace()` ends every fight the player is in, with each of `opponents`, the players they swung at
or who swung at them in the last 15 seconds. Offer it on a key the player holds:

```js
// client
const { Keybinds } = kcds.ui;
Keybinds.add('make_peace', { label: 'Make peace', key: 'h' });
let since = 0;
Keybinds.on('make_peace', (id, state) => {
  if (state === 'down') {
    since = Date.now();
  } else if (Date.now() - since >= 1000) {
    kcds.core.Events.toServer('peace');
  }
});
```

```ts
// client
const { Keybinds } = kcds.ui;
Keybinds.add('make_peace', { label: 'Make peace', key: 'h' });
let since = 0;
Keybinds.on('make_peace', (id, state) => {
  if (state === 'down') {
    since = Date.now();
  } else if (Date.now() - since >= 1000) {
    kcds.core.Events.toServer('peace');
  }
});
```

```lua
-- client
local Keybinds = kcds.ui.Keybinds
Keybinds.add("make_peace", { label = "Make peace", key = "h" })
-- held for a second: a timer marks it
local held = false
local timer = nil
Keybinds.on("make_peace", function(id, state)
  if state == "down" then
    held = false
    timer = setTimeout(function()
      held = true
    end, 1000)
  else
    clearTimeout(timer)
    if held then
      kcds.core.Events.toServer("peace")
    end
  end
end)
```

```cs
// client
Keybinds.Add("make_peace", new { label = "Make peace", key = "h" });
long since = 0;
Keybinds.On("make_peace", new Action<string, string>((id, state) =>
{
    var now = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds();
    if (state == "down")
    {
        since = now;
    }
    else if (now - since >= 1000)
    {
        Events.ToServer("peace");
    }
}));
```

```js
kcds.core.Events.fromClient('peace', (player) => {
  player.makePeace();
});
```

```ts
kcds.core.Events.fromClient('peace', (player) => {
  player.makePeace();
});
```

```lua
kcds.core.Events.fromClient("peace", function(player)
  player:makePeace()
end)
```

```cs
Events.FromClient("peace", (Player player) =>
{
    player.MakePeace();
});
```

## Leading on a rope

`player.setLeadRope(target, options?)` gives the player a rope from their hand to another body, such as a guard leading a
cuffed prisoner or a herder an animal. The target is a player, an NPC or a horse. Every player sees it, those who join
later too: it hangs while the two stand close and pulls straight at its length. It is only drawn, so it holds nobody back;
pair it with `setCuffed` or `setPace` for a prisoner who must keep up. `null` takes it away, and a player has one rope at a
time. `player.leadRope` reads it as `{ target, from, to, length }`, or `null`. A horse that follows someone gets its own rope
with `horse.follow(target, { rope: true })` ([Horses](horses-dogs.md#orders)).

| Option   | Default   | Meaning                                                                       |
| -------- | --------- | ----------------------------------------------------------------------------- |
| `from`   | `'right'` | `'right'`, `'left'` or a joint of the leading player's body                   |
| `to`     | `'Spine'` | a joint of the target's body (the waist); a name it does not have falls back to it |
| `length` | 2.5       | metres, 0.5 to 20                                                             |

```js
// /lead <nickname> ties the named player to you; /lead alone lets go.
kcds.core.Events.on('playerCommand', (guard, command, args) => {
  if (command !== 'lead') {
    return;
  }
  const prisoner = kcds.players.Player.all().find((p) => p.nickname === args[0]);
  guard.setLeadRope(prisoner ?? null, { to: 'Spine', length: 2 });
});
```

```ts
// /lead <nickname> ties the named player to you; /lead alone lets go.
kcds.core.Events.on('playerCommand', (guard, command, args) => {
  if (command !== 'lead') {
    return;
  }
  const prisoner = kcds.players.Player.all().find((p) => p.nickname === args[0]);
  guard.setLeadRope(prisoner ?? null, { to: 'Spine', length: 2 });
});
```

```lua
-- /lead <nickname> ties the named player to you; /lead alone lets go.
kcds.core.Events.on("playerCommand", function(guard, command, args)
  if command ~= "lead" then
    return
  end
  local prisoner = nil
  for _, p in ipairs(kcds.players.Player.all()) do
    if p.nickname == args[1] then
      prisoner = p
    end
  end
  guard:setLeadRope(prisoner, { to = "Spine", length = 2 })
end)
```

```cs
// /lead <nickname> ties the named player to you; /lead alone lets go.
Events.On("playerCommand", (Player guard, string command, string[] args) =>
{
    if (command != "lead")
    {
        return;
    }
    var prisoner = Player.All().FirstOrDefault((p) => args.Length > 0 && p.Nickname == args[0]);
    guard.SetLeadRope(prisoner, new { to = "Spine", length = 2 });
});
```

Reference: [Player](/docs/api/player).
