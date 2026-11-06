# Character creator

`kcds.ui.CharacterCreator` opens the game's character stage, with a panel of choices next to the figure.

```js
// client
kcds.ui.CharacterCreator.open({ title: 'Who are you?', genders: 'both' });

kcds.core.Events.on('characterCreatorClosed', (appearance, reason) => {
  if (reason === 'accepted') {
    kcds.core.Events.toServer('saveLook', appearance);
  }
});
```

```lua
-- client
kcds.ui.CharacterCreator.open({ title = "Who are you?", genders = "both" })

kcds.core.Events.on("characterCreatorClosed", function(appearance, reason)
  if reason == "accepted" then
    kcds.core.Events.toServer("saveLook", appearance)
  end
end)
```

```cs
// client
CharacterCreator.Open(new { title = "Who are you?", genders = "both" });

Events.On("characterCreatorClosed", (JsValue appearance, string reason) =>
{
    if (reason == "accepted")
    {
        Events.ToServer("saveLook", appearance);
    }
});
```

It changes nothing itself. Send the accepted look to the server and apply it with `player.setAppearance`
([Players](players.md)).
The `freeroam` resource does this with `/creator`. <!-- only examples -->

Reference: [CharacterCreator](/docs/api/charactercreator).
