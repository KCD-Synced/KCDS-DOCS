# Nameplates

Each player has a plate above their head on other players' screens, with an optional title, the name, and
a health bar. The server decides what a plate says for everyone, and each player's client script decides
what its own screen draws.

## Setting a plate from the server

```js
kcds.core.Events.on('playerReady', (player) => {
  player.setNameplateTitle('Town guard');
  player.setNameplateText(`[Guard] ${player.nickname}`);
  player.setNameplateColor(0xffc9a227);
});
```

```ts
kcds.core.Events.on('playerReady', (player) => {
  player.setNameplateTitle('Town guard');
  player.setNameplateText(`[Guard] ${player.nickname}`);
  player.setNameplateColor(0xffc9a227);
});
```

```lua
kcds.core.Events.on("playerReady", function(player)
  player:setNameplateTitle("Town guard")
  player:setNameplateText("[Guard] " .. player.nickname)
  player:setNameplateColor(0xffc9a227)
end)
```

```cs
Events.On("playerReady", (Player player) =>
{
    player.SetNameplateTitle("Town guard");
    player.SetNameplateText($"[Guard] {player.Nickname}");
    player.SetNameplateColor(0xffc9a227);
});
```

| Call                                      | Effect                                                                                  |
| ----------------------------------------- | --------------------------------------------------------------------------------------- |
| `setNameplateVisible(visible)`            | shows or hides the whole plate                                                          |
| `setNameplateMode(mode)`                  | `'both'` (default), `'name'`, `'health'` or `'none'`                                    |
| `setNameplateText(text)`                  | replaces the nickname; `''` restores it                                                 |
| `setNameplateTitle(title, color?, size?)` | a line above the name; `''` removes it                                                  |
| `setNameplateColor(color)`                | the name's colour; 0 or `0xffffffff` restores the default cream                         |

**Colours** are written `0xAARRGGBB`. A title colour of 0 is the default gold.

**Title size** is in pixels on a 1080-high screen, from 8 to 48, and 0 uses the default.

**Mode** does the same as `setNameplateNameVisible` and `setNameplateHealthVisible` together. With the
health bar alone, the bar takes the name's place.

**Text** is cut to 64 bytes of UTF-8 and loses line breaks and control characters.

Each setter has a getter on both sides: `isNameplateVisible`, `isNameplateHealthVisible`,
`isNameplateNameVisible`, `getNameplateText`, `getNameplateTitle` and `getNameplateColor`. The server also
has `getNameplateMode`, `getNameplateTitleColor` and `getNameplateTitleSize`. A plate set before a player is
in the world appears once they are, and late joiners see it.

NPCs can have the same plate, using `npc.nameplate = true` and the same setters; see [NPCs](npcs.md).

## Overhead text

`player.showOverheadText(text, durationMs?, color?, range?)` shows a temporary line above the plate. It
appears for players near them in the same dimension, by default within 20 m, and for the player too.
Role-play servers use it for `/me` and `/do`. It returns how many players received it, and a player who
comes near later does not see it.

```js
kcds.core.Events.on('playerCommand', (player, command, args) => {
  if (command === 'me') {
    player.showOverheadText(`* ${args.join(' ')}`, 8000, 0xffc9a227);
  }
});
```

```ts
kcds.core.Events.on('playerCommand', (player, command, args) => {
  if (command === 'me') {
    player.showOverheadText(`* ${args.join(' ')}`, 8000, 0xffc9a227);
  }
});
```

```lua
kcds.core.Events.on("playerCommand", function(player, command, args)
  if command == "me" then
    player:showOverheadText("* " .. table.concat(args, " "), 8000, 0xffc9a227)
  end
end)
```

```cs
Events.On("playerCommand", (Player player, string command, string[] args) =>
{
    if (command == "me")
    {
        player.ShowOverheadText($"* {string.Join(" ", args)}", 8000, 0xffc9a227);
    }
});
```

## How a plate looks

- **Style:** a cream name in the game's menu font, a gold title above it, and a health bar in a gold frame
  below. Under a quarter of health, the bar turns a brighter red and pulses.
