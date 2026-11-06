# Asset/Animation Studio

The Asset/Animation Studio is where you make new things for the game: props with their own
physics, such as a rope, a chain or a whip; your own meshes from Blender, glTF or FBX, as props,
as cloth or worn on the character; textures; and animations of the character, posed and keyed on
a timeline or brought from a file. It is part of KCDS, like [World Architect](world-architect.md).
It runs only on your own machine, in the game's developer mode, with no server. No server can turn
it on and no resource script can open it. What you make goes to your server as a resource.

## Opening it

In KCDS's main menu, press **ANIM / ASSETS** in the row above the server list, between the
refresh button and **WORLD ARCHITECT**. The game you pressed it in goes into its developer mode
and loads the studio's own empty level straight from the main menu, with no second game. From
then on it shows no server list and joins no server. KCDS's loading screen stays up until the
studio is drawn, and the studio opens with the scene you left there last time. **WORLD ARCHITECT** next to it loads
[World Architect](world-architect.md) the same way. Both buttons work only on the server list,
not while you are joining a server. **Leave** at the right end of the menu bar saves your work,
takes you back to the main menu and shows the server list again, so you can join a server or
open a tool again in the same game. A game you started in the studio with the development
launcher (below) has no server list, so there Leave closes the game.

You can also start the game with KCDS's development launcher yourself, which runs the game in
its developer mode. Set both `KCDS_WORLD_ARCHITECT=1` and `KCDS_ASSET_STUDIO=1` in its
environment. Without the developer mode, neither setting does anything.

The studio's level is a flat 128 m floor under a clear sky at noon. The clock is stopped, so the
light stays the same. No map, buildings, plants or people are loaded. The floor you see is the
studio's own cobblestone floor (the game's own texture), drawn in every start; the toolbar's
**Floor** switch hides and shows it.

Under the studio's floor lies the level's own ground, which the character stands on. The Floor
switch hides both, so with it off nothing is drawn under the character but the grid, and the
character still stands.

## The screen

The studio uses World Architect's screen layout, with its own content:

- **Menu bar (top):** the studio's name (a button that opens the Saves tab), the name of the
  save, `Saved · local only` and **Leave** (back to the main menu and the server list).
