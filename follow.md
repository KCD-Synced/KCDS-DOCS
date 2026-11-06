# Following someone

A client script can make its player follow another player or a server NPC, using the same behaviour
the game uses when an NPC leads the player somewhere. The game walks the player to a spot about 1.5 m
behind the target and keeps moving as the target moves.

```js
// client
const { Follow } = kcds.characters;
const { Hud, Key } = kcds.ui;
const { Events } = kcds.core;

// F starts following the nearest person within 20 m; F again stops.
Key.bind('f', () => {
  if (Follow.getTarget() || Follow.isPending()) {
    Follow.stop();
    return;
  }
  const nearby = Follow.targets();
  if (nearby.length > 0 && Follow.canStart(nearby[0])) {
    Follow.start(nearby[0]);
  }
});

Events.on('followStarted', (target) => {
  Hud.showNotification(`Following ${target.nickname}`);
});

Events.on('followStopped', (target, reason) => {
  if (reason === 'targetLost') {
    Hud.showNotification(`${target.nickname} is gone`);
  }
});
```

```ts
// client
const { Follow } = kcds.characters;
const { Hud, Key } = kcds.ui;
const { Events } = kcds.core;

// F starts following the nearest person within 20 m; F again stops.
Key.bind('f', () => {
  if (Follow.getTarget() || Follow.isPending()) {
    Follow.stop();
    return;
  }
  const nearby = Follow.targets();
  if (nearby.length > 0 && Follow.canStart(nearby[0])) {
    Follow.start(nearby[0]);
  }
});

Events.on('followStarted', (target) => {
  Hud.showNotification(`Following ${target.nickname}`);
});

Events.on('followStopped', (target, reason) => {
  if (reason === 'targetLost') {
    Hud.showNotification(`${target.nickname} is gone`);
  }
});
```

```lua
-- client
local Follow = kcds.characters.Follow
local Hud, Key = kcds.ui.Hud, kcds.ui.Key
local Events = kcds.core.Events

-- F starts following the nearest person within 20 m; F again stops.
Key.bind("f", function()
  if Follow.getTarget() or Follow.isPending() then
    Follow.stop()
    return
  end
  local nearby = Follow.targets()
  if #nearby > 0 and Follow.canStart(nearby[1]) then
    Follow.start(nearby[1])
  end
end)

Events.on("followStarted", function(target)
  Hud.showNotification("Following " .. target.nickname)
end)

Events.on("followStopped", function(target, reason)
  if reason == "targetLost" then
    Hud.showNotification(target.nickname .. " is gone")
  end
end)
```

```cs
// client
// F starts following the nearest person within 20 m; F again stops.
Key.Bind("f", new Action(() =>
{
    if (!Follow.GetTarget().IsNull || Follow.IsPending())
    {
        Follow.Stop();
        return;
    }
    var nearby = Follow.Targets();
    if (nearby.Length > 0 && Follow.CanStart(nearby[0]))
    {
        Follow.Start(nearby[0]);
    }
}));

Events.On("followStarted", (JsValue target) =>
{
    Hud.ShowNotification($"Following {target["nickname"]}");
});

Events.On("followStopped", (JsValue target, string reason) =>
{
    if (reason == "targetLost")
    {
        Hud.ShowNotification($"{target["nickname"]} is gone");
    }
});
```

## Finding a target

- `Follow.targets()` lists players and server NPCs within 20 m who can be followed, as
  `{ id, kind: 'player' | 'npc', name, position }`.
- `Follow.canStart(target)` tells whether the game would allow a follow right now, including its own
  checks and the distance.

Anywhere a target is expected, you can pass a `Player`, an entry from `targets()` or a network id.

## Starting

`Follow.start(target, options?)` queues the follow, and `followStarted` fires when the game's follow
is actually running. It returns `false` if the target is missing or the game refuses. Only one follow
can run at a time, so a new one ends the current one with the reason `script`.

| Option          | Default | Effect                                                               |
| --------------- | ------- | -------------------------------------------------------------------- |
| `hint`          | `true`  | `false` hides the game's "Stop following" prompt                     |
| `playerCanStop` | `true`  | `false` takes that key and the movement keys while the follow runs   |
| `distance`      |         | how close the follower keeps, 0.5 to 10 m                            |
| `pace`          |         | `'run'` lets the follower run when the leader runs                   |

With `playerCanStop: false`, the follow ends only through `Follow.stop()` or one of the automatic
reasons below.

**Why set a distance?** Without it, the game's follow stops at 1.5 m but only sets off again once the
leader is 5 m away, so the follower trails a walking leader by 5 to 8 m. With a `distance`, the
follower stops at that distance, sets off when half a metre beyond it, and jogs when a metre behind.
A cuffed player only walks.

To lead a prisoner 1.5 m behind a guard, with no prompt and no way to walk off:

```js
// client
kcds.characters.Follow.start(guard, { hint: false, playerCanStop: false, distance: 1.5 });
```

```ts
// client
kcds.characters.Follow.start(guard, { hint: false, playerCanStop: false, distance: 1.5 });
```

```lua
-- client
kcds.characters.Follow.start(guard, { hint = false, playerCanStop = false, distance = 1.5 })
```

```cs
// client
Follow.Start(guard, new { hint = false, playerCanStop = false, distance = 1.5 });
```

## Checking and stopping

`Follow.getTarget()` returns who is being followed, or `null` while the follow is only queued.
`Follow.isPending()` tells whether one is queued. `Follow.stop()` ends it.

## Why a follow ended

`followStopped(target, reason)` fires with one of these reasons:

| Reason                            | What happened                                                       |
| --------------------------------- | ------------------------------------------------------------------- |
| `script`                          | `Follow.stop()` was called, or a new follow started                 |
| `input`                           | the player pressed a movement key or the game's "Stop following" key |
| `mountChanged`                    | the player got on or off a horse                                    |
| `teleported`                      | the player moved more than 5 m at once                              |
| `worldChanged`                    | the player changed dimension                                        |
| `unavailable`                     | the target died, left this game, or is in another dimension         |
| `targetLost`                      | the target left or went out of range                                |
| `interrupted`                     | the game ended the follow itself                                    |
| `failed`                          | the game's follow did not start within 3 seconds                    |
| `resourceStopped`, `sessionEnded` | your resource stopped, or the player left the server                |

Reference: [Follow](/docs/api/follow).
