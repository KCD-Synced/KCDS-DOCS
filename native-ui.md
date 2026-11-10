# The game's own interface

`kcds.ui.NativeUI` works with the game's Scaleform (Flash) interface from a client script. You can
build a screen in the game's visual style, or control the interface elements the game already has.
For pages written in HTML, see [Web views](web-views.md).

## Building a screen

`NativeUI.createScreen()` gives you a game movie with an empty root clip. Wait for `'ready'` before
adding anything to it.

```js
// client
const board = kcds.ui.NativeUI.createScreen();

board.on('ready', () => {
  const panel = board.root.createClip();
  panel.draw([
    ['beginFill', 0x15130f, 80],
    ['moveTo', 0, 0],
    ['lineTo', 400, 0],
    ['lineTo', 400, 200],
    ['lineTo', 0, 200],
    ['endFill'],
  ]);

  const heading = panel.createText({ x: 20, y: 20, width: 360, height: 40, text: 'Town board' });
  heading.setTextStyle({ font: 'DisplayFont', size: 24 });

  const closeLabel = panel.createText({ x: 20, y: 150, width: 120, height: 30, text: 'Close' });
  closeLabel.on('press', () => board.close());

  board.setVisible(true);
  board.focus();
});

board.on('input', (action, phase) => {
  if (action === 'kcds_ui_back' && phase === 'press') {
    board.close();
  }
});
```

```ts
// client
const board = kcds.ui.NativeUI.createScreen();

board.on('ready', () => {
  const panel = board.root.createClip();
  if (!panel) {
    return;
  }
  panel.draw([
    ['beginFill', 0x15130f, 80],
    ['moveTo', 0, 0],
    ['lineTo', 400, 0],
    ['lineTo', 400, 200],
    ['lineTo', 0, 200],
    ['endFill'],
  ]);

  const heading = panel.createText({ x: 20, y: 20, width: 360, height: 40, text: 'Town board' });
  heading?.setTextStyle({ font: 'DisplayFont', size: 24 });

  const closeLabel = panel.createText({ x: 20, y: 150, width: 120, height: 30, text: 'Close' });
  closeLabel?.on('press', () => board.close());

  board.setVisible(true);
  board.focus();
});

board.on('input', (action, phase) => {
  if (action === 'kcds_ui_back' && phase === 'press') {
    board.close();
  }
});
```

```lua
-- client
local board = kcds.ui.NativeUI.createScreen()

board:on("ready", function()
  local panel = board.root:createClip()
  panel:draw({
    { "beginFill", 0x15130f, 80 },
    { "moveTo", 0, 0 },
    { "lineTo", 400, 0 },
    { "lineTo", 400, 200 },
    { "lineTo", 0, 200 },
    { "endFill" },
  })

  local heading = panel:createText({ x = 20, y = 20, width = 360, height = 40, text = "Town board" })
  heading:setTextStyle({ font = "DisplayFont", size = 24 })

  local closeLabel = panel:createText({ x = 20, y = 150, width = 120, height = 30, text = "Close" })
  closeLabel:on("press", function() board:close() end)

  board:setVisible(true)
  board:focus()
end)

board:on("input", function(action, phase)
  if action == "kcds_ui_back" and phase == "press" then
    board:close()
  end
end)
```

```cs
// client
var board = NativeUI.CreateScreen();

board.On("ready", () =>
{
    var panel = board.Root.CreateClip()!;
    panel.Draw(new[]
    {
        new object[] { "beginFill", 0x15130f, 80 },
        new object[] { "moveTo", 0, 0 },
        new object[] { "lineTo", 400, 0 },
        new object[] { "lineTo", 400, 200 },
        new object[] { "lineTo", 0, 200 },
        new object[] { "endFill" },
    });

    var heading = panel.CreateText(new { x = 20, y = 20, width = 360, height = 40, text = "Town board" })!;
    heading.SetTextStyle(new { font = "DisplayFont", size = 24 });

    var closeLabel = panel.CreateText(new { x = 20, y = 150, width = 120, height = 30, text = "Close" })!;
    closeLabel.On("press", () => board.Close());

    board.SetVisible(true);
    board.Focus();
});

board.On("input", (string action, string phase) =>
{
    if (action == "kcds_ui_back" && phase == "press")
    {
        board.Close();
    }
});
```

