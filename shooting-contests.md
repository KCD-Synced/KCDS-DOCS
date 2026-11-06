# Shooting contests and races

The game's archery contest table, bottom left: the player's and up to two opponents' names, the arrows each has
left, their points, and a target picture that marks where an arrow struck. The server keeps the score and sends it
to the player's game, whose client script shows it:

```js
// server
kcds.core.Events.fromClient('archery:join', (player) => {
  player.emit('archery:start', { rival: 'Jindra', arrows: 5 });
});
```

```ts
// server
kcds.core.Events.fromClient('archery:join', (player) => {
  player.emit('archery:start', { rival: 'Jindra', arrows: 5 });
});
```

```lua
-- server
kcds.core.Events.fromClient("archery:join", function(player)
  player:emit("archery:start", { rival = "Jindra", arrows = 5 })
end)
```

```cs
// server
Events.FromClient("archery:join", (Player player) =>
{
    player.Emit("archery:start", new { rival = "Jindra", arrows = 5 });
});
```

```js
// client
const { Hud } = kcds.ui;

kcds.core.Events.on('archery:start', (contest) => {
  Hud.showShootingContest({
    player: { name: 'You', arrows: contest.arrows },
    opponents: [{ name: contest.rival, arrows: contest.arrows }],
  });
});
kcds.core.Events.on('archery:shot', (shot) => {
  Hud.showShootingHit(shot.x, shot.y);
  Hud.updateShootingContest({
    player: { arrows: shot.arrows, points: shot.points },
    opponents: [{ arrows: shot.rivalArrows, points: shot.rivalPoints }],
  });
});
kcds.core.Events.on('archery:end', () => Hud.hideShootingContest());
```

```ts
// client
const { Hud } = kcds.ui;

kcds.core.Events.on('archery:start', (contest) => {
  Hud.showShootingContest({
    player: { name: 'You', arrows: contest.arrows },
    opponents: [{ name: contest.rival, arrows: contest.arrows }],
  });
});
kcds.core.Events.on('archery:shot', (shot) => {
  Hud.showShootingHit(shot.x, shot.y);
  Hud.updateShootingContest({
    player: { arrows: shot.arrows, points: shot.points },
    opponents: [{ arrows: shot.rivalArrows, points: shot.rivalPoints }],
  });
});
kcds.core.Events.on('archery:end', () => Hud.hideShootingContest());
```

```lua
-- client
local Hud = kcds.ui.Hud

kcds.core.Events.on("archery:start", function(contest)
  Hud.showShootingContest({
    player = { name = "You", arrows = contest.arrows },
    opponents = { { name = contest.rival, arrows = contest.arrows } },
  })
end)
kcds.core.Events.on("archery:shot", function(shot)
  Hud.showShootingHit(shot.x, shot.y)
  Hud.updateShootingContest({
    player = { arrows = shot.arrows, points = shot.points },
    opponents = { { arrows = shot.rivalArrows, points = shot.rivalPoints } },
  })
end)
kcds.core.Events.on("archery:end", function()
  Hud.hideShootingContest()
end)
```

```cs
// client
Events.On("archery:start", (JsValue contest) =>
{
    Hud.ShowShootingContest(new
    {
        player = new { name = "You", arrows = contest["arrows"] },
        opponents = new object[] { new { name = contest["rival"], arrows = contest["arrows"] } },
    });
});
Events.On("archery:shot", (JsValue shot) =>
{
    Hud.ShowShootingHit(shot["x"], shot["y"]);
    Hud.UpdateShootingContest(new
    {
        player = new { arrows = shot["arrows"], points = shot["points"] },
        opponents = new object[] { new { arrows = shot["rivalArrows"], points = shot["rivalPoints"] } },
    });
});
Events.On("archery:end", () => Hud.HideShootingContest());
```

`showShootingHit(x, y)` takes the spot on the target from -1 to 1 across and up. Horse races have their own table:
`showHorseContest({ player, opponent, track })`, `updateHorseContest(score)` and `hideHorseContest()`.

Reference: [Hud](/docs/api/hud).
