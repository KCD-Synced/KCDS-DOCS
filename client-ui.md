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

The KCD2 Synced watermark in the bottom left corner is not a part: the client draws it last, over the game's HUD
and every web view, and nothing turns it off. The [Server hosting rules](/server-rules) forbid hiding or covering
it in any other way.

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

The Player screen's list (Attributes, Derived) follows the same settings: a hidden stat leaves the list, and a set
one shows your label and number in its row. The list fills each time the Player screen opens. The client log names
the stats of both the first time each fills (`HUD character stats: the game's inventory panel rows` and
`the game's Player screen rows`).

The game fills the panel and the list in the same moment when the menu opens, so a resource cannot switch settings
between them. To give the list settings of its own, add `list`: its own `{ hide, set }`, or `null` to leave the game's
list as it is while the panel shows yours.

```js
// client
kcds.ui.Hud.setCharacterStats({
  set: [{ stat: 'speech', label: 'Thirst', value: 40 }],
  list: { hide: ['speech'] },
});
```

```ts
// client
kcds.ui.Hud.setCharacterStats({
  set: [{ stat: 'speech', label: 'Thirst', value: 40 }],
  list: { hide: ['speech'] },
});
```

```lua
-- client
kcds.ui.Hud.setCharacterStats({
  set = { { stat = "speech", label = "Thirst", value = 40 } },
  list = { hide = { "speech" } },
})
```

```cs
// client
Hud.SetCharacterStats(new
{
    set = new[] { new { stat = "speech", label = "Thirst", value = 40 } },
    list = new { hide = new[] { "speech" } },
});
```

### Your own icon on a stat row

A stat you `set` keeps the game's icon. To show your own, load a picture into the row's icon with
[NativeElement](/docs/api/nativeelement)`.load` once the panel is open: the inventory's character panel is the
element `ApseCharacter`, and the game fills it again each time it opens. Each stat keeps its row clip, also when
`hide` moves it up:

| Stat       | Icon clip                                        |
| ---------- | ------------------------------------------------ |
| `health`   | `main.stats.soulProperties.soulProperty1.icon`   |
| `exhaust`  | `main.stats.soulProperties.soulProperty2.icon`   |
| `hunger`   | `main.stats.soulProperties.soulProperty3.icon`   |
| `charisma` | `main.stats.soulProperties.soulProperty4.icon`   |
| `rsb`      | `main.stats.soulProperties.soulProperty5.icon`   |
| `evi`      | `main.stats.soulProperties.soulProperty6.icon`   |
| `con`      | `main.stats.soulProperties.soulProperty7.icon`   |
| `noi`      | `main.stats.soulProperties.soulProperty8.icon`   |
| `strength` | `main.stats.soulProperties.soulProperty9.icon`   |
| `agility`  | `main.stats.soulProperties.soulProperty10.icon`  |
| `vitality` | `main.stats.soulProperties.soulProperty11.icon`  |
| `speech`   | `main.stats.soulProperties.soulProperty12.icon`  |

The picture is a `.dds` your resource lists in the manifest's `files` (`"files": ["icons/*.dds"]`), named by its
path in the resource. A square picture of 64 x 64 pixels shows at the size of the game's own icon.

```js
// client: thirst in place of speech, with its own icon
kcds.ui.Hud.setCharacterStats({ set: [{ stat: 'speech', label: 'Thirst', value: 40 }] });

kcds.core.Events.on('nativeScreenOpened', (name) => {
  if (name === 'ApseCharacter') {
    kcds.ui.NativeUI.element('ApseCharacter').load(
      'main.stats.soulProperties.soulProperty12.icon',
      'icons/thirst.dds',
    );
  }
});
```

```ts
// client: thirst in place of speech, with its own icon
kcds.ui.Hud.setCharacterStats({ set: [{ stat: 'speech', label: 'Thirst', value: 40 }] });

kcds.core.Events.on('nativeScreenOpened', (name: string) => {
  if (name === 'ApseCharacter') {
    kcds.ui.NativeUI.element('ApseCharacter').load(
      'main.stats.soulProperties.soulProperty12.icon',
      'icons/thirst.dds',
    );
  }
});
```

```lua
-- client: thirst in place of speech, with its own icon
kcds.ui.Hud.setCharacterStats({ set = { { stat = "speech", label = "Thirst", value = 40 } } })

kcds.core.Events.on("nativeScreenOpened", function(name)
  if name == "ApseCharacter" then
    kcds.ui.NativeUI.element("ApseCharacter"):load("main.stats.soulProperties.soulProperty12.icon", "icons/thirst.dds")
  end
end)
```

```cs
// client: thirst in place of speech, with its own icon
Hud.SetCharacterStats(new { set = new[] { new { stat = "speech", label = "Thirst", value = 40 } } });

Events.On("nativeScreenOpened", (string name) =>
{
    if (name == "ApseCharacter")
    {
        NativeUI.Element("ApseCharacter").Load("main.stats.soulProperties.soulProperty12.icon", "icons/thirst.dds");
    }
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
