# Chat and translated texts

## Sending messages

The server can write to everyone or to one player, with an optional sender name:

```js
const { Chat } = kcds.core;

Chat.sendToAll('The tournament starts at noon.', { author: 'Herald' });
Chat.sendToPlayer(player, 'Only you can read this.');
```

```lua
local Chat = kcds.core.Chat

Chat.sendToAll("The tournament starts at noon.", { author = "Herald" })
Chat.sendToPlayer(player, "Only you can read this.")
```

```cs
Chat.SendToAll("The tournament starts at noon.", new { author = "Herald" });
Chat.SendToPlayer(player, "Only you can read this.");
```

## What happens to what players type

A line starting with `/` is a command. Nobody else sees it, and your script receives it as
`playerCommand(player, command, args)`.

Any other line is first sent to all players by the built-in relay, and then `playerChat(player,
text)` fires. For local chat, radio or roleplay channels, switch the relay off and deliver the lines
yourself:

```js
const { Chat, Events } = kcds.core;
const { Player } = kcds.players;

Chat.setChatRelay(false);

Events.on('playerChat', (speaker, text) => {
  for (const listener of Player.inRadius(speaker.position, 30)) {
    Chat.sendToPlayer(listener, text, { author: speaker.nickname });
  }
});
```

```lua
local Chat, Events = kcds.core.Chat, kcds.core.Events
local Player = kcds.players.Player

Chat.setChatRelay(false)

Events.on("playerChat", function(speaker, text)
  for _, listener in ipairs(Player.inRadius(speaker.position, 30)) do
    Chat.sendToPlayer(listener, text, { author = speaker.nickname })
  end
end)
```

```cs
Chat.SetChatRelay(false);

Events.On("playerChat", (Player speaker, string text) =>
{
    foreach (var listener in Player.InRadius(speaker.Position, 30))
    {
        Chat.SendToPlayer(listener, text, new { author = speaker.Nickname });
    }
});
```

## Chat in a client script

Inside a player's game:

- `kcds.core.Chat.send(text)` writes as that player.
- `kcds.core.Chat.open()` and `kcds.core.Chat.close()` open and close the chat window.
- `kcds.core.Chat.setUIVisible(visible)` shows or hides it.

## Translated texts

`Locales` holds texts by key and language. It covers KCDS's own short messages, such as the notice a
pickpocket's victim gets, a carry limit or a chest in use, and you can add keys for your own content.
Giving an existing key your own text replaces KCDS's wording.

For each player, the text is chosen in this order:

1. your text in the player's game language (`player.language`)
2. your English text
3. the built-in default

Placeholders such as `{name}` are filled from the values you pass.

```js
const { Chat, Events, Locales } = kcds.core;

Locales.set('english', { 'myshop.welcome': 'Welcome, {name}!' });
Locales.set('czech', { 'myshop.welcome': 'Vítej, {name}!' });

Events.on('playerReady', (player) => {
  const line = Locales.text(player, 'myshop.welcome', { name: player.nickname });
  Chat.sendToPlayer(player, line);
});
```

```lua
local Chat, Events, Locales = kcds.core.Chat, kcds.core.Events, kcds.core.Locales

Locales.set("english", { ["myshop.welcome"] = "Welcome, {name}!" })
Locales.set("czech", { ["myshop.welcome"] = "Vítej, {name}!" })

Events.on("playerReady", function(player)
  local line = Locales.text(player, "myshop.welcome", { name = player.nickname })
  Chat.sendToPlayer(player, line)
end)
```

```cs
Locales.Set("english", new Dictionary<string, string> { ["myshop.welcome"] = "Welcome, {name}!" });
Locales.Set("czech", new Dictionary<string, string> { ["myshop.welcome"] = "Vítej, {name}!" });

Events.On("playerReady", (Player player) =>
{
    var line = Locales.Text(player, "myshop.welcome", new { name = player.Nickname });
    Chat.SendToPlayer(player, line);
});
```

Also: `Chat.relaysChat()` says whether the default relay is on, and on a client `Chat.isUIVisible()` whether the chat
window shows. `Locales.defaultLanguage()` is the language used when a player's own has no text, `'english'`.

Reference: [Chat](/docs/api/chat), [Locales](/docs/api/locales).
