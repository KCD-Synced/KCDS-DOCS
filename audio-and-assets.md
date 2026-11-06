# Sounds and game files

Resources can play the game's sounds, play their own `.ogg` files as speech, and add or replace game
files such as models, textures, sounds and animations.

## Playing the game's sounds

`kcds.world.Audio` is client-only and plays sounds on that one machine. For a sound everyone should
hear, have the server tell all clients to play it:

```js
// server
kcds.core.Events.broadcast('bells:ring', { x: 1998, y: 2498.75, z: 116 });
```

```lua
-- server
kcds.core.Events.broadcast("bells:ring", { x = 1998, y = 2498.75, z = 116 })
```

```cs
// server
Events.Broadcast("bells:ring", new { x = 1998, y = 2498.75, z = 116 });
```

```js
// client
kcds.core.Events.on('bells:ring', (spot) => {
  kcds.world.Audio.playAt('a_o_bell_kkut_kostelni', spot);
});
```

```lua
-- client
kcds.core.Events.on("bells:ring", function(spot)
  kcds.world.Audio.playAt("a_o_bell_kkut_kostelni", spot)
end)
```

```cs
// client
Events.On("bells:ring", (JsValue spot) =>
{
    Audio.PlayAt("a_o_bell_kkut_kostelni", spot);
});
```

| Call                                     | What it does                                       |
| ---------------------------------------- | -------------------------------------------------- |
| `play(trigger)`                          | plays at the local player                          |
| `playAt(trigger, position)`              | plays at a point                                   |
| `loopAt(trigger, position)`              | keeps playing at a point; returns its number       |
| `stopLoop(loop)`                         | stops a sound `loopAt` keeps playing               |
| `playFrom(trigger, entityId)`            | plays on an entity and follows it                  |
| `stopFrom(trigger, entityId)`            | stops that sound                                   |
| `setSoundParameter(entityId, name, value)` | sets one of the game's audio parameters          |
| `setSoundSwitch(entityId, name, value)`  | sets one of the game's audio switches              |
| `triggerExists(trigger)`                 | true when the game's audio data has this name      |

```js
// client
const { Audio } = kcds.world;

Audio.play('v_horse_whistle');
Audio.setSoundParameter(horseId, 'horse_speed', 0.5);
Audio.setSoundSwitch(playerId, 'sword_mat', 'mat_metal');
```

```lua
-- client
local Audio = kcds.world.Audio

Audio.play("v_horse_whistle")
Audio.setSoundParameter(horseId, "horse_speed", 0.5)
Audio.setSoundSwitch(playerId, "sword_mat", "mat_metal")
```

```cs
// client
Audio.Play("v_horse_whistle");
Audio.SetSoundParameter(horseId, "horse_speed", 0.5);
Audio.SetSoundSwitch(playerId, "sword_mat", "mat_metal");
```

Each call returns whether the game accepted it. You get `false` before a world has loaded, for a name
the game does not have, and for an entity that is not there. Names can be up to 256 bytes, and
positions must be within 16 km of the map's origin.

## Speech from a file

`Audio.playFile(file, entityId, purpose?)` plays an `.ogg` on an entity like a spoken line. It comes
from that position, fades with distance and follows the player's voice volume.

```js
// client: your file is stream/sounds/kcds/guards/halt.ogg
kcds.world.Audio.playFile('sounds/kcds/guards/halt', guardId, '20');
```

```lua
-- client: your file is stream/sounds/kcds/guards/halt.ogg
kcds.world.Audio.playFile("sounds/kcds/guards/halt", guardId, "20")
```

```cs
// client: your file is stream/sounds/kcds/guards/halt.ogg
Audio.PlayFile("sounds/kcds/guards/halt", guardId, "20");
```

**`file`** is a game path, with or without `.ogg`. It can be a file you ship, where
`stream/sounds/kcds/<resource>/line.ogg` becomes `sounds/kcds/<resource>/line`, or one of the game's
lines under `Localization/dialog/...`. A path with a backslash, a drive, a leading `/` or `..` throws.
A file the game cannot find plays nothing.

**`purpose`** sets how it sounds:

