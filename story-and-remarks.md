# Cutscenes and the character's remarks

Two things of the single-player story stay silent on KCDS, and each tells your client script when it would have
happened.

## The character's remarks

In the game, the player's character remarks on things, such as horse commands, an empty pot or photo mode, in Henry's
voice regardless of who plays. On KCDS these stay silent, and each fires `monologueSuppressed` with the line in the
player's language, so you can show it your own way:

```js
// client
kcds.core.Events.on('monologueSuppressed', (text, textKey) => {
  if (text) {
    kcds.ui.Hud.showNotification(`(you think) ${text}`);
  }
});
```

```ts
// client
kcds.core.Events.on('monologueSuppressed', (text, textKey) => {
  if (text) {
    kcds.ui.Hud.showNotification(`(you think) ${text}`);
  }
});
```

```lua
-- client
kcds.core.Events.on("monologueSuppressed", function(text, textKey)
  if text and text ~= "" then
    kcds.ui.Hud.showNotification("(you think) " .. text)
  end
end)
```

```cs
// client
Events.On("monologueSuppressed", (string? text, string? textKey) =>
{
    if (!string.IsNullOrEmpty(text))
    {
        Hud.ShowNotification($"(you think) {text}");
    }
});
```

`Hud.setInnerVoice(true)` lets the game voice and caption them on this player's screen, with no event. `Hud.innerVoiceOn()`
reads the setting.

## Story cutscenes

A quest plays story cutscenes, including fades and time skips, when the player walks into a trigger. In the game's
cutscene state the player loses control, the world clock can stop and the quest's scenery loads, so on KCDS they do
not play. The quest carries on as if they had, and each one fires `storyCutsceneSuppressed` with its name from the
game's cutscene table:

```js
// client
kcds.core.Events.on('storyCutsceneSuppressed', (cutscene) => {
  console.info(`skipped the cutscene ${cutscene}`);
});
```

```ts
// client
kcds.core.Events.on('storyCutsceneSuppressed', (cutscene) => {
  console.info(`skipped the cutscene ${cutscene}`);
});
```

```lua
-- client
kcds.core.Events.on("storyCutsceneSuppressed", function(cutscene)
  print("skipped the cutscene " .. cutscene)
end)
```

```cs
// client
Events.On("storyCutsceneSuppressed", (string cutscene) =>
{
    Log.Info($"skipped the cutscene {cutscene}");
});
```

`Hud.setCutscenes(true)` lets them play on this player's screen for the session, with no event. `Hud.cutscenesOn()`
reads the setting. A skipped cutscene never plays afterwards. The player's own fast travel, sleep and waiting are a
different kind of scene and always play.

Reference: [Hud](/docs/api/hud), [Events](/docs/api/events).
