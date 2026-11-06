# Local NPCs and the local player

## Local NPCs

`kcds.characters.LocalNpc` is a body shown only on this machine, such as a menu figure, a guide or a preview. A
machine can have up to 64, and they disappear when your resource stops.

```js
// client
const guide = kcds.characters.LocalNpc.create({
  soul: '210170d8-afb2-4cbd-9df5-da2a8e722af4',
  name: 'Guide',
  position: new kcds.math.Vector3(100, 200, 10),
});
guide.say('Follow the road to the mill.');
```

```lua
-- client
local guide = kcds.characters.LocalNpc.create({
  soul = "210170d8-afb2-4cbd-9df5-da2a8e722af4",
  name = "Guide",
  position = kcds.math.Vector3.new(100, 200, 10),
})
guide:say("Follow the road to the mill.")
```

```cs
// client
var guide = LocalNpc.Create(new
{
    soul = "210170d8-afb2-4cbd-9df5-da2a8e722af4",
    name = "Guide",
    position = new Vector3(100, 200, 10),
});
guide.Say("Follow the road to the mill.");
```

NPCs every player sees come from the server; see [NPCs](npcs.md).

## The local player

`kcds.players.LocalPlayer` is the `Player` at this machine, or `null` while loading. Compare ids with it when the
server sends an event meant for one player.

Reference: [LocalNpc](/docs/api/localnpc), [LocalPlayer](/docs/api/localplayer).
