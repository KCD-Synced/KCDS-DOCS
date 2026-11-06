# Pictures from any camera

`kcds.world.Render.view` takes a picture from a camera you choose, without the player seeing that view. The
promise gives a PNG data URL to show in a web view or send to your server.

```js
// client
const { Vector3 } = kcds.math;

const shot = await kcds.world.Render.view({
  position: new Vector3(1652.5, 2151.5, 37.5),
  lookAt: new Vector3(1647.5, 2155.5, 36),
  fov: 60, // degrees; leave out or use 0 for the game's value
  width: 960, // PNG width; leave out for the window's size
});
// shot is { png: 'data:image/png;base64,iVBORw0...', width: 960, height: 530 }
```

```ts
// client
const { Vector3 } = kcds.math;

const shot = await kcds.world.Render.view({
  position: new Vector3(1652.5, 2151.5, 37.5),
  lookAt: new Vector3(1647.5, 2155.5, 36),
  fov: 60, // degrees; leave out or use 0 for the game's value
  width: 960, // PNG width; leave out for the window's size
});
// shot is { png: 'data:image/png;base64,iVBORw0...', width: 960, height: 530 }
```

```lua
-- client
local Vector3 = kcds.math.Vector3

-- :await() waits at the top of a file; in a handler, use :next(function(shot) ... end).
local shot = kcds.world.Render.view({
  position = Vector3.new(1652.5, 2151.5, 37.5),
  lookAt = Vector3.new(1647.5, 2155.5, 36),
  fov = 60, -- degrees; leave out or use 0 for the game's value
  width = 960, -- PNG width; leave out for the window's size
}):await()
-- shot is { png = "data:image/png;base64,iVBORw0...", width = 960, height = 530 }
```

```cs
// client
var shot = await Render.View(new
{
    position = new Vector3(1652.5, 2151.5, 37.5),
    lookAt = new Vector3(1647.5, 2155.5, 36),
    fov = 60, // degrees; leave out or use 0 for the game's value
    width = 960, // PNG width; leave out for the window's size
});
// shot is { png: "data:image/png;base64,iVBORw0...", width: 960, height: 530 }
Log.Info($"{shot["width"]} x {shot["height"]}");
```

**Pose:** give it as for `Camera.startShot`: `position`, then either `rotation` (a quaternion) or `lookAt` with an
optional `roll`. A pose `startShot` refuses throws the same error here.

**Contents:** the picture is the frame as the game renders it, with the HUD as the player has it. Hide elements
with `Hud.setHidden`. KCDS's chat and nameplates are not included.

**While it is taken:** this takes about a tenth of a second, during which the player keeps seeing the previous
frame, so the game seems to pause briefly. The game's eye adaptation follows the other view, so a much brighter or
darker view can briefly change the player's brightness.

**Timing and failure:** one picture is taken at a time, and others wait. The promise rejects with `Render.view
failed: <reason>` when the world changed, the view could not be made, or no frame arrived in 15 seconds. A minimised
window draws no frames.

A picture of a character alone on a transparent background is not available yet.

Reference: [Render](/docs/api/render).