**Clips** offer these calls ([NativeClip](/docs/api/nativeclip)):

| Call                                  | Purpose                                                       |
| ------------------------------------- | ------------------------------------------------------------- |
| `createClip`, `createText`            | add a child clip or text                                      |
| `attach`                              | add a symbol exported by a game movie (see `NativeUI.libraries()`) |
| `draw`                                | draw shapes                                                   |
| `set({ x, y, alpha, ... })`           | change properties                                             |
| `load`                                | load a picture                                                |
| `on('press' \| 'rollOver' \| ...)`    | react to the mouse                                            |

**Fonts** from the game include `DefaultFont`, `DefaultFontBold`, `LightFont`, `Manuscript` and
`DisplayFont`.

**Input:** while a screen has focus, key binds are paused, and menu keys reach the screen as `'input'`
events such as `kcds_ui_accept`, `kcds_ui_back` and `kcds_ui_left`.

**Your own movie:** `createScreen({ movie })` loads a `.swf` shipped with your resource. See
[NativeScreen](/docs/api/nativescreen).

## Controlling existing elements

`NativeUI.element(name)` returns one of the game's interface elements:

```js
// client
const hud = kcds.ui.NativeUI.element('HUD');
hud.call('ShowInfoText', ['Hello', 0, 3000, false]);
```

```ts
// client
const hud = kcds.ui.NativeUI.element('HUD');
hud.call('ShowInfoText', ['Hello', 0, 3000, false]);
```

```lua
-- client
local hud = kcds.ui.NativeUI.element("HUD")
hud:call("ShowInfoText", { "Hello", 0, 3000, false })
```

```cs
// client
var hud = NativeUI.Element("HUD");
hud.Call("ShowInfoText", new object[] { "Hello", 0, 3000, false });
```

An element ([NativeElement](/docs/api/nativeelement)) has `call`, `setVariable`, `getVariable`,
`setClip`, `gotoClip`, `load`, `show`, `hide` and its own events. `call` runs one of the movie's
ActionScript functions by its name; it does not take a clip's path, so `call('row.icon.loadMovie', ...)`
does nothing. To put your own picture into a clip of a game screen, such as a stat row's icon, use
`load(clip, picture)` with the clip's path below `_root`, written with dots
(`'main.stats.soulProperties.soulProperty12.icon'`), and a `.dds` your resource ships, or an `img://` address.
A shipped `.dds` is listed in the manifest's `files` and named by its path in the resource
(`'icons/thirst.dds'`); KCDS sends it and mounts it for the game, as `Book.image` does. `load` returns `false`
when the element has no such clip.

