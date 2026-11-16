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

```ts
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

`CharacterCreator.previewClothes(items)` dresses the figure in clothing and armour, by item class name or GUID,
for an outfit chooser of your own: nothing is given to the player and their equipment is not touched. The
pieces go when the preview changes or the creator closes, and `[]` or `null` takes the preview off. The game may
still refuse a piece the figure cannot wear, such as a woman's piece on a man. The accepted look does not include
the clothes: give and equip them from your server once the player confirms.
The `freeroam` resource does this with `/creator`. <!-- only examples -->

Reference: [CharacterCreator](/docs/api/charactercreator).
