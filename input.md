# Keys and controls

Client scripts can react to keys, offer rebindable actions in the game's settings, and temporarily
take away the player's controls.

## Binding a key

```js
// client
const { Key } = kcds.ui;

Key.bind('f', () => kcds.core.Events.toServer('openDoor'));
Key.bind('shift', 'both', (key, edge) => console.log(`${key} ${edge}`)); // edge is 'down' or 'up'
Key.unbind('f');
```

```lua
-- client
local Key = kcds.ui.Key

Key.bind("f", function() kcds.core.Events.toServer("openDoor") end)
Key.bind("shift", "both", function(key, edge) print(key .. " " .. edge) end) -- edge is "down" or "up"
Key.unbind("f")
```

```cs
// client
Key.Bind("f", new Action(() => Events.ToServer("openDoor")));
Key.Bind("shift", "both", (string key, string edge) => Log.Info($"{key} {edge}")); // edge is "down" or "up"
Key.Unbind("f");
```

`Key.getLabel(key)` gives the text printed on that key for the player's keyboard, and `Key.isDown(key)`
says whether it is held.

Binds fire only in a world, with the game window active and with no chat or web view taking typed text.
While one of your NativeUI screens has focus, binds pause and keys go to that screen.

### Key names

Keys are named by their position on a US keyboard, whatever layout the player uses.

| Group        | Names                                                                                     |
| ------------ | ----------------------------------------------------------------------------------------- |
| Letters      | `a` to `z`                                                                                |
| Numbers      | `0` to `9`, `numpad0` to `numpad9`                                                        |
| Arrows       | `up`, `down`, `left`, `right`                                                             |
| Modifiers    | `shift`, `ctrl`, `alt`, and side versions such as `lshift` and `rctrl`                    |
| Editing      | `space`, `enter`, `escape`, `tab`, `backspace`, `insert`, `delete`, `capslock`            |
| Navigation   | `home`, `end`, `pageup`, `pagedown`                                                       |
| Mouse        | `mouse1` to `mouse5`                                                                      |
| Punctuation  | `grave`, `minus`, `equals`, `lbracket`, `rbracket`, `backslash`, `semicolon`, `apostrophe`, `comma`, `period`, `slash` |

### Controller buttons

Any connected Xbox-style controller works too, using the game's button names:

| Buttons            | Names                                                           |
| ------------------ | --------------------------------------------------------------- |
| Face               | `xi_a`, `xi_b`, `xi_x`, `xi_y`                                  |
| D-pad              | `xi_dpad_up`, `xi_dpad_down`, `xi_dpad_left`, `xi_dpad_right`   |
| LB, RB             | `xi_shoulderl`, `xi_shoulderr`                                  |
| LT, RT             | `xi_triggerl_btn`, `xi_triggerr_btn`                            |
| Sticks pressed     | `xi_thumbl`, `xi_thumbr`                                        |
| Menu               | `xi_start`, `xi_back`                                           |

## Actions the player can rebind

`Key.bind` always uses the key you name. To let the player choose, add an action with
`kcds.ui.Keybinds`. It appears in the game's **Settings > Key bindings**, and the player changes it like
any other action. The game prevents it from sharing a key with a game action or another resource's
action.

```js
// client
const { Keybinds } = kcds.ui;
const { Events } = kcds.core;

Keybinds.addCategory('roleplay', 'Roleplay');
Keybinds.add('emote_menu', {
  label: 'Emote menu',
  category: 'roleplay',
  key: 'f6',
  controller: 'xi_dpad_up',
});
Keybinds.add('radio', { label: 'Talk on the radio', key: 'numpad1' });

Keybinds.on('emote_menu', () => Events.toServer('emotes:open'));
Keybinds.on('radio', 'both', (id, edge) => {
  Events.toServer('radio:talk', edge === 'down');
});
```

```lua
-- client
local Keybinds = kcds.ui.Keybinds
local Events = kcds.core.Events

Keybinds.addCategory("roleplay", "Roleplay")
Keybinds.add("emote_menu", {
  label = "Emote menu",
  category = "roleplay",
  key = "f6",
  controller = "xi_dpad_up",
})
Keybinds.add("radio", { label = "Talk on the radio", key = "numpad1" })

Keybinds.on("emote_menu", function() Events.toServer("emotes:open") end)
Keybinds.on("radio", "both", function(id, edge)
  Events.toServer("radio:talk", edge == "down")
end)
```

```cs
// client
Keybinds.AddCategory("roleplay", "Roleplay");
Keybinds.Add("emote_menu", new
{
    label = "Emote menu",
    category = "roleplay",
    key = "f6",
    controller = "xi_dpad_up",
});
Keybinds.Add("radio", new { label = "Talk on the radio", key = "numpad1" });

Keybinds.On("emote_menu", new Action(() => Events.ToServer("emotes:open")));
Keybinds.On("radio", "both", (string id, string edge) =>
{
    Events.ToServer("radio:talk", edge == "down");
});
```

