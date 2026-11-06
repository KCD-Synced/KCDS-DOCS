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

Reference: [Player](/docs/api/player).
