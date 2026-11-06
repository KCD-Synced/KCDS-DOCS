# Camera

The camera on the player's own machine: flying it, paths through the world and shaking the view. Related pages:
[What the player looks at](looking-at.md), [Pictures from any camera](pictures.md),
[Character creator](character-creator.md) and [Local NPCs and the local player](local-figures.md).

## Free camera

`kcds.ui.NoClip` flies the camera through walls. In `'body'` mode the player's body goes too, so others see it
fly, and it lands where the flight ends. In `'camera'` mode only the view moves.

```js
// client
const { NoClip, Key } = kcds.ui;

Key.bind('f3', () => {
  if (NoClip.isActive()) {
    NoClip.disable();
  } else {
    NoClip.enable({ mode: 'body', speed: 20 });
  }
});
```

```lua
-- client
local NoClip, Key = kcds.ui.NoClip, kcds.ui.Key

Key.bind("f3", function()
  if NoClip.isActive() then
    NoClip.disable()
  else
    NoClip.enable({ mode = "body", speed = 20 })
  end
end)
```

```cs
// client
Key.Bind("f3", new Action(() =>
{
    if (NoClip.IsActive())
    {
        NoClip.Disable();
    }
    else
    {
        NoClip.Enable(new { mode = "body", speed = 20 });
    }
}));
```

**Keys:** W A S D move, Space goes up, C goes down, Shift is fast and Alt is slow.

While the flight turns with the mouse, the mouse pointer is hidden, even when a page of yours has the focus (for
example a page that catches keys during the flight). The page still gets the keys. With `look: 'rightButton'`, or
`input: false`, the pointer shows as usual.

**For cutscenes and spectators,** use `input: false` and steer it yourself with `NoClip.move(...)`, `setPose`,
`lookAt` and `focus(target, distance)`. `NoClip.toScreen(point)` and `screenLine(x, y)` help pick things on
screen. `noclipChanged(active, reason)` tells you when it ends.

While flying, `NoClip.getState()` gives the whole flight (`mode`, `position`, `forward`, `yaw`, `pitch`, `speed`,
`fov` and more), and `getPosition`/`setPosition`, `getRotation`/`setRotation`, `getForward`, `getSpeed`/`setSpeed`
(0.05 to 400 m/s), `getFov`/`setFov` (20 to 140 degrees) and `isInputEnabled`/`setInputEnabled` (leave steering to
`move()`) read and change its parts.

### Who may fly

Everyone may fly by default, and the server decides otherwise. Setting `player.noclipAllowed = false` ends that
player's flight with `noclipChanged(false, 'notAllowed')`. After that, `NoClip.enable()` returns `'notAllowed'`
and does nothing. The refusal comes from their game, so a client script cannot get around it.

To keep flying for staff, set it when a player joins:

```js
// server
kcds.core.Events.on('playerConnect', (player) => {
  player.noclipAllowed = staff.has(player.steamId);
});
```

```lua
-- server
kcds.core.Events.on("playerConnect", function(player)
  player.noclipAllowed = staff[player.steamId] == true
end)
```

```cs
// server
Events.On("playerConnect", (Player player) =>
{
    player.NoclipAllowed = staff.Contains(player.SteamId.AsString()!);
});
```

Client scripts can check `NoClip.isAllowed()` and the `noclipAllowedChanged(allowed)` event, or test the result:

```js
// client
const { NoClip, Hud, Key } = kcds.ui;

Key.bind('f3', () => {
  if (NoClip.isActive()) {
    NoClip.disable();
  } else if (NoClip.enable() === 'notAllowed') {
    Hud.showNotification('Flying is not allowed here.');
  }
});
```

```lua
-- client
local NoClip, Hud, Key = kcds.ui.NoClip, kcds.ui.Hud, kcds.ui.Key

Key.bind("f3", function()
  if NoClip.isActive() then
    NoClip.disable()
  elseif NoClip.enable() == "notAllowed" then
    Hud.showNotification("Flying is not allowed here.")
  end
end)
```

```cs
// client
Key.Bind("f3", new Action(() =>
{
    if (NoClip.IsActive())
    {
        NoClip.Disable();
    }
    else if (NoClip.Enable() == "notAllowed")
    {
        Hud.ShowNotification("Flying is not allowed here.");
    }
}));
```

For a shot the server controls, see `Camera.startShot` in [Server rules](server-rules.md).

## Camera paths

`Camera.playPath(keys, { smooth })` flies this machine's view through up to 64 keys. The game works out the pose every
frame, so the flight has none of the steps a script timer leaves. Each key is a pose as for `startShot`, `position` and
`rotation` or `lookAt`, with `fov` and `time`, its seconds from now. With `smooth`, the default, the view curves through
the keys and starts and ends slowly. `smooth: false` runs straight from key to key at one speed. A first key after 0
seconds starts from where the camera is now.

```js
// client
const { Camera } = kcds.ui;
const { Vector3 } = kcds.math;

const square = new Vector3(512, 830, 31);
Camera.playPath([
  { position: new Vector3(480, 800, 45), lookAt: square, time: 2 },
  { position: new Vector3(540, 800, 40), lookAt: square, time: 8 },
  { position: new Vector3(545, 860, 36), lookAt: square, time: 14, fov: 45 },
]);
kcds.core.Events.on('cameraPathFinished', () => Camera.endShot());
```

```lua
-- client
local Camera = kcds.ui.Camera
local Vector3 = kcds.math.Vector3

local square = Vector3.new(512, 830, 31)
Camera.playPath({
  { position = Vector3.new(480, 800, 45), lookAt = square, time = 2 },
  { position = Vector3.new(540, 800, 40), lookAt = square, time = 8 },
  { position = Vector3.new(545, 860, 36), lookAt = square, time = 14, fov = 45 },
})
kcds.core.Events.on("cameraPathFinished", function()
  Camera.endShot()
end)
```

```cs
// client
var square = new Vector3(512, 830, 31);
Camera.PlayPath(new object[]
{
    new { position = new Vector3(480, 800, 45), lookAt = square, time = 2 },
    new { position = new Vector3(540, 800, 40), lookAt = square, time = 8 },
    new { position = new Vector3(545, 860, 36), lookAt = square, time = 14, fov = 45 },
});
// In the player's game EndShot takes no player (C# names the server's parameter).
Events.On("cameraPathFinished", () => Camera.EndShot(null));
```

`cameraPathFinished` fires at the last key, and the view holds there until `Camera.endShot()`. A new path or shot
replaces the one playing.

`Camera.setThirdPerson(options)` shows this machine's view through the game's third-person camera until `null`, and
`Camera.isThirdPerson()` reads it; `Camera.inShot(player)` says whether a player is in a shot.

## Shaking the view

`Camera.shake({ angle, shift, duration, frequency, randomness })` uses the game's own view shake, as a blow or a fall
does, and the view settles back by itself. `angle` is in degrees, up to 90 either way: `x` pitches the view, positive
down, `y` rolls it and `z` turns it. `frequency` 0, the default, gives one push, and higher values shake that many
times a second.

```js
// client
kcds.core.Events.on('kicked', () => {
  kcds.ui.Camera.shake({ angle: { x: -15 }, duration: 1 });
});
```

```lua
-- client
kcds.core.Events.on("kicked", function()
  kcds.ui.Camera.shake({ angle = { x = -15 }, duration = 1 })
end)
```

```cs
// client
Events.On("kicked", () =>
{
    Camera.Shake(new { angle = new { x = -15 }, duration = 1 });
});
```

Reference: [NoClip](/docs/api/noclip), [Camera](/docs/api/camera).