| Value                                | Meaning                                     |
| ------------------------------------ | ------------------------------------------- |
| `inner`                              | an inner voice                              |
| `10`, `20`, `35`, `50`, `100`, `200` | heard up to that many metres; default is `50` |
| `sfp_dialog`, `sfp_cutscene`, `sfp_music` | the game's own sound categories        |

One file per entity can start every 0.1 s.

## Voice chat

Players near each other talk through KCDS's own voice chat, built into the server. Its ranges, the
players' settings and the rules a server sets are in [Voice chat](/docs/voice).

## Adding game files: stream/

Your resource's `stream/` folder holds new files at their game paths, in a folder named after your
resource: `stream/<top folder>/kcds/<resource>/...`. Each player running the resource receives them, and
the game loads them by path like its own.

```text
resources/
  furniture/
    package.json
    server.js
    stream/
      objects/kcds/furniture/chair.cgf
      objects/kcds/furniture/chair.cgfm
      sounds/kcds/furniture/creak.ogg
```

```js
// server: spawn the chair by its full path
kcds.world.Prop.spawn('objects/kcds/furniture/chair.cgf', new kcds.math.Vector3(1650, 2150, 37));
```

```lua
-- server: spawn the chair by its full path
kcds.world.Prop.spawn("objects/kcds/furniture/chair.cgf", kcds.math.Vector3.new(1650, 2150, 37))
```

```cs
// server: spawn the chair by its full path
Prop.Spawn("objects/kcds/furniture/chair.cgf", new Vector3(1650, 2150, 37));
```

```js
// client: check whether this game has it yet
if (kcds.core.Assets.has('objects/kcds/furniture/chair.cgf')) {
  console.log('the chair is here');
}
```

```lua
-- client: check whether this game has it yet
if kcds.core.Assets.has("objects/kcds/furniture/chair.cgf") then
  print("the chair is here")
end
```

```cs
// client: check whether this game has it yet
if (Assets.Has("objects/kcds/furniture/chair.cgf"))
{
    Log.Info("the chair is here");
}
```

- `stream/` and `replace/` always reach clients, whatever `files` says.
- A file outside `<top folder>/kcds/<resource>/` is refused, so a streamed file cannot take the place
  of a game file or another resource's file.
- To play a `.caf` animation you ship, use `player.playAnimation('', { clip })`; see
  [Animations](animations.md).

## Replacing game files: replace/

A file at `replace/<game path>` is used in place of the game's file at that path, for every client
running the resource:

```text
replace/objects/manmade/task_specific_props/household/cooking_eating/eating_tools/wooden_scoop.cgf
```

Keep in mind:

- A joining player's game puts these files in before it loads the world, so even the level's own objects,
  such as a sign in a town, use yours. Joining waits for the server's client resources for up to two minutes.
- A resource started while players are already in the world reaches only what their games load afterwards. A
  model or texture already loaded stays as it was until the game loads it again, at the latest when they join
  again.
- `kcds.core.Assets.has` does not count replaced files.
- Every player who joins downloads these files, so replace only what your server needs.

## Custom textures

A texture is a game file like any other: put your `.dds` at the game texture's path under `replace/`, and every
player sees it in place of the game's.

```text
resources/
  armourer/
    package.json
    replace/
      objects/manmade/task_specific_props/metal_industry/armoursmith/shop_sign_armourer_fancy_diff.dds
```

Every armourer's sign with this texture, the ones the level places in its towns and those a script spawns,
shows yours.

- **One file:** save the texture as a DDS with all its mip levels. The game keeps its own textures split
  into the `.dds` and `.dds.1`, `.dds.2` and further files; yours replaces the whole texture and needs no
  others.
- **Same layout:** a model maps its texture by its own coordinates, so paint over the game's texture rather
  than drawing a new picture, or parts land in the wrong places.
- **Format:** a compressed format (BC1, or BC3 or BC7 with transparency) keeps the download small. A colour
  texture (`_diff`) is read as sRGB.
- **Other maps:** the normal map (`_ddna`) and the other textures of a material are replaced the same way.

For a model of your own, put its textures in `stream/` beside it, and name them in its material (`.mtl`) by
their full paths, such as `objects/kcds/furniture/chair_diff.dds`.

`Audio.silenceEntity(entityId)` stops every sound on an entity in this game, the game's own included.

Reference: [Audio](/docs/api/audio), [Assets](/docs/api/assets).
