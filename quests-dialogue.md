# Quests and conversations

## Quests

`kcds.characters.Quest` writes entries into the game's own journal. The server stores each quest, so
players who join late or reload still have it. A quest goes to one player or, when you leave out
`player`, to everyone in its dimension.

```js
const { Quest } = kcds.characters;
const { Vector3 } = kcds.math;

const delivery = Quest.give('deliver_grain', 'Grain for the mill', {
  player,
  description: 'The miller needs three sacks of grain.',
  objectives: [
    { text: 'Collect the grain', position: new Vector3(512, 830, 31) },
    { text: 'Bring it to the miller' },
  ],
  track: true,
});

delivery.setObjective(0, { text: 'Collect the grain', progress: 'done' });
delivery.setProgress('done');
```

```lua
local Quest = kcds.characters.Quest
local Vector3 = kcds.math.Vector3

local delivery = Quest.give("deliver_grain", "Grain for the mill", {
  player = player,
  description = "The miller needs three sacks of grain.",
  objectives = {
    { text = "Collect the grain", position = Vector3.new(512, 830, 31) },
    { text = "Bring it to the miller" },
  },
  track = true,
})

-- Objectives are numbered from 0, as the SDK numbers them.
delivery:setObjective(0, { text = "Collect the grain", progress = "done" })
delivery:setProgress("done")
```

```cs
var delivery = Quest.Give("deliver_grain", "Grain for the mill", new
{
    player,
    description = "The miller needs three sacks of grain.",
    objectives = new object[]
    {
        new { text = "Collect the grain", position = new Vector3(512, 830, 31) },
        new { text = "Bring it to the miller" },
    },
    track = true,
});

delivery.SetObjective(0, new { text = "Collect the grain", progress = "done" });
delivery.SetProgress("done");
```

A quest has up to 8 objectives. An objective with a `position` shows on the map while the quest is
being tracked.

| Setting    | Values                                                                  |
| ---------- | ----------------------------------------------------------------------- |
| `type`     | `'main'`, `'side'` (default), `'activity'`, `'event'`, `'micro'`, `'racing'` |
| `progress` | `'active'`, `'done'`, `'failed'`                                        |
| `announce` | `false` leaves out the game's notification                              |

To look quests up, use `Quest.find(key, dimension?, player?)` or `Quest.all()`. On a quest, `remove()`
deletes it and `track(player?)` and `untrack(player?)` change tracking. The event
`questTrackingChanged(quest, player, tracked)` tells you when tracking changes.

## Conversations

`kcds.characters.Dialogue` opens the game's dialogue list with a line above it. Your script decides
what each choice does.

```js
const { Dialogue } = kcds.characters;
const { Events } = kcds.core;

Events.on('npcInteract', (npc, player) => {
  const page = {
    line: 'What brings you to the mill?',
    options: [
      { id: 'work', text: 'I am looking for work.' },
      { id: 'bye', text: 'Nothing. Farewell.' },
    ],
  };
  Dialogue.open(player, page, { npc });
});

Events.on('dialogueChoice', (session, player, optionId) => {
  if (optionId === 'bye') {
    Dialogue.close(session);
    return;
  }
  Dialogue.update(session, {
    line: 'Carry these sacks.',
    options: [{ id: 'bye', text: 'Right away.' }],
  });
});
```

```lua
local Dialogue = kcds.characters.Dialogue
local Events = kcds.core.Events

Events.on("npcInteract", function(npc, player)
  local page = {
    line = "What brings you to the mill?",
    options = {
      { id = "work", text = "I am looking for work." },
      { id = "bye", text = "Nothing. Farewell." },
    },
  }
  Dialogue.open(player, page, { npc = npc })
end)

Events.on("dialogueChoice", function(session, player, optionId)
  if optionId == "bye" then
    Dialogue.close(session)
    return
  end
  Dialogue.update(session, {
    line = "Carry these sacks.",
    options = { { id = "bye", text = "Right away." } },
  })
end)
```

```cs
Events.On("npcInteract", (Npc npc, Player player) =>
{
    var page = new
    {
        line = "What brings you to the mill?",
        options = new[]
        {
            new { id = "work", text = "I am looking for work." },
            new { id = "bye", text = "Nothing. Farewell." },
        },
    };
    Dialogue.Open(player, page, new { npc });
});

Events.On("dialogueChoice", (JsValue session, Player player, string optionId) =>
{
    if (optionId == "bye")
    {
        Dialogue.Close(session);
        return;
    }
    Dialogue.Update(session, new
    {
        line = "Carry these sacks.",
        options = new[] { new { id = "bye", text = "Right away." } },
    });
});
```

Each page needs 1 to 8 options with different ids. Set `enabled: false` to grey one out.

You can also say when a conversation should end by itself:

- `npc`: when this speaker dies
- `position` and `radius`: when the player walks away
- `timeout`: after a set time

The event `dialogueClosed(session, player, reason)` reports the end. The reason is `0` when your
script closed it, `1` when the player backed out, `2` when another dialogue replaced it and `3` when
it was interrupted.

The player moves through the list with W and S or the arrow keys (the game's forward and back keys)
and picks a row with the use key (E) or Enter, or with its number key. Only a fresh press of E picks: the E that opened
the list at an NPC, still held or pressed again within 0.35 s of a page showing, picks nothing, so you can open a
`Dialogue` right from a `World.addInteraction` handler on E. Backspace backs out. While
the list shows, those keys reach nothing else: a `World.addInteraction` prompt or an `ActionHint` on
the same key waits until the conversation ends.

## The game's own conversations

Conversations from the game with its NPCs are always off on KCDS servers, because they crashed
players' games (the "Vyřešíme to spolu" option). Use `Dialogue` instead.

Talking to another player is a separate game feature that you can enable with
`kcds.players.Interactions.set('playerTalk', true)`; see [Server rules](server-rules.md).

Also: `quest.setObjectives(objectives, announce)` replaces all of a quest's objectives at once, up to 8, and
`quest.objectiveCount` counts them.

Reference: [Quest](/docs/api/quest), [Dialogue](/docs/api/dialogue).
