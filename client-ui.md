# HUD and messages

Client scripts can draw on this player's screen using the game's own HUD. The rest of the game's screens have pages of
their own: ["Press [key] to ..." hints](action-hints.md), [Books and letters](books.md),
[Map and compass](world-map.md), [Shooting contests and races](shooting-contests.md) and
[Cutscenes and the character's remarks](story-and-remarks.md).

## Messages

```js
// client
const { Hud } = kcds.ui;

Hud.showNotification('The gates close at dusk.');
Hud.showBanner('Welcome to Kuttenberg', 5000);
Hud.showTutorialHint({
  title: 'Fishing',
  text: 'Stand at the water and cast.',
  durationMs: 10000,
});
```

```ts
// client
const { Hud } = kcds.ui;

Hud.showNotification('The gates close at dusk.');
Hud.showBanner('Welcome to Kuttenberg', 5000);
Hud.showTutorialHint({
  title: 'Fishing',
  text: 'Stand at the water and cast.',
  durationMs: 10000,
});
```

```lua
-- client
local Hud = kcds.ui.Hud

Hud.showNotification("The gates close at dusk.")
Hud.showBanner("Welcome to Kuttenberg", 5000)
Hud.showTutorialHint({
  title = "Fishing",
  text = "Stand at the water and cast.",
  durationMs = 10000,
})
```

```cs
// client
Hud.ShowNotification("The gates close at dusk.");
Hud.ShowBanner("Welcome to Kuttenberg", 5000);
Hud.ShowTutorialHint(new
{
    title = "Fishing",
    text = "Stand at the water and cast.",
    durationMs = 10000,
});
```

The HUD also has the game's perk, combo, reputation and XP notices, such as `notifyPerk` and `notifyXp`, and its
tutorials through `showTutorial(name)`. These calls return `false` when there is no HUD, such as in the main menu
or during a level load.

## Hiding parts of the HUD

```js
// client
kcds.ui.Hud.setHudPart('Compass', false);
```

```ts
// client
kcds.ui.Hud.setHudPart('Compass', false);
```

```lua
-- client
kcds.ui.Hud.setHudPart("Compass", false)
```

```cs
// client
Hud.SetHudPart("Compass", false);
```

Parts include `Compass`, `Stats`, `QAMWeapon`, `QAMFood`, `Subtitles`, `InfoText`, `GameLog`, `Hints`,
`DialogLeft`, `DialogRight`, `Cursor`, `Crime`, `Wanted`, `Buffs`, `Trespassing` and `Vignette`; see
[Hud](/docs/api/hud) for all. A part stays hidden while any resource hides it.

## The character panel's stats

The inventory's character panel shows each stat in a fixed place with its own icon. `Hud.setCharacterStats`
changes it: `hide` takes a stat off the panel, icon and all, and the stats below it in its column move up; `set`
shows your own label and whole number in a stat's place, such as a server's thirst and needs instead of speech
and noise. The stat names are `'lvl'`, `'health'`, `'exhaust'`, `'hunger'`, `'charisma'`, `'rsb'` (speed),
`'evi'` (visibility), `'con'` (conspicuousness), `'noi'` (noise), `'strength'`, `'agility'`, `'vitality'`
and `'speech'`. The panel fills each time the inventory opens, so call it again when a value changes, and `null`
brings the game's own panel back. `Hud.characterStats()` reads it back.

```js
// client
kcds.ui.Hud.setCharacterStats({
  hide: ['charisma'],
  set: [
    { stat: 'speech', label: 'Thirst', value: 40 },
    { stat: 'noi', label: 'Toilet', value: 75 },
  ],
});
```

```ts
// client
kcds.ui.Hud.setCharacterStats({
  hide: ['charisma'],
  set: [
    { stat: 'speech', label: 'Thirst', value: 40 },
    { stat: 'noi', label: 'Toilet', value: 75 },
  ],
});
```

```lua
-- client
kcds.ui.Hud.setCharacterStats({
  hide = { "charisma" },
  set = {
    { stat = "speech", label = "Thirst", value = 40 },
    { stat = "noi", label = "Toilet", value = 75 },
  },
})
```

```cs
// client
Hud.SetCharacterStats(new
{
    hide = new[] { "charisma" },
    set = new[]
    {
        new { stat = "speech", label = "Thirst", value = 40 },
        new { stat = "noi", label = "Toilet", value = 75 },
    },
});
```

## Screen effects

```js
// client
const { Hud } = kcds.ui;

Hud.startEffect(
  'screenBlood',
  { amount: 0.6 },
  { fadeInMs: 200, durationMs: 1500, fadeOutMs: 800 },
);
Hud.startEffect('colorGrade', { saturation: 0.3, brightness: 0.9 }); // grey and gloomy
```

```ts
// client
const { Hud } = kcds.ui;

Hud.startEffect(
  'screenBlood',
  { amount: 0.6 },
  { fadeInMs: 200, durationMs: 1500, fadeOutMs: 800 },
);
Hud.startEffect('colorGrade', { saturation: 0.3, brightness: 0.9 }); // grey and gloomy
```

```lua
-- client
local Hud = kcds.ui.Hud

Hud.startEffect("screenBlood", { amount = 0.6 }, { fadeInMs = 200, durationMs = 1500, fadeOutMs = 800 })
Hud.startEffect("colorGrade", { saturation = 0.3, brightness = 0.9 }) -- grey and gloomy
```

```cs
// client
Hud.StartEffect("screenBlood", new { amount = 0.6 }, new { fadeInMs = 200, durationMs = 1500, fadeOutMs = 800 });
Hud.StartEffect("colorGrade", new { saturation = 0.3, brightness = 0.9 }); // grey and gloomy
```

The effects are `colorGrade`, `blur`, `ghosting` and `screenBlood`. They combine, and end when your resource stops
or the level changes.

Also on the HUD: the game's notices `notifyPerkUse(icon, name)`, `notifyCombo(icon, name)`, `notifyReputation(eventId,
message)` and `notifyEventResult(type, result, name)`, and `clearNotifications()`; `hideBanner()`, `hideTutorial(name)`,
`closeTutorial()`, `hideCodexHint()`, `hideTutorialHint(id)` and `isTutorialHintShown(id)`; `hudPartShown(name)`;
`setHorseCompare(enabled)` and `isHorseCompare()` for the horse inspect screen; `changeEffect`, `endEffect`,
`endAllEffects` and `effectRunning` for screen effects; `inConversation()` while the player is in one of the game's own
conversations; and `worldToScreen(point)`, a world point's place on the screen in pixels.

Reference: [Hud](/docs/api/hud).