The game's own categories are `general` (the default), `movement`, `combat`, `chat`, `qam`, `gui`,
`photomode` and `other`.

Handlers follow whatever key the player picks. To display the current key in a hint, use
`Keybinds.get(id)`:

```js
// client
const menu = kcds.ui.Keybinds.get('emote_menu');
// { id: 'emote_menu', label: 'Emote menu', category: 'roleplay', key: 'f6', keys: ['f6'],
//   controller: 'xi_dpad_up', defaultKey: 'f6', defaultController: 'xi_dpad_up' }
```

```lua
-- client
local menu = kcds.ui.Keybinds.get("emote_menu")
-- { id = "emote_menu", label = "Emote menu", category = "roleplay", key = "f6", keys = { "f6" },
--   controller = "xi_dpad_up", defaultKey = "f6", defaultController = "xi_dpad_up" }
```

```cs
// client
var menu = Keybinds.Get("emote_menu");
// { id: "emote_menu", label: "Emote menu", category: "roleplay", key: "f6", keys: ["f6"],
//   controller: "xi_dpad_up", defaultKey: "f6", defaultController: "xi_dpad_up" }
Log.Info(menu["key"].ToString());
```

### Where choices are stored

Choices are saved on the player's computer separately for each server, and return when they rejoin.
Other servers' resources cannot read or change them. They stay out of the game's profile, so the game
played without KCDS is not affected.

### Controller buttons and changes from a script

The settings screen lists only keyboard and mouse keys. A controller button is the default you give it,
or what your script sets, perhaps from your own settings page:

```js
// client
const { Keybinds } = kcds.ui;

Keybinds.set('radio', { key: 'f7', controller: 'xi_y' }); // saved like a choice on the screen
Keybinds.set('emote_menu', { controller: null }); // no controller button
Keybinds.reset('radio'); // back to the defaults
```

```lua
-- client
local Keybinds = kcds.ui.Keybinds

Keybinds.set("radio", { key = "f7", controller = "xi_y" }) -- saved like a choice on the screen
Keybinds.set("emote_menu", { controller = null }) -- no controller button (null: nil would be left out)
Keybinds.reset("radio") -- back to the defaults
```

```cs
// client
Keybinds.Set("radio", new { key = "f7", controller = "xi_y" }); // saved like a choice on the screen
Keybinds.Set("emote_menu", new { controller = Sdk.Null }); // no controller button (a null property is left out)
Keybinds.Reset("radio"); // back to the defaults
```

### Removing actions

A stopping resource's actions leave the screen, and `Keybinds.remove(id)` removes one while it runs. The
screen also offers the mouse wheel and keys KCDS does not poll. Choosing one means the action never
fires.

## Taking away controls

```js
// client
const { Controls } = kcds.ui;

Controls.disable(); // no moving, fighting or using things
Controls.showCursor();
// ... your menu is open ...
Controls.hideCursor();
Controls.enable();
```

```lua
-- client
local Controls = kcds.ui.Controls

Controls.disable() -- no moving, fighting or using things
Controls.showCursor()
-- ... your menu is open ...
Controls.hideCursor()
Controls.enable()
```

```cs
// client
Controls.Disable(); // no moving, fighting or using things
Controls.ShowCursor();
// ... your menu is open ...
Controls.HideCursor();
Controls.Enable();
```

Each resource's hold is counted, so controls stay off while any resource holds them. A stopping
resource gives back what it held.

### Blocking one action

You can block a single game action, on whatever key the player uses for it. The key still reaches
`Key.bind`, which lets a resource take over a game key:

```js
// client
const { Controls, Key } = kcds.ui;

Controls.blockAction('jump'); // space no longer jumps
Key.bind('space', () => kcds.core.Events.toServer('ring:bell'));

// Later:
Controls.unblockAction('jump');
```

```lua
-- client
local Controls, Key = kcds.ui.Controls, kcds.ui.Key

Controls.blockAction("jump") -- space no longer jumps
Key.bind("space", function() kcds.core.Events.toServer("ring:bell") end)

-- Later:
Controls.unblockAction("jump")
```

```cs
// client
Controls.BlockAction("jump"); // space no longer jumps
Key.Bind("space", new Action(() => Events.ToServer("ring:bell")));

// Later:
Controls.UnblockAction("jump");
```

The names are the game's player actions, such as `jump`, `use`, `opponent_lock`,
`attack_primary_mouse`, `block`, `toggle_crouch` and `sprint`.

`Controls.isCursorVisible()` says whether the pointer shows (`cursorVisibilityChanged` when that changes), and while
your resource shows it, `Controls.cursorPosition()` is where it is, `{ x, y }` from -1 to 1, as
`Camera.rayFromScreen` takes it.

Reference: [Key](/docs/api/key), [Keybinds](/docs/api/keybinds), [Controls](/docs/api/controls).