The game fills its screens each time they open, so load your picture after the screen opens:
`nativeScreenOpened` names the screen. [HUD and messages](client-ui.md#your-own-icon-on-a-stat-row) shows it for
the inventory's stat rows.

### The game's screens

`nativeScreenOpened` and `nativeScreenClosed` fire with the name of the screen's element, the frame the game
shows or hides it, and `NativeUI.openScreens()` lists the open ones. A menu of several panels opens several:

| Screen                    | Elements                                                     |
| ------------------------- | ------------------------------------------------------------ |
| Inventory                 | `ApseInventoryList`, `ApseInventoryInfo`, `ApseCharacter` (the character panel and its stats) |
| Player                    | `ApsePlayerList`, `ApsePlayerInfo`                           |
| Map                       | `ApseMap`, `ApseMapLegendList`                               |
| Journal                   | `ApseQuestLogList`, `ApseQuestLogDiary`                      |
| Codex                     | `ApseCodexList`                                              |
| Crafting                  | `ApseCraftingList`, `ApseCraftingContent`                    |
| Chests and other containers | `ItemTransfer`                                             |
| Horse                     | `HorseInspect`                                               |
| Lockpicking, pickpocketing | `LockPicking`, `Pickpocketing`                              |
| Waiting and sleeping      | `SkipTime`                                                   |
| Books, recipes, plans     | `GeneralBook`, `AlchemyBook`, `ForgeBuilderPlan`             |
| Questions and messages    | `ApseModalDialog`                                            |
| Other                     | `ItemSelection`                                              |

## Closing menu pages

`setMenuEnabled` keeps a page of the game's menu from opening. The pages are `'crafting'`,
`'inventory'`, `'player'`, `'map'`, `'codex'` and `'journal'`:

```js
// client
kcds.ui.NativeUI.setMenuEnabled('map', false);
```

```ts
// client
kcds.ui.NativeUI.setMenuEnabled('map', false);
```

```lua
-- client
kcds.ui.NativeUI.setMenuEnabled("map", false)
```

```cs
// client
NativeUI.SetMenuEnabled("map", false);
```

## Adding a button to the Esc menu

Your button appears in the game's style above "Report a bug", each time the player opens the menu:

```js
// client
const settingsButton = kcds.ui.NativeUI.addMenuButton('Server settings', () => {
  kcds.core.Events.toServer('settings:open');
});

// When you no longer need it:
kcds.ui.NativeUI.removeMenuButton(settingsButton);
```

```ts
// client
const settingsButton = kcds.ui.NativeUI.addMenuButton('Server settings', () => {
  kcds.core.Events.toServer('settings:open');
});

// When you no longer need it:
if (settingsButton !== null) {
  kcds.ui.NativeUI.removeMenuButton(settingsButton);
}
```

```lua
-- client
local settingsButton = kcds.ui.NativeUI.addMenuButton("Server settings", function()
  kcds.core.Events.toServer("settings:open")
end)

-- When you no longer need it:
kcds.ui.NativeUI.removeMenuButton(settingsButton)
```

```cs
// client
var settingsButton = NativeUI.AddMenuButton("Server settings", () =>
{
    Events.ToServer("settings:open");
});

// When you no longer need it:
NativeUI.RemoveMenuButton(settingsButton);
```

- `addMenuButton(label, handler)` returns the button's id. Labels can be up to 64 characters.
- The menu has two free places, where the game's save buttons normally are. On a server those are not
  shown. With both taken, the call returns `null`.
- `removeMenuButton(id)` removes only your resource's own buttons and returns `false` for any other
  id. A resource that stops loses its buttons.
- Do not call the menu's Flash function `AddBasicButton` through `NativeUI.element('Menu')`. The game
  rebuilds the menu each time it opens, and such a button does not receive clicks.

## More members

- `NativeUI.isMenuEnabled(menu)` says whether a menu page can be opened.
- A screen: `hasFocus()`, `screenToStage(x, y)` (a window position in stage coordinates) and `watchClips()` (clip mouse
  events; `NativeClip.on` starts them for you).
- An element: `instanceId`, `setArray(array, values)` and `getArray(array)` for ActionScript arrays, `getClip(clip)` (a
  clip's display fields), `askHide()` (it hides itself the game's way) and `unload()`.
- A clip: `setMember(member, value)` and `invoke(method, args)` for ActionScript, `setText(text, html)` and
  `getText(html)`, `goto(frameOrLabel, play)`, `setMask(mask)`, `setHitArea(area)`, `setLeading(pixels)` (-64 to 256
  between lines), `setTabStops(stops)` (up to 15, for columns in one field) and `loadShipped(path)` (a file your
  resource ships).

Reference: [NativeUI](/docs/api/nativeui), [NativeScreen](/docs/api/nativescreen),
[NativeElement](/docs/api/nativeelement), [NativeClip](/docs/api/nativeclip).