- **Speaking:** a horn icon appears left of the name while the player talks. See `player.setSpeaking` in
  [Animations](animations.md).
- **Distance:** full size up to 10 m, shrinking to 56 % at 30 m. Beyond 30 m, only a faded name shows at
  45 %, and beyond 60 m nothing does.
- **Hidden:** behind walls or terrain, and during loading screens.
- **Position:** above the middle of the body, just over the head. It follows the body rather than the head,
  so leaning or hammering at an anvil does not shake it. It rises immediately when the head goes higher,
  and comes down only when the player sits, crouches or lies down.
- **Close up:** when there is no room above the head, below the top of the screen and the compass, the plate
  moves down onto the top of the head.

## In a crowd

To keep a full server smooth and readable, the 32 nearest plates in view use the full design, and those
further away use a simpler drawing. Only about the nearest 48 bodies in view get a plate at all. A plate
with a label or speaking icon is always drawn.

## Choosing what a screen draws

These client calls affect only this player's screen.

```js
// client
const { Nameplates, Key } = kcds.ui;

Nameplates.setMode('name'); // names without health bars
Nameplates.setSelfVisible(true); // also this player's own plate, off by default

Key.bind('f8', () => {
  Nameplates.setVisible(!Nameplates.isVisible());
});
```

```ts
// client
const { Nameplates, Key } = kcds.ui;

Nameplates.setMode('name'); // names without health bars
Nameplates.setSelfVisible(true); // also this player's own plate, off by default

Key.bind('f8', () => {
  Nameplates.setVisible(!Nameplates.isVisible());
});
```

```lua
-- client
local Nameplates, Key = kcds.ui.Nameplates, kcds.ui.Key

Nameplates.setMode("name") -- names without health bars
Nameplates.setSelfVisible(true) -- also this player's own plate, off by default

Key.bind("f8", function()
  Nameplates.setVisible(not Nameplates.isVisible())
end)
```

```cs
// client
Nameplates.SetMode("name"); // names without health bars
Nameplates.SetSelfVisible(true); // also this player's own plate, off by default

Key.Bind("f8", new Action(() =>
{
    Nameplates.SetVisible(!Nameplates.IsVisible());
}));
```

A plate shows only what both sides allow: the server's mode for that player and this screen's mode.

### Labels

A client script can add a short line above any plate on its screen:

```js
// client
kcds.core.Events.on('trade:offer', (offer) => {
  kcds.ui.Nameplates.setLabel(offer.playerId, 'Wants to trade', 6000, 0xffffff00);
});
```

```ts
// client
kcds.core.Events.on('trade:offer', (offer) => {
  kcds.ui.Nameplates.setLabel(offer.playerId, 'Wants to trade', 6000, 0xffffff00);
});
```

```lua
-- client
kcds.core.Events.on("trade:offer", function(offer)
  kcds.ui.Nameplates.setLabel(offer.playerId, "Wants to trade", 6000, 0xffffff00)
end)
```

```cs
// client
Events.On("trade:offer", (JsValue offer) =>
{
    Nameplates.SetLabel(offer["playerId"], "Wants to trade", 6000, 0xffffff00);
});
```

In `setLabel(entityId, text, durationMs?, color?)`, the text can be up to 256 bytes. `durationMs` defaults
to 6000, and 0 or less keeps the label until it is cleared. A colour of 0 uses the plate's colour.
`clearLabel(entityId)` and `clearLabels()` remove labels, and a stopped resource's labels go too.

On a client, `Nameplates.setNameVisible(visible)` and `Nameplates.setHealthVisible(visible)` show or hide names and
health bars on this screen, `isNameVisible()`, `isHealthVisible()` and `getMode()` (`'both'`, `'name'`, `'health'` or
`'none'`) read it, and `isSelfVisible()` says whether the local player's own plate shows.

Reference: [Player](/docs/api/player), [Npc](/docs/api/npc), [Nameplates](/docs/api/nameplates).