- **Toolbar:**
  - the tools (Select, Move, Rotate, Scale);
  - the studio's switches: where the prop is held (Right hand, Left hand, Free), Floor, Swing
    test, and **Grid** (on at first: one-metre squares on the floor, 20 m across round the
    character, the world's X axis red and Y axis green, hidden behind what stands on it);
  - the modes: Props, Physics, Animation, Textures and Bones (keys 1 to 5).
  - Undo, Redo, and **View as character**: the character's own first-person view with the
    studio still open. Press **Free camera** (the same button) to fly again.
- **Outliner (left):** the scene, which is the character and the props you made.
- **Details (right):** the chosen prop's settings. Values with `-` and `+` change by one step.
  As in World Architect, press a value and hold the button: moving the mouse sideways steps it
  on (right up, left down, one step every 12 pixels), and the whole drag is one step you can
  undo. A list longer than the panel scrolls with the mouse wheel over it, with a bar at its
  right edge showing where you are; it goes back to the top when you choose something else.
  World Architect's Details scroll the same way.
- **Content (bottom):** what the studio can make. **Rope** and **Whip** work now. Chain and your
  own mesh are listed as later. The dock's other tabs are **Timeline**, **History**, **Saves**
  and **AI** (connecting an AI assistant, [below](#working-with-an-ai-assistant-mcp)).

Hold the **right mouse button** to look around with the mouse and fly with **W A S D**
(**Space** up, **C** down, **Shift** faster, **Alt** slower). As in Blender, hold the **middle
mouse button** over the view and drag to orbit round what is chosen (a free prop or your mesh,
else the character), with **Shift** to pan and with **Ctrl** to zoom (up nearer, down farther).
**Numpad 1**, **3** and **7** show it from the front, the side and the top. **F** frames what is
chosen, or brings the camera back in front of the character. **Ctrl+Z** and **Ctrl+Y** undo and
redo. **Escape** lets go of the selection.

The character stands still in the middle of the floor: its own idle is held while the studio
is open, so the poses you make are all that moves it.

### The scene's sky

Choose **Scene** at the top of the outliner. Its details set the sky:

| Detail      | Does                                                               | At first |
| ----------- | ------------------------------------------------------------------ | -------- |
| Time of day | The hour the clock stands still at, round the clock               | 12:00    |
| Weather     | The game's weather: Clear, Overcast with rain, Storm              | Clear    |
| Rain        | How hard it rains, in tenths                                       | None     |

The project keeps the sky.

### The character's clothes

Choose **Character** in the outliner. **Outfit** steps through the game's own outfits (Henry,
Henry travelling, Nothing), and each piece of the outfit has its own switch. The project keeps
what the character wears.

The character puts the pieces on as soon as the studio opens, and changes when you change
them.

## A rope in the hand

Press **Rope** in Content. The game makes a rope with its own rope physics, the same kind the
game uses for its own ropes. The rope's start is in the character's right hand, and the rest
hangs, swings and lies on the floor.

| Detail     | Does                                                                         |
| ---------- | ---------------------------------------------------------------------------- |
| Segments   | How many physics segments the rope has: 2 to 64, in steps of 2 (16 at first) |
| Length     | The rope's length: 0.2 m to 20 m, in steps of 0.1 m (1.2 m at first)          |
| Held in    | Right hand (at first), Left hand, or Free in the scene (see below)           |
| Pin end    | The rope's end tied to a point, so the rope hangs between its two points; **End X/Y/Z** move the point (steps of 0.05 m). Off: the end hangs free |
| Physics    | Opens Physics mode with how the rope moves                                   |
| Remove     | Takes the rope away                                                          |

### Held in, and moving a prop

**Held in** (the ‹ › of its row, or the first chip in the toolbar) steps through the right hand,
the left hand and **Free in the scene**. A new prop is held where the one before was. Free, the
prop stands where you put it, and Details show **Place X/Y/Z** (steps of 0.05 m) and **Turn
X/Y/Z** (steps of 15°, about the world's axes).

Press the prop in the view to choose it. What you chose is marked in orange (a cross on the
prop's start or the rope's end, the box round your mesh), and the gizmo shows there (for a
whip, the hand's place on the handle). With **Pin end** on, press the rope's or chain's end in
the view to move that end the same way (it only moves; pressing the prop again, or choosing it
in the outliner, puts the gizmo back on its start):

- **Select** or **Move** tool: three arrows, X red, Y green, Z blue. Drag an arrow and the prop
  slides along that axis.
- **Rotate** tool: three rings. Drag across a ring and the prop turns about that axis, half a
  degree a pixel.

Dragging a prop that is in a hand sets it free where it is. Each drag is one step you can undo,
and the project keeps where the prop is held.

## A chain

Press **Chain** in Content: the same rope physics, drawn with the game's chain textures (24
segments and 1.5 m at first). It has the rope's details: Segments, Length, Held in, Pin end and
the Physics settings (a heavier Weight and a small Bend limit make it hang like a chain).
`player.setPhysicalProp({ kind: 'chain' })` gives one to a player on a server, and the export
does so for a chain.

*Not yet in this version:* the chain is drawn as a smooth metal band with the chain texture on
it; its links are not shaped one by one.

## A whip in the hand

Press **Whip** in Content. The character holds the game's own whip handle (the horse whip) in
the right hand, and a lash made with the game's rope physics hangs from the handle's tip. The
lash's details are the same as a rope's: Segments (20 at first), Length (1.6 m at first), Held
in and Remove.

The hand holds the handle by its end. **Grip angle** turns the handle about the hand: 90° (the
start) points it out of the fist, 0° down along the forearm; **Grip twist** turns it about the
forearm. Both go in steps of 15°. In the left hand the same grip is mirrored, so the handle
points out of that fist too. A free whip stands with the hand's place on the handle at its place,
turned by its turn and then its grip.

The studio's scene holds one prop: a whip made after a rope takes the rope's place, and the
other way round.

### How a rope or a lash moves

Press **Physics** (mode 2, or the Physics row of the prop's Details) with the prop chosen:

| Detail     | Does                                                                  | At first |
| ---------- | --------------------------------------------------------------------- | -------- |
| Swing test | The prop's start goes round a circle fast (a rope 0.8 m, a whip 0.3 m, 1.5 times a second), and the rope or lash swings round after it | Off |
| Gravity    | How hard it falls, 0 to 30 m/s²                                       | 9.8      |
| Damping    | How fast its swinging dies down, 0 to 5                               | 0.3      |
| Weight     | The whole rope, 0.05 to 10 kg                                         | 0.5 kg   |
| Air drag   | How much the air slows it, 0 to 2                                     | 0.1      |
| Bend limit | How far each joint may bend, 0° to 90°: Free bends like a limp rope, a limit stiffens it | Free |
| Wind       | Wind along the world's x, 0 to 20 m/s                                 | 0        |
| Stiffness  | How hard it resists being stretched, 0 to 500 (0: the game's own)     | 0        |
| Friction   | How it slides on what it lies on, 0 to 2                              | 0.5      |
| Thickness  | How far round its line it collides with the floor and bodies, 0.005 to 0.2 m | 0.02 m |
| Collide    | On: it collides with the character, props and the floor; Floor only: with the floor alone, so a lash swings past the body that holds it instead of catching on the head and shoulders; Off: it falls through everything | On |

Each change is one step you can undo, and the project keeps them.

A whip's lash moves its own way. The game's rope has one weight and one thickness all along,
so it only hangs and swings from the hand. A whip cracks because its lash gets lighter toward the
tip: the wave the hand starts runs faster and faster down it. So KCDS moves a whip's lash with
its own tapered lash (stiff at the handle, limp and light at the tip, leaving the handle along
its long side) and draws it, 14 mm thick, with the game's rope. Its tip outruns the hand (2 to 3
times on a strike), it trails behind on the back cast, loops over the hand and rolls out
straight in front before it falls. For a whip, **Gravity**, **Damping**, **Air drag** and
**Collide** (with the floor or not) apply; Weight, Bend limit, Stiffness, Friction, Thickness and
Wind belong to ropes and chains.

**Ignore characters** (Physics, a switch) says whether the prop passes through the bodies of
the character, players and NPCs. On, it lies only on the floor (Collide: Floor only); off, a
rope or a chain also collides with the bodies (Collide: On). A new whip has it on, so its lash
swings past the body that holds it. Off, a whip's lash collides with the body that holds it
and with every other character within 3 m of the hand (players and NPCs): their torso, head,
arms and legs (capsules along each skeleton; the forearm of the holding hand is left out, as the
lash leaves that hand).

Local NPCs (`LocalNpc`) also live in the studio's world, so a character can stand in the
lash's way while you set up a whipping.

#### A posed lash: animating the swing yourself

A whip's Physics also has **Shape**, **Curl** and **Sway**: the lash posed instead of moved by
physics.

| Detail | Does                                                                                                   |
| ------ | ------------------------------------------------------------------------------------------------------ |
| Shape  | 0: physics alone (as before); 1: the lash holds its posed shape exactly, with no gravity; between: it is pulled toward the shape (0.5 follows it closely, 0.2 within about half a second, 0.1 barely) |
| Curl   | How far the posed lash bends from the handle to the tip toward the handle's up side, -540° to 540° (0 is straight out of the handle, 180 a half circle back over it, negative bends the other way) |
| Sway   | How far it bends sideways, -540° to 540°                                                              |

All three are prop key settings (below), so they blend from key to key along the animation. To
animate a swing exactly, put Shape 1 on the keys of the wind-up and set Curl and Sway on each key
to the shape you want, then a key with Shape 0 where the lash should fly on by itself. It leaves
with the speed the posed swing gave it, so it rolls out and cracks with physics. On a server, the
same settings go in `physics` (`Player.setPhysicalProp`) and in a key's `prop`
(`Player.setPoseAnimation`):

```js
// A whip whose lash starts posed in a curl and lets go after half a second.
player.setPhysicalProp({ kind: 'whip', physics: { shape: 1, curl: 150 } });
player.setPoseAnimation({
  keys: [
    { time: 0, pose: {}, prop: { shape: 1, curl: 150, sway: 0 } },
    { time: 0.3, pose: {}, prop: { shape: 1, curl: -60, sway: 20 } },
    { time: 0.5, pose: {}, prop: { shape: 0 } },
  ],
  loop: false,
});
```

```lua
-- A whip whose lash starts posed in a curl and lets go after half a second.
player:setPhysicalProp({ kind = "whip", physics = { shape = 1, curl = 150 } })
player:setPoseAnimation({
  keys = {
    { time = 0, pose = {}, prop = { shape = 1, curl = 150, sway = 0 } },
    { time = 0.3, pose = {}, prop = { shape = 1, curl = -60, sway = 20 } },
    { time = 0.5, pose = {}, prop = { shape = 0 } },
  },
  loop = false,
})
```

```cs
// A whip whose lash starts posed in a curl and lets go after half a second.
player.SetPhysicalProp(new { kind = "whip", physics = new { shape = 1, curl = 150 } });
player.SetPoseAnimation(new
{
    keys = new object[]
    {
        new { time = 0, pose = new { }, prop = new { shape = 1, curl = 150, sway = 0 } },
        new { time = 0.3, pose = new { }, prop = new { shape = 1, curl = -60, sway = 20 } },
        new { time = 0.5, pose = new { }, prop = new { shape = 0 } },
    },
    loop = false,
});
```

#### The lash posed by hand: LASH KEYS

A whip's Physics panel has **LASH KEYS**: you animate the lash yourself, frame by frame, as
you want it, with no gravity or physics.

1. Put the playhead where you want the lash posed and press **Lash key**: the lash is keyed as
   it shows there (a key holds its whole shape, 12 pieces).
2. Shape it with **Aim side** and **Aim up** (the whole lash turns toward the body's right or
   up), **Curl** (it bends up toward the tip, down with <), **Sway** (it bends toward the
   right toward the tip), **Sag** (its tip sinks) and **Tip twist** (its last third turns);
   each press of < or > is one step and one undo step.
3. Key more frames. Between keys each piece moves smoothly through the keys' shapes, round the
   loop when the animation loops.

**From the lash** makes keys every 0.1 s of the lash as it shows (a Lash from motion), so you
only change the frames that need it. **Remove** takes out the key at the playhead, **Clear** all
of them. The keys show on the timeline's **Lash** row; a press on one moves the playhead to it.
The lash plays exactly as keyed, here and on every player: its track is baked of the keys at 40
frames a second in the body's frame. Over the MCP, `studio_lash_pose` keys, shapes, sets,
removes and clears them.

#### The lash from the hand's motion: Lash from motion

**Lash from motion** (a whip's Physics, under RECORD INTO KEYS) gives the animation a lash that
moves the way a whip does, as frames at its own times.

The animation plays from its start while the studio records where the whip's handle is (its tip,
where it points and turns), 30 times a second: two rounds of a looping animation (when they fit in
10 s), one otherwise. When the game draws fewer frames than that, a pose recorded twice is taken
once, at the time the game drew it. A loop's rounds are then fitted as one smooth loop that keeps
the hand's moves up to 8 times a second, so a frame the game drew late does not make the hand
seem to jump.

By default the lash is **designed** on that path, the way a whip is animated by hand:

- The studio finds the throw in the hand's path: the fast move that stops hard (a wind-up flows on
  into the throw, a throw stops), from when the hand speeds up to when it stops, and which way it
  goes. Over the MCP, `throw: { at }` sets its fastest moment instead.
- During the throw a turn runs along the lash from the hand to the tip, faster toward the tip.
  Each piece swings from the way it had over to the throw's way, one after another: the turned
  pieces are the part along the throw, the ones turning are the loop, the rest is as it was.
  Only the pieces in the turn move against each other, so the lash rolls smoothly and never
  snaps into its next shape. When the turn reaches the tip, the lash is straight along the
  throw: the crack.
- The rest of the time the lash follows the hand like a rope drawn through the air: it trails
  behind the hand, droops tip first after the crack and lies on the floor. It has no swing of
  its own, so it
  cannot whirl round the hand, bounce or ball up.

Over the MCP, `studio_lash_motion` answers the throw it found (`lastLash.throw`: start, end,
the hand's top speed, its way) and the numbers it used (`lastLash.design`). Its `design` option
changes them:

| Number | Default | Does |
| ------ | ------- | ---- |
| `turnSeconds` | 0.1 | How long each piece takes to swing over to the throw (s); the loop's size |
| `rollAfter` | 0.1 | How long after the hand stops the turn reaches the tip (s) |
| `rollPower` | 1.6 | How the turn speeds up along the lash: 1 evenly, more is faster toward the tip |
| `fallSpeed` | 4 | How fast the tip sinks outside the throw (m/s) |
| `rootFallShare` | 0.15 | The share of that the handle's end sinks (between them by the square of how far along) |
| `rootKeep` | 0.15 | How much of the handle's line the first piece keeps |
| `throwShare` | 0.35 | The share of the hand's top speed that starts and ends the throw |
| `stopSeconds` | 0.2 | How soon after its fastest moment a throw must stop |

With `mode: "simulated"` the lash is simulated instead:

1. The studio simulates the lash on the hand's path, away from the game: 24 pieces of exactly
   the lash's length, stiff near the handle and loose at the tip, lighter toward the tip (so the
   wave from the hand speeds up along it into a crack), with gravity, the air's drag and the
   floor. A looping animation goes round four times first, so the lash moves as it would after
   many rounds; the end of the last round is blended into its start, so the loop has no seam.

Either way the lash's shape is kept as the animation's **lash track**: 40 frames a second, each
frame the 12 segments' ways from the handle's tip in the handle's own frame. It takes the place of
the keys' lash shapes.

A simulated crack is chaotic: the same settings can fold one hand's motion and not another's,
and the lash can whirl round the hand after the crack. That is why the designed lash is the
default. For a simulation the studio
simulates the lash twelve times, with two strengths of the leather's own friction (which keeps a
wave from ringing up and down the lash), three tip stiffnesses and two air drags. It measures
each one as `studio_lash_check` does and keeps the one with the fewest problems, then the least
time balled up, the straightest roll-out and the gentlest fold.

While the animation plays, the lash takes the track's shapes, here and on every player's game
alike, with no physics. Between two frames each segment's way is blended, never the points, so
the lash is always its whole length. A 2.4 s loop's track is about 8 KB. Over the MCP,
`studio_lash_motion` makes it (its options change gravity, damping, drag, stiffness, the
taper, floor friction and the frames a second), and `studio_lash_check` measures it.

The track is the animation's `lashTrack` in the export, `Player.setPoseAnimation` and
`Animations.define`: `rate` frames a second (1 to 60), the lash's `length` in metres (0.1 to 6) and
`frames`, each `[yaw, pitch, ...]` in degrees for every segment (2 to 16 segments, up to 600
frames). Yaw turns toward the handle's right side, pitch toward its up side. With
`space: "body"` the ways are in the holding body's frame instead (yaw from its forward toward its
right, pitch toward its up): only the lash's start follows the handle's tip, and a turn of the
wrist does not swing the whole lash. A lash from motion is kept so (the studio records the
character's frame with the hand's path); in the handle's frame every fast turn of the wrist swung
all of the lash at once, and it looked as if it teleported.

```js
player.setPoseAnimation({
  keys: [
    { time: 0, pose: {} },
    { time: 0.4, pose: {} },
  ],
  lashTrack: {
    rate: 40,
    length: 1.6,
    frames: [
      [0, -5, 3, -8],
      [2, -6, 5, -9],
    ],
  },
});
```

```lua
player:setPoseAnimation({
  keys = {
    { time = 0, pose = {} },
    { time = 0.4, pose = {} },
  },
  lashTrack = {
    rate = 40,
    length = 1.6,
    frames = {
      { 0, -5, 3, -8 },
      { 2, -6, 5, -9 },
    },
  },
})
```

```cs
player.SetPoseAnimation(new
{
    keys = new object[]
    {
        new { time = 0, pose = new { } },
        new { time = 0.4, pose = new { } },
    },
    lashTrack = new
    {
        rate = 40,
        length = 1.6,
        frames = new[]
        {
            new[] { 0, -5, 3, -8 },
            new[] { 2, -6, 5, -9 },
        },
    },
});
```

`studio_lash_check` (and `studio_check_animation`'s problems) tell what is wrong with a lash, however it is shaped:

| Measure | Says |
| ------- | ---- |
| `length` | The shortest and longest the lash is as it plays; off by more than 5 % means it shrinks or stretches between its shapes |
| `fold` | Its sharpest bend between two pieces (degrees, when, which piece); over 110° is a kink |
| `crack` | When the tip is fastest, and how sharply its last two pieces turn over then: a whip's crack, not counted as a kink |
| `reach` | Whether it strikes: how straight it rolls out at most (its tip's distance from the hand as a share of its length, and when; under 0.95 means no strike) and `coiled`, the share of the round its tip is within half its length of the hand (over 15 % means it dangles instead of moving with the strike) |
| `jumps` | Frames where the lash flicks out and back at once (when, how far) |
| `snaps` | Frames where the whole lash turns more than 30° on average at once (when, how much): it teleports into its next shape instead of rolling |
| `seam` | How far the loop's end is from its start |
| `fastest` | Its fastest point against the handle (m/s) |

Keys alone are also reported when they are too far apart for a whip: a crack is faster than the
time between two keys.

#### The lash as frames: Lash to keys

**Lash to keys** (a whip's Physics, under RECORD INTO KEYS) makes the lash part of the
animation's keys, frames instead of physics:

1. The animation plays from its start while the physics moves the lash, one round (up to 10 s),
   and the studio records the lash with the handle's place and turn.
2. Each key of the animation keeps the lash's shape at its time: 12 points along it, from the
   handle's tip, in the handle's own frame (metres along it, to its right and toward its up
   side). A looping animation's last key gets the first key's shape, so it goes round without a
   seam. Shape, Curl and Sway keys the animation had are taken out.

From then on, while the animation plays, the lash takes exactly those shapes, blended from key
to key piece by piece (each piece's way and length), here and on every player's game alike: no
gravity, nothing for the game's physics to stretch. Keys are often too far apart for a whip's
fast moves: Lash from motion is the better choice for a strike. Lash to keys takes a lash track out. Over the MCP, `studio_get_animation` shows each key's `lash` and `studio_set_animation`
takes it, so the shapes can be changed by hand. Each key's shape goes in the export with the
animation; a key's `lash` is the same in `Player.setPoseAnimation` and `Animations.define`:

```js
player.setPoseAnimation({
  keys: [
    { time: 0, pose: {}, lash: [[0.1, 0, 0], [0.2, 0, -0.05]] },
    { time: 0.4, pose: {}, lash: [[0.1, 0, 0.02], [0.2, 0, 0.05]] },
  ],
});
```

```lua
player:setPoseAnimation({
  keys = {
    { time = 0, pose = {}, lash = { { 0.1, 0, 0 }, { 0.2, 0, -0.05 } } },
    { time = 0.4, pose = {}, lash = { { 0.1, 0, 0.02 }, { 0.2, 0, 0.05 } } },
  },
})
```

```cs
player.SetPoseAnimation(new
{
    keys = new object[]
    {
        new { time = 0, pose = new { }, lash = new[] { new[] { 0.1, 0, 0 }, new[] { 0.2, 0, -0.05 } } },
        new { time = 0.4, pose = new { }, lash = new[] { new[] { 0.1, 0, 0.02 }, new[] { 0.2, 0, 0.05 } } },
    },
});
```

### Recording the simulation into keys

Below the settings, **RECORD INTO KEYS** turns what the game simulates into keys you keep:

| Detail      | Does                                                                           |
| ----------- | ------------------------------------------------------------------------------ |
| Seconds     | How long a recording runs, 1 to 10 s (3 at first)                              |
| Record      | Records the prop's points 30 times a second for that long; move it meanwhile (the Swing test, its hand, or drag it) |
| Keys        | How many keys the prop keeps and how long they run (shown once there are some) |
| Play keys   | Plays the keys in a loop instead of the simulation; off, it simulates again     |
| Remove keys | Takes the keys away                                                            |

Each recording and each removal is one step you can undo. The project and saves keep the keys
with the prop, until its segments change (the keys fit the rope they were recorded on). An
export carries them as `rope_keys.json` beside the scripts: `{ "rate": 30, "frames": [[x, y,
z, ...], ...] }`, each key every point of the rope in world metres, its start first. A
recording of up to 10 s comes to the editor and plays back whole, however many points the prop
has. Keys of more than about 48 KB (8 s of a whip's lash, say) stay with the prop while the
studio is open, but the kept project, saves and the export leave them out.

## Your own meshes (Static props)

Make a mesh in Blender and export it with `Dev/tools/assets/blender/kcds_export.py`:

```
blender --background my_scene.blend --python Dev/tools/assets/blender/kcds_export.py -- my_mesh
```

The exporter writes `my_mesh.cgf` and `my_mesh.mtl` to `%LOCALAPPDATA%\KCDS\studio\meshes`.
Each material keeps its Base Color; a material whose Base Color comes from an **Image Texture**
node also gets that image as `my_mesh_<material>.dds` (the game's format, at most 1024 pixels a
side), and the mesh is drawn with it. The studio loads these textures with the mesh, and the
export copies them into the resource.
In the studio, open **Props → Static props** in Content. It lists every mesh in that folder;
a new export shows up the next time you open the folder. Press a tile and the mesh stands at the
character's right side, and the Outliner gets a line for it. Its Details show the file,
**Reload** (load the file again after a new export), **Place X/Y/Z**, **Turn X/Y/Z**, **Size**
(steps of 1.25 times) and **Remove**.

**Held in** puts the mesh in the character's right or left hand (in the palm, where the game puts
a weapon) or sets it free beside the character again. Held, it goes with the hand as the character
moves and animates, and **Place** and **Turn** are its place and turn in the hand. Over the MCP,
`studio_place` can hold it in any joint of the character by name (`"Head"`, `"Spine2"`). A held
mesh is not made cloth, and dragging its gizmo sets it free where it is. The export stands a held
mesh beside each player, where a free one stands at first.

Press the mesh in the view to choose it, then drag the gizmo: arrows move it (Select or Move
tool), rings turn it (Rotate), and with the **Scale** tool dragging a handle across sizes it,
twice as big for 200 pixels to the right. Each change is one step you can undo. The project
remembers the shown mesh and where it stands. The mesh is loaded only through the KCDS loader;
it is never copied into the game's folders.

### glTF files without Blender

A `.gltf` or `.glb` file (glTF 2.0, from Blender, another 3D program or a model shop) becomes one
of your meshes with `Dev/tools/assets/gltf/kcds_gltf.py` (Python 3, nothing else), or the MCP
tool `studio_import_gltf`, which then shows it:

```
python Dev/tools/assets/gltf/kcds_gltf.py my_model.glb my_model
```

Every mesh of the file's scene is taken where its nodes put it (glTF's y up becomes the game's z
up), with each material's colour, its base colour texture and its normal map (at most 1024
pixels a side). It writes `my_model.cgf`, `my_model.mtl` and the textures into
`%LOCALAPPDATA%\KCDS\studio\meshes` and answers one JSON line:

```json
{ "name": "my_model", "vertices": 2925, "triangles": 3948, "materials": 2, "textures": 1, "files": ["..."] }
```

Textures may be PNG or JPEG. What it cannot take yet is said, and nothing is written: progressive
or arithmetic-coded JPEGs (`a JPEG texture: a progressive JPEG (save it as PNG or as a baseline
JPEG)`), other picture formats, lines or points, files
older than glTF 2.0, more than 65535 vertices. Skins, skeletons and animations are not read yet:
the mesh comes as a static model.

### Simple props without Blender

A box, a cylinder, a sphere or a flat plane needs no Blender. In the studio, **Static props**
ends with **New box**, **New cylinder**, **New sphere** and **New plane**: a press makes one
(`box_1`, then `box_2`, ...), writes it into `%LOCALAPPDATA%\KCDS\studio\meshes` and shows it
beside the character. Its Details get **SHAPE** rows: its lengths (**Width**, **Depth**,
**Height** or **Radius**, 0.05 m a step), its **Colour** (light grey, wood, iron, stone, red,
blue, green, white, black) and its **Surface**; each change writes it anew and shows it again where
it stands, with its texture. The project remembers each shape's settings.

`Dev/tools/assets/kcds_studio/studio_shapes.py` (Python 3, nothing else) writes the same files from
the command line, and the MCP tool `studio_make_mesh` does it for an AI assistant and then shows
the shape:

```
python Dev/tools/assets/kcds_studio/studio_shapes.py crate "{\"shape\": \"box\", \"size\": [0.6, 0.4, 0.3], \"colour\": [0.45, 0.3, 0.15]}"
```

| Option    | Shapes           | Is                                                                     |
| --------- | ---------------- | ---------------------------------------------------------------------- |
| `shape`   | all              | `box`, `cylinder`, `sphere` or `plane`                                 |
| `size`    | box, plane       | `[width, depth, height]` metres for a box (0.5 each), `[width, depth]` for a plane (1 each) |
| `radius`  | cylinder, sphere | metres, 0.25 at first                                                  |
| `height`  | cylinder         | metres, 0.5 at first                                                   |
| `sides`   | cylinder, sphere | how many sides around, 3 to 128 (24)                                   |
| `rings`   | sphere           | how many rings from top to bottom, 2 to 64 (12)                        |
| `colour`  | all              | `[r, g, b]`, each 0 to 1 (0.8 grey)                                    |
| `surface` | all              | how it sounds and what it does when hit, one of the game's surface types: `mat_wood` (at first), `mat_wood_soft`, `mat_metal`, `mat_rock`, `mat_plaster`, `mat_ceramics`, `mat_glass`, `mat_fabric`, `mat_leather`, `mat_sack`, `mat_thatch`, `mat_wicker`, `mat_soil`, `mat_default` |

Lengths are 0.01 to 50 metres. Each shape stands on the ground about its middle, and every side's
picture runs once across it: put one of your textures on it (Textures, or `studio_texture`) with
the tiling you want. It answers with one JSON line:

```json
{ "name": "crate", "shape": "box", "vertices": 24, "triangles": 12, "files": ["...\\crate.cgf", "...\\crate.mtl"] }
```

A wrong option is answered with what is wrong, for example
`{ "error": "radius is from 0.01 to 50 metres" }`, and nothing is written. Made again under the
same name while it is shown, the mesh loads anew where it stands (as **Reload** does), with its
texture.

### Your mesh's body (Physics mode)

Choose your mesh and open **Physics**: its Details start with **BODY**.

| Detail      | Does                                                                                   |
| ----------- | -------------------------------------------------------------------------------------- |
| Body        | **None** (at first): it stands as placed and nothing hits it. **Solid**: it stands and the character and other things hit it. **Falls**: a rigid body that falls from where it is placed and comes to rest on what is below |
| Body weight | A falling body's weight, 0.1 to 200 kg, steps of 1.25 times (5 kg at first)             |

A falling mesh falls each time it is placed: move it up with **Place Z** or the gizmo and it drops
again from there. A body needs the mesh's collision: the game's own objects (Game props) bring
theirs, and a mesh of yours (from Blender or `studio_shapes.py`) gets one made of its own
triangles when it is given a body. Should the game find no mass in it, the body goes back to
**None** and the status line says `it has no collision of its own, so it cannot fall`. A body makes a cloth mesh solid, and cloth takes the body away; a mesh held in a
joint has no body while it is held. Each change is one step you can undo, and the project
remembers it. The body lives in the studio only: an export carries your mesh as a solid model.

### Your mesh as cloth (Physics mode)

Below **BODY** come the **CLOTH** details.

| Detail     | Does                                                                                |
| ---------- | ----------------------------------------------------------------------------------- |
| Cloth      | On: the game simulates the mesh as cloth (its own soft-body physics); off: solid again |
| Weight     | The cloth's weight, 0.25 to 10 kg in steps of 0.25 (1 kg at first)                   |
| Hangs from | Where it is held: its highest vertices, where they stand when it is made cloth       |

The cloth is made of exactly the vertices and triangles of your mesh, so it bends where the
mesh has vertices: a plane needs subdivisions (a 12 × 12 grid moves well, a single quad cannot
bend). The vertices highest in the world (within 1 cm of the top) stay where they are, and the
rest hangs from them and swings. Turn the mesh to choose the edge: a plane turned 90° about X
stands up and hangs from its top edge; turned 45° it swings down from it. A mesh lying flat has
all its vertices at the top, so it stays still.

Moving, turning or sizing a cloth makes it solid while you move it; half a second after the last
change it is cloth again at its new place. Each change (on, off, weight) is one step you can
undo, and the project remembers it. The cloth lives in the studio only: an export carries your
mesh as a solid model.

### Your skinned mesh on the character

A skinned mesh is worn by the character and moves with its bones, as the game's own clothes do.
It is a `.skin` file, the game's own format for clothes and bodies, bound to the character's
skeleton. `Dev/tools/assets/blender/kcds_skin.py` writes one from a mesh whose vertices carry
bone weights. It takes the skeleton from one of the game's own skins in your installation (a
male torso piece carries all 133 bones of the male skeleton), so your mesh binds to the same
bones in the same rest pose. Each vertex keeps its four heaviest bones, and a bone the skeleton
does not have is refused by name.

Put `<name>.skin` and its `<name>.mtl` in `%LOCALAPPDATA%\KCDS\studio\meshes\`. Then the MCP tool
`studio_wear_skin` with `{ "name": "<name>" }` puts it on the character, and `{ "name": "" }`
takes it off. Wearing it again loads the file anew, so a changed skin shows its changes.

From a glTF file the MCP tool `studio_import_gltf_skin` does both steps: it makes `<name>.skin`
from the file's skinned meshes and the character wears it. No Blender is needed:

```json
{ "path": "C:\\models\\sleeve.glb", "name": "sleeve" }
```

The file's joints match the game's bones by name, in any case and without a rig's prefix
(`mixamorig:`, `DEF-`, ...). `map` names the rest, `{ "<game bone>": "<glTF joint>" }`. A joint
that matches no bone gives its weight to its nearest matching parent, and a weighted joint with
no matching parent is refused by name. The answer tells you what matched (`mapped`) and how far
the largest vertex moved onto the game's bones (`moved`, in metres). A rig made on the game's
skeleton binds exactly, with `moved` near 0. Another rig's proportions are taken over as far as
the weights blend them, but the turns of its rest pose are not, so model the mesh in the game's
rest pose (the arms angled down) for the best fit. The tool reads the skeleton from the game's `Data/Characters.pak`; the
`KCDS_GAME_DIR` environment variable names another installation.

### FBX files

The MCP tool `studio_import_fbx` takes an FBX file as a mesh beside the character (`"as":
"mesh"`, as `studio_import_gltf`) or as a skin the character wears (`"as": "skin"`, as
`studio_import_gltf_skin`, with the same `map`). FBX is a closed format, so the tool needs
Blender installed on your machine: Blender reads the file headless, with its factory settings
and none of your add-ons, and gives glTF, which the glTF import turns into the game's files. The
newest Blender in `Program Files\Blender Foundation` is used; `KCDS_BLENDER` names `blender.exe`
elsewhere.

## The game's own objects and clips (Game props, Game clips)

Content also lists the game's own files, everything in its packs (`Data\*.pak`):

- **Props → Game props**: every object model, `Objects/....cgf` (about 11,000 in the game's
  1.5.5; their `_lod` models are left out).
- **Animation → Game clips**: every animation clip, `Animations/....caf` (about 3,000),
  including the game's motion capture of its cutscenes.

The first time one of these folders opens, the studio reads the packs' file lists (a few
seconds; the folder says **Reading the game** until then). Each tile shows a file's name, with
its folder below. **Found** shows how many files match, and the pager turns the pages.

Press the search field (**Search the game’s files**) and type: letters, digits, space, and `-`
for `_`. Each key searches again. A file matches when its path has every word you typed, in any
case: `horse saddle` finds every saddle under the horse objects. **Enter** or **Esc** stops
typing.

Pressing a **Game props** tile shows that object beside the character, in place of your mesh,
with the game's own material. You place, turn and size it as you would your mesh, the project
remembers it, and **Export** names it by its game path, so the server puts the game's object
beside each player and the resource has no model files. Cloth and your textures are for your own
meshes only.

Pressing a **Game clips** tile plays that clip on the character in a loop. Press it again to stop
it. A prop in the hand follows the hand, so a whip's lash swings with a game clip's real motion
(for example `whipping executioner`).

While a game clip plays, Animation mode's Details start with **GAME CLIP**:

| Detail     | Does                                                                                     |
| ---------- | ---------------------------------------------------------------------------------------- |
| Time       | Where the clip is now, and its whole length (measured while it plays)                    |
| From, To   | Only this part of the clip plays, in steps of a quarter of a second                     |
| Part       | **From here** and **To here** set From or To to where the clip is now                   |
| Whole clip | All of it plays again                                                                    |
| Top body only | Only the upper body plays the clip: the legs keep the character's own stance or walk, so a player walks while it plays |
| Bake       | **Bake to keys** makes the part playing your own animation (below): the whole body, legs included |
| In place   | On at first: the character stays where it stands, as a cutscene's walking and turning would take it across the stage |
| Loop       | Again and again, or once (it holds its last pose)                                         |
| Stop       | Stops the clip; the character goes back where it stood                                   |

A cutscene's clip is a whole scene: the executioner's whipping (31.6 s) walks in, stands and only
then strikes. Its strikes are from about 19.4 s to 27.9 s, so From 19.4 s and To 27.9 s loop the
whipping alone. The studio's world turns the game's look IK off (`ca_UseLookIK 0`): it turned
the character's head toward its own look target on top of the clip and flipped it over the
shoulder now and then.

### Baking a game clip into your animation

**Bake to keys** records one round of the part playing (up to 30 frames a second) and makes it
the studio's own animation, which plays on players like every studio animation does
(`Animations.define`, `player.playAnimation(name)`), so you can change it as any other:

- Every joint of the body is baked, legs and feet included, as a turn on top of the character's
  own pose before the clip. Joints that hardly move are left out, at most 32 joints.
- Only a loop is kept: the stretch of 1.2 to 4 s that ends the way it starts, moving the same
  way, with the hips not wandering and the legs stepping least. A cutscene's walking in and out
  stays out. What is left between its end and its start is spread over the loop, so it goes
  round without a seam.
- The skeleton's root is not baked, nor the hips' slow turning: the body stays where it stands
  and faces forward, and a strike's twist stays. How far the hips go down or up (bent knees)
  is each key's **lift**, so the feet stay on the ground.
- At most 32 keys, a spline between them.

Details' message says what was made, for example `Baked: 30 keys, 25 joints, the loop 2.40 s
(2.07–4.47 s of the part)`. The game clip stops, and the animation plays. A baked animation
is longer than a player's field takes: an export defines it, and every player's game gets it
once by its name.

## Your own textures (Textures)

Put pictures in `%LOCALAPPDATA%\KCDS\studio\textures`: `.png` or `.jpg` (the studio makes them the
game's texture itself, at most 2048 pixels a side, with every smaller level for distance), or a
`.dds` as it is. Press **Textures** (mode 4): Content lists them. With your mesh shown (Static
props), press a texture and the mesh is drawn with it. The mesh's Details then show **Texture**
(‹ › steps through your textures and **Own material**, the mesh's own) and **Tiling** (how many
times the texture repeats across the mesh, halved or doubled a step, 1 at first).

Each change is one step you can undo; the project keeps the texture, and **Reload** puts it on
again. The texture is loaded only through the KCDS loader, from memory; nothing is written to
the game's folders.

A **normal map** goes with a texture when it lies beside it named `<texture>_normal` (for
`bricks.png`, `bricks_normal.png`, `.jpg` or `.dds`): the mesh is lit as its bumps say. It is not
listed as a texture of its own, and the export carries it too. From Blender, a Normal Map node on
the material's Normal input, fed by an Image Texture, is exported as `<mesh>_<material>_normal.dds`.

A picture's colours are marked as sRGB in the texture the studio makes (as the game's own
textures are), so they look as in a picture viewer; a normal map is not.

A **specular map** goes with a texture the same way, named `<texture>_spec` (white where the
surface shines, black where it is dull); the export carries it too.

## The character's skeleton (Bones)

Press **Bones** (mode 5) in the toolbar. The studio draws the character's skeleton over the
body every frame: a line from each joint to its parent, a small ring at each joint. The lines
are drawn over the scene, so you can see joints inside the body. The skeleton's root stands on
the floor and holds helper joints (IK targets, face poses); no lines are drawn from the root, so
the body's bones stay readable.

The outliner lists every joint of the character's skeleton (the male body has 493), in the
skeleton's order, with its parent's name in the Type column. It shows them a page at a time:
use `<<` and `>>` under the list. Choose a joint in the outliner, or press its ring in the view.
The chosen joint and the part of the body it moves (every joint below it) are drawn thick in
orange, with the joint's name. The bones are drawn only inside the view, never over the panels.
Leave Bones by choosing another mode; the skeleton is no longer drawn.

### Posing a joint

The chosen joint has a turning gizmo: a ring about each of its parent's axes, **red X**,
**green Y** and **blue Z**. Press a ring and drag across to turn the joint about that axis (half
a degree a pixel); letting go is one step you can undo. The turn goes into the key at the
playhead, or a new key there.

Details shows the chosen joint's name and parent (press the parent to choose it) and its turn:
**Turn X**, **Turn Y** and **Turn Z** turn it 5° at a time about its parent's axes, on top of the
body standing still. **Reset** gives the joint back the body's own pose. **Idle animation** lets
the body's own idle play under your turns.

### An animation from a glTF file

The MCP tool `studio_import_gltf_animation` puts an animation of a glTF 2.0 file (`.gltf` or
`.glb` with a skeleton: from Mixamo, Blender or another program) on the studio's character and
plays it. Its joints match the game's by name, in any case and without a rig's prefix such as
`mixamorig:`: the game's skeleton has `Hips`, `Spine`, `Spine1`, `Spine2`, `Neck`, `Head`,
`RightShoulder`, `RightArm`, `RightForeArm`, `RightHand`, the same on the left, `RightUpLeg` and
so on. `map` adds or changes pairs, `{ "RightArm": "upperarm_r" }`.

Each game bone is first laid along the file's bone as the file's rig rests (a file resting in a
T-pose gives T-pose arms), then turns as the file's joint turns; the hips' rise becomes each
key's lift. It answers what matched, the file's joints and the game's that did not, and the
joints left out:

```json
{ "keys": 11, "mapped": { "Hips": "Hips", "RightArm": "RightArm", "...": "..." }, "unmappedGltf": [], "unmappedGame": ["..."], "leftOut": [] }
```

A player's animation takes at most 32 keys and 32 joints: the keys are spread over the file's
animation (`rate`, 10 a second at first) and the body's joints come before the fingers and toes,
those that move most first; joints that do not move are left out. Measuring the skeleton stands
the body still for a moment, so the import is two undo steps. `kcds_gltf_anim.py` in
`Dev/tools/assets/gltf` does the conversion and can be run by hand.

### Animating: the timeline

In Bones mode the content panel is a timeline, as in a video editor. In every other mode the
same timeline is the panel's **Timeline** tab (between Content and History), so Play, the
playhead and the keys are at hand while you set up props, physics or prop keys; **Space** plays
or pauses there as in Bones mode.

- **On the left:** to the start, play or pause, to the end, the time, **+ Key** (a key at the
  playhead with the pose there), **− Key** (the key at the playhead), **Loop**, and **Blend**:
  - *Spline*, the default: a smooth curve through every key, best for swings;
  - *Smooth*: eased into and out of each key;
  - *Linear*: an even speed.
- **The ruler** shows seconds and the playhead. Press the ruler or a track to put the playhead
  there: the character shows the animation at that moment.
- **The tracks:** *All keys*, then one track for each turned joint, each key a diamond. Press a
  diamond to choose that key and joint. Drag a diamond to move its key to another time.

Turning a joint where there is no key adds one there (auto key). **Space** plays and pauses.
The first key stays at 0 s. An animation can be up to 60 s with 64 keys. Undo covers every
change, and the project keeps the animation.

### The prop along the animation: prop keys

With a rope, a chain or a whip in the scene, Bones mode's Details end with **PROP AT THIS KEY**.
Put the playhead on a key and switch **Prop key** on: that key now sets the prop, starting from
the prop as it is there. Then step what it sets:

| Detail | Does |
| ------ | ---- |
| Hand | The hand that holds the prop from this key on (Right hand, Left hand) |
| Grip angle, Grip twist | A whip's grip from this key on |
| Gravity, Damping, Weight, Air drag, Bend limit, Stiffness | How the rope or lash moves from this key on |
| Collide | On, Floor only or Off from this key on |

Numbers blend from one prop key to the next that sets them (smoothly, or evenly with the
*Linear* blend); the hand and Collide switch at their key. Before the first prop key, and with
none, the prop keeps its own settings (Props and Physics mode). So a whip can lie loose in the
hand while the arm swings back, and snap into line with the forearm for the strike. While the
animation plays, and wherever you put the playhead, the prop shows its settings at that time.
Each change is one step you can undo, and the project, saves and the export keep the prop
keys.

A whip that cracks well in the studio: **Collide** on *Floor only* (so it swings past the head
and shoulders instead of catching on them) and a **Grip angle** near 20° (the handle in line
with the forearm): throw the lash back over the shoulder, wait a moment, then strike forward and
stop the hand at shoulder height.

### Your mesh along the animation: mesh keys

A key can also place your mesh. Put the playhead on a key, place the mesh where it should be
there (Details, the gizmo or `studio_place`), and turn on **Mesh key** under **MESH AT THIS KEY**
in Bones mode's Details: that key keeps the mesh's place and turn (**Place**, **Turn**; **Set
again** keeps where it stands now). Do the same on another key; between mesh keys the mesh moves
and turns as the animation blends (linear or eased), before the first it stands at the first key's
place and after the last at the last key's. Without mesh keys it stands where it was placed. Each
change is one undo step, and the project keeps the keys.

Over the MCP, `studio_mesh_key` turns the key at the playhead's mesh key on or off (and without
`on` answers the mesh keys), and `studio_set_animation` takes a key's
`mesh: { "at": [x, y, z], "turn": [x, y, z] }`.

An export with an animation carries the mesh keys too: the server moves each player's mesh along
them ten times a second (`Prop.setPose`), from where the player stood when they got it, in time
with the animation they play. A mesh held in a joint goes with the hand instead and keeps no mesh
keys in the export.

### Effects and sounds in an animation (Animation mode)

Press **Animation** (mode 3). Content lists **Effects** (the game's particle effects: a few that
suit animations first, such as `blacksmith_hit` sparks and `sweep` dust, then the whole
catalogue by pages) and **Sounds** (game sound triggers). Set the playhead (Details: **Time**,
or the timeline in Bones mode), choose **At joint** (RightHand, LeftHand, Head, Spine3,
RightFoot, LeftFoot) and **Size**, then press an effect or a sound: it is added at the playhead
and plays once at once. While the animation plays, each event plays when the playhead passes it,
every loop. Details list the events (up to 32) by time; press one to remove it. Each change is
one step you can undo, and the project, saves and exports keep the events.

On a server, an exported animation's effect events play on the player's body for everyone
within 250 m (`Animations.define`, below). Its sound events play only in the studio.

The studio's empty level loads none of the game's effect libraries when it starts, so the studio
lets the game load each library the first time one of its effects plays
(`e_ParticlesAllowRuntimeLoad`). The first play of an effect from a library you haven't used yet
may come a moment late while the library loads.

## Exporting a prop as a resource

With a rope or a whip chosen, press **Export** in Details (under RESOURCE). The studio writes a
server resource named `kcds_studio_<kind>` to `%LOCALAPPDATA%\KCDS\studio\exports\<name>\`. It
has a `package.json` and a `server.js` that gives every player who joins the prop with the
settings you made: its segments and length, the hand it is held in, a whip's grip and how it
moves (Physics mode). So it sits in the player's hand as it sat in the character's, with nothing
to turn again. Copy the folder into your server's resources and start it there:

```js
'use strict';
// Exported from the Asset/Animation Studio: every player who joins holds this prop in
// the right hand (Player.setPhysicalProp).
const PROP = {"kind":"whip","segments":20,"length":1.7,"hand":"RightHand","grip":{"pitch":20,"twist":0},"physics":{"gravity":9.8,"damping":0.1,"mass":0.3,"air":0.02,"bend":0,"stiffness":0,"collide":2}};

kcds.core.Events.on('playerReady', (player) => {
  player.setPhysicalProp(PROP);
});
```

```lua
-- The export's server.js in Lua, for a resource of yours written in Lua.
local PROP = {
  kind = "whip",
  segments = 20,
  length = 1.7,
  hand = "RightHand",
  grip = { pitch = 20, twist = 0 },
  physics = { gravity = 9.8, damping = 0.1, mass = 0.3, air = 0.02, bend = 0, stiffness = 0, collide = 2 },
}

kcds.core.Events.on("playerReady", function(player)
  player:setPhysicalProp(PROP)
end)
```

```cs
// The export's server.js in C#, for a resource of yours written in C#.
public class Main : Script
{
    private static readonly object Prop = new
    {
        kind = "whip",
        segments = 20,
        length = 1.7,
        hand = "RightHand",
        grip = new { pitch = 20, twist = 0 },
        physics = new { gravity = 9.8, damping = 0.1, mass = 0.3, air = 0.02, bend = 0, stiffness = 0, collide = 2 },
    };

    public override void OnStart()
    {
        Events.On("playerReady", (Player player) =>
        {
            player.SetPhysicalProp(Prop);
        });
    }
}
```

With an animation in the scene, the export carries it too, with its prop keys (below).

With a game clip playing, every player who joins plays its part again and again (from From to
To, `player.playAnimation('', { clip, time })`), with the whole body and without walking or
turning (`lockMovement`, `lockRotation`), or with **Top body only** on the upper body while they
walk (`upperBody: true`). A `client.js` shows each player themselves from behind through the
game's third-person camera while it plays (**V**: their own eyes, and back).

An animation of the whole body (one not **Top body only**, such as a baked game clip) plays
with the player held where they stand, as a whole-body clip does, and with the same `client.js`
view from behind:

```js
kcds.characters.Animations.define('kcds_studio_animation', ANIMATION);

function give(player) {
  player.setPhysicalProp(PROP);
  // The whole body plays it: the player does not walk or turn it meanwhile.
  player.playAnimation('kcds_studio_animation', { lockMovement: true });
}
```

```lua
kcds.characters.Animations.define("kcds_studio_animation", ANIMATION)

local function give(player)
  player:setPhysicalProp(PROP)
  -- The whole body plays it: the player does not walk or turn it meanwhile.
  player:playAnimation("kcds_studio_animation", { lockMovement = true })
end
```

```cs
Animations.Define("kcds_studio_animation", animation);

void Give(Player player)
{
    player.SetPhysicalProp(prop);
    // The whole body plays it: the player does not walk or turn it meanwhile.
    player.PlayAnimation("kcds_studio_animation", new { lockMovement = true });
}
```

With **Top body only** the export keeps only the joints from the spine up and leaves the lift
out, and the player walks while it plays.

### Your mesh in the export

With your mesh shown (Static props), **Export** also puts it in the resource, named
`kcds_studio_mesh` when the scene has no prop and no animation:

```text
kcds_studio_mesh/
  package.json
  server.js
  stream/objects/kcds/kcds_studio_mesh/suzanne.cgf      your mesh, drawn with the material below
  stream/objects/kcds/kcds_studio_mesh/suzanne.mtl      its material (with a texture: one that shows it)
  stream/objects/kcds/kcds_studio_mesh/suzanne_tex.dds  the texture on it, made the game's .dds
```

Players get these files through the resource download, as any resource's `stream/` files (see
[Sounds and game files](audio-and-assets.md)), and their game loads them from memory. The
`server.js` spawns the mesh beside each player who joins, as far from them as it stood from the
character in the studio, turned and sized as there, and takes it away when they leave:

```js
const MESH = 'objects/kcds/kcds_studio_mesh/suzanne.cgf';
const MESH_PLACE = { offset: [0.64, -0.64, 1], turn: [0, 0, 0], scale: 1 };
const meshes = new Map();

kcds.core.Events.on('playerReady', (player) => {
  const { x, y, z } = player.position;
  const [dx, dy, dz] = MESH_PLACE.offset;
  const at = new kcds.math.Vector3(x + dx, y + dy, z + dz);
  const [rx, ry, rz] = MESH_PLACE.turn;
  const turn = new kcds.math.Vector3(rx, ry, rz);
  meshes.set(player.id, kcds.world.Prop.spawn(MESH, at, turn, MESH_PLACE.scale));
});

kcds.core.Events.on('playerDisconnect', (player) => {
  meshes.get(player.id)?.destroy();
  meshes.delete(player.id);
});
```

```lua
local MESH = "objects/kcds/kcds_studio_mesh/suzanne.cgf"
local MESH_PLACE = { offset = { 0.64, -0.64, 1 }, turn = { 0, 0, 0 }, scale = 1 }
local meshes = {}

kcds.core.Events.on("playerReady", function(player)
  local position = player.position
  local offset, turn = MESH_PLACE.offset, MESH_PLACE.turn
  local at = kcds.math.Vector3.new(position.x + offset[1], position.y + offset[2], position.z + offset[3])
  local turned = kcds.math.Vector3.new(turn[1], turn[2], turn[3])
  meshes[player.id] = kcds.world.Prop.spawn(MESH, at, turned, MESH_PLACE.scale)
end)

kcds.core.Events.on("playerDisconnect", function(player)
  local mesh = meshes[player.id]
  if mesh ~= nil then
    mesh:destroy()
  end
  meshes[player.id] = nil
end)
```

```cs
public class Main : Script
{
    private const string Mesh = "objects/kcds/kcds_studio_mesh/suzanne.cgf";
    private static readonly double[] Offset = { 0.64, -0.64, 1 };
    private readonly Dictionary<double, Prop> meshes = new();

    public override void OnStart()
    {
        Events.On("playerReady", (Player player) =>
        {
            var position = player.Position;
            var at = new Vector3(position.X + Offset[0], position.Y + Offset[1], position.Z + Offset[2]);
            meshes[player.Id] = Prop.Spawn(Mesh, at, new Vector3(0, 0, 0), 1);
        });

        Events.On("playerDisconnect", (Player player) =>
        {
            if (meshes.Remove(player.Id, out var mesh))
            {
                mesh.Destroy();
            }
        });
    }
}
```

Spawn it anywhere else with `kcds.world.Prop.spawn(MESH, position)`.

## Giving players a rope or a whip on your server

A server script can put the studio's props in a player's hand with
`player.setPhysicalProp`. Every player sees it, including players who join later, until it is
set to `null` or the player leaves.

```js
player.setPhysicalProp({ kind: 'whip' }); // the game's whip handle with a 1.6 m lash
player.setPhysicalProp({ kind: 'rope', segments: 24, length: 2.5 });
player.setPhysicalProp({
  kind: 'whip',
  hand: 'LeftHand',
  grip: { pitch: 20 },
  physics: { damping: 0.1, collide: 2 },
});
player.setPhysicalProp(null); // takes it away
console.log(player.physicalProp.grip); // { pitch: 20, twist: 0 }
```

```lua
player:setPhysicalProp({ kind = "whip" }) -- the game's whip handle with a 1.6 m lash
player:setPhysicalProp({ kind = "rope", segments = 24, length = 2.5 })
player:setPhysicalProp({
  kind = "whip",
  hand = "LeftHand",
  grip = { pitch = 20 },
  physics = { damping = 0.1, collide = 2 },
})
player:setPhysicalProp(null) -- takes it away
print(player.physicalProp.grip.pitch) -- 20
```

```cs
player.SetPhysicalProp(new { kind = "whip" }); // the game's whip handle with a 1.6 m lash
player.SetPhysicalProp(new { kind = "rope", segments = 24, length = 2.5 });
player.SetPhysicalProp(new
{
    kind = "whip",
    hand = "LeftHand",
    grip = new { pitch = 20 },
    physics = new { damping = 0.1, collide = 2 },
});
player.SetPhysicalProp(Sdk.Null); // takes it away
Log.Info(player.PhysicalProp["grip"].ToJson()); // {"pitch":20,"twist":0}
```

`player.physicalProp` gives back a copy with every setting:
`{ kind, segments, length, hand, grip: { pitch, twist }, physics: { gravity, damping, mass, air,
bend, stiffness, collide } }`, or `null`.

| Option     | Values                       | Default                    |
| ---------- | ---------------------------- | -------------------------- |
| `kind`     | `'rope'`, `'whip'` or `'chain'` | (needed)                |
| `segments` | a whole number from 2 to 64  | 16 for a rope, 20 for a whip, 24 for a chain |
| `length`   | 0.2 to 20 metres (to a cm)   | 1.2 for a rope, 1.6 for a whip, 1.5 for a chain |
| `hand`     | `'RightHand'` or `'LeftHand'` | `'RightHand'`             |
| `grip`     | a whip's `{ pitch: 0 to 180, twist: -180 to 180 }` degrees (the studio's Grip angle and twist) | `{ pitch: 90, twist: 0 }` |
| `physics`  | `{ gravity: 0 to 30, damping: 0 to 5, mass: 0.05 to 10, air: 0 to 2, bend: 0 to 90, stiffness: 0 to 500, collide: 0, 1 or 2 }` (the studio's Physics mode) | `{ gravity: 9.8, damping: 0.3, mass: 0.5, air: 0.1, bend: 0, stiffness: 0, collide: 1 }` |

Leave out any part of `grip` or `physics` and it keeps its default. An unknown kind, hand or
setting, or options that are not an object, throw a `TypeError`. Segments, a length or a setting
out of range throw a `RangeError`. A whip's segments, length and physics are its lash's.

Each client builds the prop itself with the game's rope physics and ties its start to that
player's hand. The hand moves with the player's synchronised animation, so the prop goes
wherever the player goes. The rest of the rope swings by each client's own physics. Its exact
curve is not sent, so it can look slightly different on two machines. A studio animation's prop
keys (below) change the hand, grip and physics while it plays.

## Playing a studio animation on players

**Export** in Bones mode's Details (or under RESOURCE with a prop) writes the animation into the
resource too: a scene with a prop exports as `kcds_studio_<kind>` with both, an animation alone
as `kcds_studio_animation`. Its `server.js` gives the animation the name `kcds_studio_animation`
on the server and plays it on every player who joins:

```js
'use strict';
// Exported from the Asset/Animation Studio: every player who joins plays the animation
// kcds_studio_animation (player.playAnimation).
const ANIMATION = {
  keys: [
    { time: 0, pose: { RightArm: [40, 0, 0] } },
    { time: 0.6, pose: { RightArm: [100, 0, 0], RightForeArm: [0, 0, 60] } },
    { time: 1.2, pose: { RightArm: [40, 0, 0] } },
  ],
  blend: 'spline',
  loop: true,
};

// Any resource on the server plays it by name, as one of the game's own:
// player.playAnimation('kcds_studio_animation'), player.stopAnimation().
kcds.characters.Animations.define('kcds_studio_animation', ANIMATION);

kcds.core.Events.on('playerReady', (player) => {
  player.playAnimation('kcds_studio_animation');
});
```

```lua
-- The export's server.js in Lua, for a resource of yours written in Lua.
local ANIMATION = {
  keys = {
    { time = 0, pose = { RightArm = { 40, 0, 0 } } },
    { time = 0.6, pose = { RightArm = { 100, 0, 0 }, RightForeArm = { 0, 0, 60 } } },
    { time = 1.2, pose = { RightArm = { 40, 0, 0 } } },
  },
  blend = "spline",
  loop = true,
}

-- Any resource on the server plays it by name, as one of the game's own:
-- player:playAnimation("kcds_studio_animation"), player:stopAnimation().
kcds.characters.Animations.define("kcds_studio_animation", ANIMATION)

kcds.core.Events.on("playerReady", function(player)
  player:playAnimation("kcds_studio_animation")
end)
```

```cs
// The export's server.js in C#, for a resource of yours written in C#.
// A pose is a Dictionary: it keeps the joints' names as written, where an anonymous object's
// names would start lower-case.
public class Main : Script
{
    private static readonly object Animation = new
    {
        keys = new object[]
        {
            new { time = 0, pose = new Dictionary<string, int[]> { ["RightArm"] = new[] { 40, 0, 0 } } },
            new
            {
                time = 0.6,
                pose = new Dictionary<string, int[]>
                {
                    ["RightArm"] = new[] { 100, 0, 0 },
                    ["RightForeArm"] = new[] { 0, 0, 60 },
                },
            },
            new { time = 1.2, pose = new Dictionary<string, int[]> { ["RightArm"] = new[] { 40, 0, 0 } } },
        },
        blend = "spline",
        loop = true,
    };

    public override void OnStart()
    {
        // Any resource on the server plays it by name, as one of the game's own:
        // player.PlayAnimation("kcds_studio_animation"), player.StopAnimation().
        Animations.Define("kcds_studio_animation", Animation);

        Events.On("playerReady", (Player player) =>
        {
            player.PlayAnimation("kcds_studio_animation");
        });
    }
}
```

### By name, like the game's animations

`kcds.characters.Animations.define(name, animation)` names a studio animation for every resource
on the server. Then `player.playAnimation(name)` plays it as it plays one of the game's own:
until `player.stopAnimation()` or the next `playAnimation`, and `playerAnimationEnd(player, name,
reason)` follows with `'stopped'` or `'replaced'`. `playAnimation`'s `loop` option overrides the
animation's own (`player.playAnimation('whip_strike', { loop: false })`). An animation's
`events` (`{ time, kind: 'effect', name: 'Library.group.name', joint?, scale? }`) play on the
body at their times while it plays, again each loop; an effect the game does not have throws a
`RangeError`.
`Animations.defined()` lists the names and `Animations.undefine(name)` forgets one. A name is 1
to 48 letters, digits or `_`; anything else throws a `TypeError`.

### Directly on one player

A server script can also play one with `player.setPoseAnimation`: every player sees it on
that player's body, on top of the body's own animation, including players who join later, until
it is set to `null` or the player leaves. `player.poseAnimation` gives it back (a copy).

| Option  | Values                                                                 | Default  |
| ------- | ---------------------------------------------------------------------- | -------- |
| `keys`  | 1 to 32 keys `{ time, pose: { <joint name>: [x, y, z] }, prop? }`, the first at 0 s, each later, up to 60 s; up to 32 joints a key | (needed) |
| `blend` | `'spline'`, `'smooth'` or `'linear'`                                    | `'spline'` |
| `loop`  | `true` or `false` (it stays on its last key)                            | `true`   |

Turns are degrees about each joint's parent's axes, as the studio's Turn X, Y and Z. A key's
`prop` is a prop key: it sets the player's physical prop (`setPhysicalProp`) from that key on,
with any of `hand`, `pitch`, `twist`, `gravity`, `damping`, `mass`, `air`, `bend`, `stiffness`
and `collide`, the same values as `setPhysicalProp`'s `hand`, `grip` and `physics`. Numbers
blend toward the next key that sets them, the hand and `collide` switch at the key:

```js
player.setPhysicalProp({ kind: 'whip', physics: { collide: 2 } });
player.setPoseAnimation({
  keys: [
    { time: 0, pose: { RightArm: [-170, 0, 10] }, prop: { pitch: 90 } },
    { time: 0.4, pose: { RightArm: [-110, 0, 5] }, prop: { pitch: 20 } },
  ],
});
```

```lua
player:setPhysicalProp({ kind = "whip", physics = { collide = 2 } })
player:setPoseAnimation({
  keys = {
    { time = 0, pose = { RightArm = { -170, 0, 10 } }, prop = { pitch = 90 } },
    { time = 0.4, pose = { RightArm = { -110, 0, 5 } }, prop = { pitch = 20 } },
  },
})
```

```cs
player.SetPhysicalProp(new { kind = "whip", physics = new { collide = 2 } });
player.SetPoseAnimation(new
{
    keys = new object[]
    {
        // A Dictionary keeps the joint's name as written ("RightArm", not "rightArm").
        new
        {
            time = 0,
            pose = new Dictionary<string, int[]> { ["RightArm"] = new[] { -170, 0, 10 } },
            prop = new { pitch = 90 },
        },
        new
        {
            time = 0.4,
            pose = new Dictionary<string, int[]> { ["RightArm"] = new[] { -110, 0, 5 } },
            prop = new { pitch = 20 },
        },
    },
});
```

The animation is sent as text of up to 2000 bytes. A joint name that is not a name, a turn that
is not three numbers, an unknown prop setting or hand, or options that are not an animation
throw a `TypeError`; too many keys or joints, times that do not start at 0 and rise, a prop
setting out of range, or more than 2000 bytes throw a `RangeError`.

Each client plays the animation from when it got it: a loop is not in step between two
machines. A joint the body does not have is left out.

## Your project

The studio keeps your scene on this machine in
`%LOCALAPPDATA%\KCDS\architect\projects\kcds_studio.json`. The file is written after every
change and rebuilt the next time the studio opens: the rope or the whip with its settings and
where it is held, the animation, your mesh with its place, texture and cloth, the clothes and
the sky.
Undo and Redo cover one session.

### Saves

Press the studio's name in the menu bar (**Asset/Animation Studio ▾**), or the **Saves** tab in
the dock, as in World Architect. Press the name field, type a name (letters, digits, spaces, `-`,
Shift+`-` for `_`; Enter or Escape ends) and press **Save as**: the scene is kept in
`Documents\KCDS\Asset Studio\<name>.json`, replacing a save of that name. Each save shows when it
was written and its size, with **Load** (the save's scene instead of the one now; Undo starts
afresh from it) and **Delete**. The menu bar shows the name of the save last saved or loaded.

## Working with an AI assistant (MCP)

World Architect and the studio share one MCP server, `Dev/tools/architect-mcp/server.cjs`. It
needs Node.js and nothing else. Your AI assistant starts it over stdio, for example with this
MCP configuration:

```json
{ "mcpServers": { "kcds-architect": { "command": "node", "args": ["Dev/tools/architect-mcp/server.cjs"] } } }
```

The studio's **AI** tab, the last tab of the dock as in World Architect, gives these lines ready
to copy: the Claude Code command and the MCP entry with this game's server path, and the token.
Press **Copy** beside a line and paste it into your AI app; the tab says whether an assistant is
connected.

The server reaches the open editor through a named pipe of the game on your own machine,
`\\.\pipe\kcds-editor-<process id>`. There is no network and no server. The game opens the pipe
only in the editor's local mode, and only your own programs can open it. If several KCDS editor
games are running, set `KCDS_EDITOR_PID` to the process id of the one the assistant should use.

| Tool                     | Does                                                             |
| ------------------------ | ---------------------------------------------------------------- |
| `studio_state`           | The studio as it is now: mode, tab and folder, selection, the prop (hold, grip, physics), the mesh, the animation (keys, length, time, playing, prop keys), the game clip playing (part, time, length) and the status message |
| `studio_make_prop`       | Makes a `rope` or a `whip` in the right hand                     |
| `studio_change_prop`     | `segments` or `length` one step up (`1`) or down (`-1`)          |
| `studio_swing_test`      | Turns the swing test on or off                                   |
| `studio_record_keys`     | Records the prop's simulation into keys for 1 to 10 seconds      |
| `studio_get_keys`        | The recorded keys, at most 30 at a time (`first`, `count`): each the prop's points in world metres, to measure a swing (where the lash went, how fast its tip was) |
| `studio_play_keys`       | Plays the recorded keys in a loop, or lets the prop simulate again |
| `studio_lash_keys`       | Lash to keys: the whip's lash recorded along the animation, kept as each key's lash shape (frames, no physics) |
| `studio_lash_motion`     | Lash from motion: the lash simulated on the hand's recorded path and kept as the animation's lash track, 40 frames a second (options: gravity, damping, air, stiffRoot, stiffTip, taper, friction, rate) |
| `studio_lash_check`      | How the lash plays, measured: its length all along, its sharpest fold, the crack, whether it strikes (reach, coiled), single-frame jumps, the loop's seam, its fastest point, and the problems |
| `studio_lash_path`       | The hand's path the last Lash from motion was made on: the handle's tip and frame, 30 a second |
| `studio_lash_pose`       | The lash posed by hand: lash keys at their own times (add, shape by aimSide, aimUp, curl, sway, sag, tipTwist, set exactly, remove, from the lash shown, clear); the lash plays exactly as keyed |
| `studio_lash_live_check` | The lash as the game draws it: recorded while the animation plays and measured in the world (its worst turn of the whole lash from one drawn frame to the next, snaps where all of it turns over 30° at once, frames drawn a second) |
| `studio_patch_animation` | Changes only some keys (by index, time or a time range): sets, adds to or takes out joints, a key's lift and lash; one step to undo |
| `studio_track`           | One joint's whole track: copy to another joint, mirror from the other side (or every Left/Right pair), scale, offset, smooth or take out |
| `studio_retime`          | The timing: faster or slower, only a part (trim), or every key after a time later or sooner (shift) |
| `studio_check_animation` | What the animation is and could have wrong: keys and joints against 32, joints going the long way round between keys, fast turns, a loop's seam, the lash as it plays (as `studio_lash_check`), its size as sent |
| `studio_compare_animation` | How far another animation is from this one, joint by joint, most apart first |
| `studio_lash_shape`      | The whip's lash on some keys from curl, sway and sag (the whip's length kept) |
| `studio_sample`          | Where joints are at given times as the animation poses the body (from its place: right, forward, up) |
| `editor_frame`           | The camera by a joint, the legs, the holding hand or the whole body, from the front, back, sides or top |
| `studio_cvar`            | An engine console variable read or set (the studio has `ca_UseLookIK` and `ca_LockFeetWithIK` 0) |
| `studio_wait`            | Waits for a bake, a Lash to keys or a Lash from motion to end and answers what it made |
| `studio_history`         | History (newest first) and what undo and redo would do; a change made over the MCP is marked `MCP ·` in History |
| `editor_tab`             | Shows one of the dock's tabs (Content, Timeline, History, Saves, AI) |
| `editor_ui`              | Hides the whole editor (panels and their texts, gizmo, grid, bones) so only the scene shows, to watch or record it; shows it again as it was |
| `studio_remove_prop`     | Takes the prop away                                              |
| `studio_bones`           | Shows (`on: true`) or hides the skeleton (Bones mode)            |
| `studio_joints`          | The joints: `{ id, name, parent }`, parent `-1` for the root     |
| `studio_choose_joint`    | Chooses a joint by `id` (drawn in orange with its name)          |
| `studio_find_joints`     | The joints whose names hold a text: `{ id, name, parent }`       |
| `studio_set_animation`   | A whole animation at once: keys with joint turns by name, blend, loop; plays it |
| `studio_get_animation`   | The animation now, joints by name                                |
| `studio_play`            | Plays or pauses the animation                                    |
| `studio_playhead`        | Puts the playhead at a time                                      |
| `studio_prop_physics`    | How the rope or lash moves (gravity, damping, mass, air, bend, wind) and the whip's grip |
| `studio_sky`             | The scene's sky: hour, weather preset, rain                      |
| `studio_joint_info`      | Joints' geometry in the body's frame (right, forward, up): place, bone, the parent's axes the turns are about, and how far +30° about each moves the bone's end |
| `studio_clothes`         | The character's outfit and the pieces taken off                  |
| `studio_make_mesh`       | Makes a simple prop (`box`, `cylinder`, `sphere` or `plane`: its size, colour and surface) as one of your meshes and shows it; made again under the same `name`, the one shown loads anew |
| `studio_texture`         | Your textures: the list, or one on your mesh with its tiling (`""` for the mesh's own material) |
| `studio_cloth`           | Your mesh as cloth (`on`, its `mass` in kg) or solid again       |
| `studio_wear_skin`       | Puts your skinned mesh (`name`: `<name>.skin` in the meshes folder) on the character; `""` takes it off |
| `studio_import_gltf_skin` | Makes a skin of a glTF file's skinned meshes (`path`, `name`, `map` for joints the names miss) and the character wears it |
| `studio_import_fbx`      | An FBX file (`path`, `name`) as a mesh (`as: "mesh"`) or a worn skin (`as: "skin"`), through Blender installed on this machine |
| `studio_import_gltf_animation` | Puts a glTF file's animation on the character, its joints matched by name (`map` for the rest), and plays it |
| `studio_import_gltf`     | Imports a glTF 2.0 file (`path`, `.gltf` or `.glb`) as one of your meshes (`name`) and shows it |
| `studio_mesh_key`        | The key at the playhead places your mesh where it stands (`on`), or no longer (`off`); without `on`, the mesh keys |
| `studio_mesh_body`       | Your mesh's body: `none`, `solid` (things hit it) or `falls` (a rigid body of `mass` kg that falls from where it is placed) |
| `studio_place`           | Where the prop is held (`RightHand`, `LeftHand`, or free at a place and turn) and where your mesh stands (place, turn, size) or which joint holds it (`joint`: a hand or any joint's name, `""` free) |
| `studio_game_files`      | Searches the game's own objects or animation clips (`kind`, `query` words, `page`): 50 paths a page with `pages` and `total` |
| `studio_game_prop`       | Shows one of the game's objects (`path`, `Objects/....cgf`) in place of the mesh; `""` takes it away |
| `studio_game_clip`       | Plays one of the game's clips (`path`, `Animations/....caf`) on the character, `loop` or once, its part `from`–`to`, `in_place` and `top_body`; `""` stops it, `"state"` answers where it is (and `lastBake`), `"bake"` bakes the part playing into the animation |
| `editor_screenshot`      | A picture of the game window as it is now (an image result)      |
| `editor_burst`           | Motion in one picture: `count` pictures `every_ms` apart in a contact sheet, each labelled with its time, the 3D view alone at first (`view_only`) |

Every change these make is one step you can undo. The answers of the changing tools and of
`studio_export` carry the same check (`problems`, `fastTurns`, `bytes`, `sent`, `fits`). A step
of any size goes through: a long one is sent in parts the editor puts together.

`studio_joint_info` is what lets an assistant pose a joint on purpose. Its `plus30moves` say,
per axis, where a turn of +30° moves the end of the joint's bone: if RightArm's `plus30moves.z`
is `[0, 0.2, 0.1]`, a positive Z turn swings the arm forward and up. After setting an animation,
`editor_screenshot` shows the assistant what it made.

`studio_game_files` answers, for example, `{ "kind": "clips", "query": "whipping executioner" }`
with:

```json
{
  "kind": "clips",
  "query": "whipping executioner",
  "page": 0,
  "pages": 1,
  "total": 1,
  "paths": [
    "Animations/humans/male/cinematics/cin_s9906t_crime__whipping_troskovice/cin_s9906t_crime__whipping_troskovice_cin_s9906t_executioner.caf"
  ]
}
```

The first search reads the game's packs, which takes a few seconds. Until then the tool answers
`the game client is still reading the game’s packs: ask again`.

An animation for `studio_set_animation`, a whip strike that loops:

```json
{
  "keys": [
    { "time": 0, "pose": { "RightArm": [40, 0, 0], "RightForeArm": [0, 0, 0] } },
    { "time": 0.4, "pose": { "RightArm": [130, 0, -40], "RightForeArm": [0, 0, 70] } },
    { "time": 0.6, "pose": { "RightArm": [60, 0, 50], "RightForeArm": [0, 0, -10] } },
    { "time": 1.2, "pose": { "RightArm": [40, 0, 0], "RightForeArm": [0, 0, 0] } }
  ],
  "blend": "spline",
  "loop": true
}
```

The answer names the keys, the length and any joint names it did not find
(`unknownJoints`).
| `studio_undo`, `studio_redo` | Undo and redo                                                |
| `editor_camera`          | Puts the free camera at a position, looking at another           |
| `editor_camera_position` | Where the free camera is                                         |

Each tool is a step of the open editor, and its answer comes back as the tool's result. The
studio's screen shows each change as if you had made it yourself, and each change can be undone.

## How studio assets stay KCDS-only

The studio's files are served to the game from memory, and no plain game file is written to
disk. The only thing written to the game's folder is an empty 22-byte stub,
`Data/Levels/kcds_studio/level.pak`. It is there because the game lists as levels only the
folders that have a `level.pak` on disk.

Packed assets are sealed in a `.kcdsasset` container, encrypted with AES-256-GCM. A container
opens only with its resource's name and secret, and only when nothing in it was changed. To
the plain game and to other mods its bytes mean nothing. The studio's own level is sealed for
your machine, so a copy of the file does not open on another PC.

The honest limit is that KCDS must decrypt an asset to show it. Someone who reads the game's
memory while it runs can take the decrypted files. The container stops casual extraction and
reuse by other mods, not a determined attacker with a debugger.
