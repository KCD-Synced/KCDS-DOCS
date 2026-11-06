# "Press [key] to ..." hints

`kcds.ui.ActionHint` adds a prompt to this player's screen, as the game's own "E Talk" prompts:

```js
// client
kcds.ui.ActionHint.create(
  { key: 'primary_action', text: 'Ring the bell', mode: 'hold', holdDuration: 2 },
  () => {
    kcds.core.Events.toServer('ringBell');
  },
);
```

```ts
// client
kcds.ui.ActionHint.create(
  { key: 'primary_action', text: 'Ring the bell', mode: 'hold', holdDuration: 2 },
  () => {
    kcds.core.Events.toServer('ringBell');
  },
);
```

```lua
-- client
kcds.ui.ActionHint.create(
  { key = "primary_action", text = "Ring the bell", mode = "hold", holdDuration = 2 },
  function()
    kcds.core.Events.toServer("ringBell")
  end
)
```

```cs
// client
ActionHint.Create(
    new { key = "primary_action", text = "Ring the bell", mode = "hold", holdDuration = 2 },
    () =>
    {
        Events.ToServer("ringBell");
    });
```

**Keys:** a hint sits on one of the game's controls: `primary_action`, `attack_special`, `torch`,
`companion_interaction`, `distract` or `jump`. Or it can use `script_action`, a free key KCDS keeps for scripts
(U). The newest hint on a control is the one shown.

**An enabled hint takes over its key.** The game's action on that key does not run, so E puts down no sack and
starts no talk. The game's prompts on that key (Talk, Loot, Pickpocket behind a player) are not drawn next to it. A
disabled hint takes nothing.

**Presses:** one press calls the handler once. A hint appearing while the key is still held waits for the next
press, for example "Release" right after "Put in the pillory". `kcds.world.World.addInteraction` behaves the same.

**Many interactions:** a player's game gets at most 255 of the `World.addInteraction` hints they can see. When a server
has more, each player gets the 255 nearest to them, picked again every second as they move, so you do not have to
narrow them yourself with `setPlayers`. A hint on a level entity by name, whose place the server does not know, comes
after the hints whose place it knows.

## A press and a hold on one key

Press hints and hold hints stack separately, so a key can have both, as the game's prompts do. The newest of each
shows. Letting go before `holdDuration` runs the press hint, and a full hold runs the hold hint. E, X and the special
attack key draw both. Other keys draw only the hold hint, but both still work.

```js
// client
const { ActionHint } = kcds.ui;
const { Events } = kcds.core;

ActionHint.create({ key: 'companion_interaction', text: 'Untie' }, () => {
  Events.toServer('untie');
});
ActionHint.create(
  { key: 'companion_interaction', text: 'Lead on a rope', mode: 'hold', holdDuration: 1 },
  () => Events.toServer('lead'),
);
```

```ts
// client
const { ActionHint } = kcds.ui;
const { Events } = kcds.core;

ActionHint.create({ key: 'companion_interaction', text: 'Untie' }, () => {
  Events.toServer('untie');
});
ActionHint.create(
  { key: 'companion_interaction', text: 'Lead on a rope', mode: 'hold', holdDuration: 1 },
  () => Events.toServer('lead'),
);
```

```lua
-- client
local ActionHint = kcds.ui.ActionHint
local Events = kcds.core.Events

ActionHint.create({ key = "companion_interaction", text = "Untie" }, function()
  Events.toServer("untie")
end)
ActionHint.create(
  { key = "companion_interaction", text = "Lead on a rope", mode = "hold", holdDuration = 1 },
  function() Events.toServer("lead") end
)
```

```cs
// client
ActionHint.Create(new { key = "companion_interaction", text = "Untie" }, () =>
{
    Events.ToServer("untie");
});
ActionHint.Create(
    new { key = "companion_interaction", text = "Lead on a rope", mode = "hold", holdDuration = 1 },
    () => Events.ToServer("lead"));
```

## Drawing hints yourself

`ActionHint.shown()` lists all hints the HUD is drawing, including the game's prompts, as `{ key, map, text, mode,
enabled, progress }`, with text in the player's language. `ActionHint.shownKeys()` and `ActionHint.isKeyInUse(key)`
tell which keys a hint is waiting on, and `actionHintsChanged` fires when the set changes. So a resource can hide the
game's hints and draw its own:

```js
// client
kcds.ui.Hud.setHudPart('Hints', false);

kcds.core.Events.on('actionHintsChanged', () => {
  console.log(JSON.stringify(kcds.ui.ActionHint.shown()));
});
```

```ts
// client
kcds.ui.Hud.setHudPart('Hints', false);

kcds.core.Events.on('actionHintsChanged', () => {
  console.log(JSON.stringify(kcds.ui.ActionHint.shown()));
});
```

```lua
-- client
kcds.ui.Hud.setHudPart("Hints", false)

kcds.core.Events.on("actionHintsChanged", function()
  print(json.encode(kcds.ui.ActionHint.shown()))
end)
```

```cs
// client
Hud.SetHudPart("Hints", false);

Events.On("actionHintsChanged", () =>
{
    Log.Info(ActionHint.Shown().ToJson());
});
```

While the player lies knocked out or fainted, the game's world hints, such as loot or picking up a body, are not
drawn. Your hints and those of the pause screen still are.

`kcds.ui.Web.anyFocused()` says whether any resource's shown page has the keyboard and mouse.
`ActionHint.setText(id, text)` changes a hint's text.

The server's own prompts on a place, a level entity or a body, the same for every player, are
`World.addInteraction` ([World](/docs/api/world)).

Reference: [ActionHint](/docs/api/actionhint).
