# World Architect

World Architect is the mod's world editor. It runs only on your own machine, in the game's developer mode, with no server:
no server can turn it on and no resource script can open it. Inside the game you fly a free camera and build with the
cursor: you place the game's houses, structures and doors, shape the ground, plant trees and bushes, and add your own
invisible collision. What you build exists in that game only, the same as the client's `Prefab`, `Terrain`,
`Vegetation` and `Collision` (see [Props](props.md#in-client-scripts)).

## Opening it

Start the game with KCDS's development launcher, which runs the game in its developer mode, and set
`KCDS_WORLD_ARCHITECT=1` in its environment. The game then shows no server list and joins no server. It loads Trosecko
on its own and opens the editor when the world is ready. To start on another map, set `KCDS_WORLD_ARCHITECT_MAP` to
`kutnohorsko`, `klaster` or one of your [custom maps](#custom-maps). Without the developer mode,
`KCDS_WORLD_ARCHITECT` does nothing. On a game's first start, its own pages (the EULA, telemetry) come first: the world
loads once you have answered them. Nothing is answered for you.

**WORLD ARCHITECT** in KCDS's main menu, above the server list, opens the editor in the game you are playing.
[ANIM / ASSETS](asset-studio.md#opening-it) beside it opens the Asset/Animation Studio the same way. <!-- only studio -->
**Leave** at the right end of the menu bar saves your work,
takes it out of the world and goes back to the main menu with its server list. A game started in the editor with the
development launcher has no server list, so there Leave closes the game.

While the editor is open, your own character is hidden; Play as player closes the editor and puts you in the world as
your character where the camera was looking, facing the same way: on what the middle of the view looks at when it is
within 80 m, otherwise on the ground under the camera. Press **Esc** (or **F10**) to go back to the editor; the game's pause menu does not open meanwhile.

What you build is kept per map in `%LOCALAPPDATA%\KCDS\architect\projects\<map>.json`: the ground you shaped, the
buildings, plants and collision shapes, the level's objects you hid, and the time of day and weather you set. It is saved a moment after each change and when
you exit, and built again the next time the editor opens on that map. The right end of the menu bar says Saved (a green
dot), or how many changes are not saved yet (an amber dot). Undo and Redo cover one session.

### Saves

The **World Architect** button at the left of the menu bar opens the **Saves** tab of the bottom panel. Type a name
(letters, digits, spaces, `-` and `_`, up to 48) in its field and press **Save as**: the work as it is now is kept in
`Documents\KCDS\World Architect\<name>.json` on this PC. Each save is listed with its date and size, newest first:

- **Load** puts the save in place of the work (the ground, buildings, plants, shapes, hidden objects and the light).
- **Merge** adds the save's ground edits, buildings, plants, shapes and hidden objects to the work; the light stays.
- **Delete** removes the file.

A save made on another map is loaded or merged on that map: switch to it first.

**Reset map** at the right end of the Saves tab starts the map again from the game's own. It asks first in the menu
bar ("Reset ... to the game's map? Your work is saved first.", **Reset** or **Cancel**). Reset keeps the work as a save
named `<map> before reset <date> <time>`, then takes everything you built off the map: buildings, plants, props,
lights, sounds, effects, collision shapes and zones, the ground's shape and paint, the cleared grass, and the level
objects you hid or moved. The time and weather go back to how the editor starts (noon, held, clear). Your saves and
your own models and textures stay, so the kept save loads or merges back at any time. A reset is not an Undo step: the
kept save is the way back.

### Switching the map

The map's name in the middle of the menu bar is a button. A press opens a list of every map under it: the game's three
(Trosecko, Kutnohorsko and Klaster), then your [custom maps](#custom-maps), with the current one marked. With more than
12 maps the list scrolls with the mouse wheel or a press on its scroll bar. A press beside the list, or on the bar again,
closes it. Choosing another map asks "Switch to ...? Your work here is saved first." with Switch and Cancel.
Switch saves the project, takes your work out of this world, closes the editor and loads the other map's world; the
editor opens again there, under its loading screen, with that map's project.

### Custom maps

A custom map is a map of your own, started from nothing: a flat square of ground with no buildings, plants or people,
which you then shape and fill in the editor (a city of your own beside the game's two regions, for example). An AI
assistant makes one with the MCP tool `new_map` (a name, `size`, `height` and `ground`): the MCP server writes it on
your PC from your own game files, the game lists it at once, and `switch_map` opens it. Or make one with the tool in
the KCDS repository:

```
python Dev/tools/architect/make_custom_map.py prague --size 1024 --height 32
```

The name is 1 to 32 lower-case letters, digits or `_`, and the map is called `kcds_map_<name>` (here
`kcds_map_prague`). `--size` is the side in metres (128, 256, 512, 1024 or 2048) and `--height` the ground's height.
To start from hills and valleys instead of flat ground, give a grey picture of the ground (8 or 16 bits per pixel):
`--heightmap valley.png --rise 80`. Black is at `--height`, white `--rise` metres above it (100 by default), and the
picture's top is the map's north; it is scaled to the map's size. This needs Python's `numpy` and `Pillow`.
`--ground` is the material the whole map starts covered with (`grass` by default; also `meadow`, `pasture`,
`trampled`, `soil`, `mud`, `stony_mud`, `sand`, `gravel`, `forest`, `forest_leaves`, `cobblestones`, `paving`,
`cobblestones_grass`, `river_shore`), and sets the colour the ground has from afar. Every part of the map carries all
fifteen materials, so the Paint brushes put any of them anywhere.
A map a running game has opened is in use until that game closes: the tool then writes nothing and says
`kcds_map_<name>: level.pak is in use (a running game opened this map): close the game or choose another name`, and
`new_map` makes no map of a name that is taken.
The tool also writes the map's navmesh, `recast.pak` beside `level.pak`, so the game's people can walk on the ground
(the People preview below shows it): walkable squares of 4 m that follow the ground, in 16 m tiles. What you build on
the map later is not in it until you bake it: **Bake** under NAVMESH in Details (custom maps only, also the AI
command `bake_navmesh`) makes it again from the ground as it is now, with your terrain shaping, and leaves out the ground
under your buildings, props and collision shapes. The game has the map's navmesh open, so the baked one is used from
the next start. `--no-navmesh` leaves it out.
The tool builds the map from your own game files (Klaster's level settings and plant kinds, nothing placed) and writes
`%LOCALAPPDATA%\KCDS\maps\kcds_map_<name>\level.pak` with a `map.json`; it never writes into the game's folder. The
game finds every map in that folder when it starts. Open it with `KCDS_WORLD_ARCHITECT_MAP=kcds_map_prague`, or from
the editor's map choice, where the custom maps follow the game's three as "Custom: prague" (an AI assistant's `state`
lists them and `switch_map` opens one): you start in its middle, and every tool works there as on the game's maps (Foliage offers Klaster's plants).

A server runs a custom map when it is started with that name as its level (for example
`kcd2s-server.exe 7861 0.0.0.0 kcds_map_prague ...`, or `KCDS_SERVER_LEVEL=kcds_map_prague`). Export your work for
the server (below): the export carries the map itself (`maps/kcds_map_<name>/`, about 1 MB), and the server sends it
with the resource to every player who joins, with its navmesh. Their game keeps it in their own `%LOCALAPPDATA%\KCDS\maps` folder and
loads it like a map of their own; the next time it is already there. A player whose game gets no map within 20 seconds
of the server's resources is told the level is missing. The map has no spawn point of its own: the export also lets
players appear on it.

## The screen

- **Menu bar (top):** the KCD2-Synced mark, the editor's name, the map switch in the middle, whether the work is saved
  and **Leave** (back to the main menu and the server list).
- **Toolbar:** the tools (Select, Move, Rotate, Scale), the snaps (grid, angle, and **Surface**: fit the ground under
  a building, and rest a prop on what you press: on a floor its foot on it, on a ceiling hung from it by its top, on a
  wall its back against it and turned to face out, as the preview shows), the modes (Place, Terrain,
  Foliage, Collision, Props, Effects and Zones), then Undo, Redo and Exit.
- **Outliner (left):** everything you placed, by group (Buildings, Foliage, Collision, and the level's own objects you
  hid). A press on a group folds or unfolds it; a press on a line selects it. Its search box finds by name; its filter
  button shows one group at a time.
- **Details (right):** the selection's location, rotation and, for collision shapes, size. Values with `-` and `+` change
  by one step when you press either end. In Terrain mode, this panel shows the brush instead. With nothing selected,
  it shows the snaps and the world's time of day and weather (see below). When its rows are more than the panel holds,
  a scroll bar shows at its right and a footer at its bottom (`rows 1–17 of 40`): ▲ and ▼ turn a page up or down. A new
  selection shows its rows from the top. On the game's own maps the NAVMESH section says the map has the game's own
  navmesh; **Bake** shows on custom maps.
- **Content (bottom):** the folders of the current mode with their counts, and their items as tiles with a picture of
  each. A press on a tile chooses it; the search box finds in the folder by name. The History tab lists the changes,
  newest first. The Validation tab lists what may be wrong: two buildings that overlap, a plant standing inside a
  building or a collision shape, a collision shape wholly inside a building (solid already), the level's own objects
  you hid, and what looks wrong at once in a room: a prop that floats over what is under it (more than 10 cm), sinks
  into the floor or, hung (a chandelier), hangs in the air under the ceiling or from nothing; a lamp, candle or
  chandelier with no light of yours within 2.5 m (a prop gives no light by itself); and a light less than half a metre
  over the floor (it lights a round spot there only). Props fixed to walls (shelves, pictures, banners) and pieces of a
  building are not checked for resting. A press on one selects it and flies the camera to it. The editor takes the pictures itself, a page at a time (the tiles say Loading meanwhile), and keeps them
  in `%LOCALAPPDATA%\KCDS\architect\pictures`, so later sessions show them at once.
- **Status bar:** the selection, what just happened, where the last press met the ground, and the snaps.

To type in a search box, press on it; Enter or Escape ends. Hold the right mouse button to fly as usual.

The screen fits any window size and shape: the panels keep their size, and the world view and the content strip take
the room that is left (a wide window gives a wider view and wider tiles, a taller one more Outliner lines). A window
too narrow for all the panels shows everything a little smaller instead of squeezing it.

The editor's text is drawn in the same font as the KCDS mark. While the editor is open, the game's own interface (compass, stats, hints and nameplates) is hidden. The KCD2-Synced mark
sits at the left end of the editor's menu bar.

## The cursor and the camera

The cursor works the editor, as in any map editor. Hold the **right mouse button** to look around with the mouse and fly
with **W A S D**; let go and the cursor is back. The editor opens while the world loads, so you land in it.

| Key            | Does                                                      |
| -------------- | --------------------------------------------------------- |
| Q, W, E, R     | Select, Move, Rotate, Scale                               |
| 1 to 7         | Place, Terrain, Foliage, Collision, Props, Effects, Zones |
| Ctrl+Z, Ctrl+Y | Undo, Redo                                                |
| Delete         | Deletes the selection                                     |
| F              | Flies the camera to the selection                         |
| Ctrl+D         | Duplicates the selection beside it                        |
| Escape         | Stops placing; pressed again, clears the selection        |

## The camera

The **Fly speed** row under CAMERA in Details sets how fast the free camera flies, from 1 to 200 metres a second; the
chip in the view's corner steps through 4, 8, 12, 24 and 48.

## Time of day and weather

The WORLD rows at the bottom of Details (whatever is selected) set the light you see your work in. Each `-` or `+`
changes the game at once:

| Row            | Values                                                                                     |
| -------------- | ------------------------------------------------------------------------------------------ |
| Time of day    | 00:00 to 23:30 in half hours, on the game's current day                                    |
| Freeze time    | on: the time stands still; off: the clock runs at Clock speed                              |
| Clock speed    | Held, ×15, ×60 or ×600 game seconds per real second                                        |
| Weather        | Clear, Few clouds, Overcast, Cloudy, Showers, Fog and drizzle, Fog, Storm (the game's own) |
| Freeze weather | on: the weather stays as you set it; off: the game's own weather runs                      |
| Wind           | Calm, Breeze, Windy or Gale                                                                |

The editor starts at 12:00, the clock held, clear, with a breeze. The project keeps what you set, and the next session
opens in the same light. While Freeze weather is on, the game's own weather does not take over.

## Modes

- **Place:** Houses, Structures and Doors from the game's prefabs. Choose a tile: a see-through preview of it follows the
  cursor over the world, E turns it by the angle snap, and a press puts it down on the grid. Then the editor is back to selecting; hold Shift to put
  down more, Escape stops. A prefab is placed whole, with its doors, and it is solid. With "fit ground" on (the default),
  the ground under it is flattened first: its own footprint (3 to 20 m from its centre) fading out over 3 m more, so a
  house on a hillside stands level, and the ground you shaped around it stays as it is.
  **People** (Man, Woman) puts one of the game's own NPCs on the ground to see where people can walk: select it, press
  **Walk to** in Details, then press a place in the world. It walks there by the game's own pathfinding on the map's
  navmesh, and Details shows Walking, then Arrived, or "No path (no navmesh)" where there is no way (a custom map
  made with `--no-navmesh` has none). It is a preview in the editor view: it moves, turns and undoes like the rest, but it is not kept in
  the project, in saves or in an export.
- **Terrain:** the Raise, Lower and Flatten brushes, Clear nature and Add nature, and the Paint brushes. An orange ring on the ground
  follows the cursor, as wide as the brush. A press shapes the ground under it; holding the left button keeps shaping
  where the cursor goes, and the whole stroke is one undo step. Flatten levels a stroke to the height where it started.
  The details set the brush's size and strength. Collision follows the new ground, the level's roads are laid on it
  again, and the grass carpet moves with it. What stands there goes up and down with the ground: your plants, props,
  effects and lights, and the level's loose objects (stones, logs, crates; not buildings, walls, roads or bridges).
  Undo puts them back, and the project keeps where the level's objects went. Lower digs as deep as you hold it (a
  quarry, a pit, a cellar hole): the ground keeps going down instead of stopping at the lowest point the map had there.
  The terrain is a height map, so the ground cannot overhang: a cave needs your own rock or building models over the
  pit.
  - **Paint** (Paint grass, meadow, pasture, trampled earth, soil, mud, stony mud, sand, gravel, forest floor,
    leaves, cobblestones, paving, cobbles and grass, river shore) covers the ground under the brush with that
    material: how it looks up close, the sound of footsteps on it and the dust it raises. The inner half of the ring
    is fully covered and the outer half blends into the ground around, up to three materials mixed on one metre, so
    a road or a muddy yard has soft edges. A custom map takes every material everywhere. On the game's own maps a
    patch of ground takes only the materials it already has there, and the rest of the stroke stays as it was. From
    far away the ground keeps the map's own colour.
  - **Clear nature** hides the level's own trees, bushes and plants under the brush in this game (they count as hidden
    level objects) and takes away your own plants there (also the ones Add nature planted, many at once). The level's grass carpet there (the low grass drawn over the
    ground) is cleared too, and comes back with Undo.
  - **Add nature** plants the grass the ground there already grows: the kinds of the level's grass carpet under the
    brush, as often as they grow there, 1.2 clumps a square metre by default (Density in Details, 0.2 to 3; at most
    160 a dab), each on the terrain and
    turned at random. Where the ground grows no grass, it scatters the plant last chosen in Foliage instead, "Plants a
    dab" at a time.
- **Foliage:** the level's Trees and Bushes. A plant stands solid and bends in the wind. With a plant chosen, the
  PAINT BRUSH rows in Details scatter it: "Plants a press" (1 to 20) and "Radius" (2 to 30 m). Each press then plants
  that many, each on the ground at a random spot within the radius and turned at random, as one undo step.
- **Collision:** your own invisible solid shapes, drawn in orange while this mode is open: a wall (6 by 0.3 by 3 m), a floor
  (4 by 4 m), a ramp (3 by 5 m, leaning 20 degrees, its low edge on the ground) or a box (2 m). They block players and
  horses.
- **Props:** any object of the game's own catalogue (over 11,000 meshes) in seven folders: Furniture, Decorations,
  Fixtures and lights, Tools, food, carts, Work props, Rocks and stones, and Building parts (walls, beams, roofs, stairs
  and the like), and **Yours**: your own models (see below). A prop stands solid where you put it, turned with the angle
  snap; it moves with the arrows and is kept in the project and in saves, in this game only. Its **Material** row in
  Details draws it with one of your own materials instead of its own.
- **Effects:** Lights, Sounds, and the game's particle effects (583): Fire and explosions, Smoke and dust, Weather and water
  (rain, lightning, streams), Work (smithy, bathhouse, woodwork ...), Combat and blood, and Other. A light (candle,
  torch, fireplace, lantern, lamp, moonlight, red, green, blue) is a point light; its Details set Colour, Intensity
  (0 to 100), Radius (0.5 to 100 m) and Shadows. An effect runs where you put it
  until you take it away; it moves and turns like a prop and is kept in the project and in saves, in this game only.
  Its Details set Size (×0.25 to ×32) and Plays again (every 1, 2, 5 or 10 s, or runs on). Effects that play once and
  are gone (blood, combat, flashes, explosions, lightning) play again every 2 s, and blood starts four times as big, so
  you see them where they stand. A sound is one of the game's world sounds that loop (about 190: a village's or a
  monastery's ambience, a pond, bees, insects, a torch, a forge, a big fire): it is heard around the place you put it
  until you take it away, moves like an effect and is kept in the project and in saves; Export for server takes it
  to every player's game.
- **Zones:** areas that block nothing, drawn in blue: Safe zone, Spawn area, Trigger area and Private ground, each with
  its labels (`safe`, `spawn`, `trigger`, `private`). They move, turn and size like collision shapes (Width, Depth,
  Height in Details), and Details shows their labels and area id. The project keeps them under `areas` exactly as a
  server's `kcds.world.Area.create` takes them ([Areas](areas.md)): a box with its `id`, `position`, `rotation`, `min`,
  `max` and `labels`, so a server script can make the same areas and react to `areaEnter` and `areaExit`.

## Tools

With nothing chosen in Content, a press in the world uses the tool:

- **Select** marks what the cursor is over with a thin box, the target a press would pick, and names it in a small box
  beside the cursor (with whether it is yours or the level's). The selection is shown with
  an orange box and the move gizmo: the X, Y and Z arrows (red, green, blue) at its foot, seen through walls. Hold an
  arrow and drag: the thing follows it while you hold, and letting go makes the move one Undo step. Select picks
  what the cursor is on: a collision shape whose footprint holds it, your own building, prop, plant or person, or the
  level's own object (a tree, a bush, a building, a table in a house or a level entity), even when a thing of yours
  stands right beside it. Your lights, effects and sounds, which a press cannot touch, are picked within 4 m of it, and
  a press on open ground picks your nearest thing within 12 m. The level's invisible collision (some buildings have their
  own, which stays where the building was deleted) is picked and named **Invisible collision**: Delete takes it away
  like any level object. Its details show where it stands; **Hidden in this game** (or Delete) hides it and takes its
  collision away, in this game only (a plant drawn in the grass carpet, such as sticks or stones on the ground, takes
  the carpet within 0.6 m of it along). Undo, or the switch again, brings it back. Dragging a level object's arrow (or
  the Move tool), or turning it with Rotate, hides it and puts your own copy there, a prop of its mesh or a plant of its kind, turned as it was (or as you turn it).
  A level entity (a door, a cart, anything the level made as an entity) moves itself instead, live while you drag:
  a door still opens and closes where you put it. The Outliner lists it under **Moved in the level**; Undo puts it back.
  Scale (the tool, or the Scale row in Details) grows or shrinks a level object where it stands, a building, a fence, a
  rock or an entity; the level's trees and plants keep their size.
- **Move** puts the selection where you press, on the grid.
- **Rotate** shows a ring around the selection's foot: drag round it and the selection turns with the cursor in steps of
  the angle snap (5, 15, 45 or 90 degrees); a press elsewhere turns it one step. A prop (and a level object, as your
  copy) also has a red ring and a green ring standing upright: they tilt it about its own X and Y axes, forwards or
  sideways, shown as you drag and one Undo step when you let go.
- **Scale** grows a plant or a prop by a tenth (Details' Scale row goes up or down), and a collision shape by one step on
  each side. Hold the button and it keeps growing; move the mouse while you hold it, and it grows to the right or up
  and shrinks to the left or down. Everything one press did is one Undo step. Buildings keep the size the game builds
  them at.

With nothing selected, a press with Move, Rotate or Scale selects what is there, as Select does.

A level entity shows its own **PROPERTIES** in Details, as the level set them (a door's `Lock.bLocked`, a light's
radius and colour, a spawn's settings): a yes / no value switches, a number steps with `-` and `+` (or held and
dragged sideways), a text is shown. A change is made in this game only and is not saved; many entities read their
properties when they are loaded again, so some show the change only then.

A level door shows **Open** in Details: switch it and the door swings open or shut as the game moves it (a locked one
is unlocked first). A level object near one of the level's gates (the Malešov drawbridge, the Suchdol portcullis, the
castle and town gates) shows the gate under **GATE**: step it through its states (closed, open, barricaded, ...) and
watch it move. Both happen in this game only and are not saved; the next load shows the level's own state.

### Doors and gates of your own

Many doors and gates in the levels are still meshes that never move. A prop of yours can open and close instead:
select it and switch **Opens** under **DOOR / GATE** in Details. A level mesh (a door or gate the level only draws)
shows **Make it open**: your copy of it takes its place, the level's own is hidden, and its copy opens. Then set:

| Row          | What it does                                                                                  |
| ------------ | --------------------------------------------------------------------------------------------- |
| Hinge        | the side it turns about: `left` or `right` edge of its box, or its `middle`                   |
| Open turn    | how far it turns to open, in 15° steps (minus turns the other way)                            |
| Lift         | how far it rises to open, in metres (a portcullis: no turn, a lift)                           |
| Slide        | how far it slides along its own width to open, in metres (a sliding gate)                     |
| Seconds      | how long opening or closing takes                                                             |
| Closes after | seconds it stays open before it closes by itself; `stays open` keeps it open until used again |
| Preview      | shows it opening or closing in the editor as players will see it                              |

Its closed pose is where the prop stands, so move and turn it as usual. Each change is one Undo step and is kept in the
project. On a server from **Export for server** (see below), every player sees the same door: walking up to it shows
the use prompt (`Open` or `Close`), and using it moves it along its way, swinging about its hinge in steps of 30°, in
everyone's game. A door left open closes by itself after **Closes after** seconds.

In Details, hold a value with steps (Location, Rotation, Scale, sizes) and move the mouse sideways: it changes as you go,
and letting go makes it one Undo step. The arrow before a section's name (TRANSFORM, ACTIONS ...) folds the section.
In the Outliner, a second press on a line flies the camera to it.

You cannot die while World Architect runs, in the editor or playing as the player. A building you place hides the
level's grass under its floor; the grass comes back when the building goes.

Each change is one step. Undo takes it back and Redo does it again. When a house was placed on fitted ground, undoing it
also gives back the ground it stood on.

## Your own models and textures

Put your own files in `Documents\KCDS\World Architect\assets`, at their game paths in a folder of your own, the same
layout a resource's `stream/` folder has ([Sounds and game files](audio-and-assets.md)):

```text
Documents/KCDS/World Architect/assets/
  objects/kcds/market/stall.cgf
  objects/kcds/market/stall.mtl
  objects/kcds/market/red_cloth.mtl
  objects/kcds/market/textures/red_cloth_diff.dds
```

Each time the editor opens it looks at the folder again and loads what is there into the game. Your models (`.cgf`)
are in Props' **Yours** folder, and your materials (`.mtl`) in a prop's Material row. The folder takes models (`.cgf`,
`.cgfm`, `.chr`, `.skin`, `.cdf`), materials (`.mtl`) and textures (`.dds` with its `.dds.1` ... parts), each up to
64 MB and 512 MB in all. A file outside `<top folder>/kcds/<your folder>/` is left out, so your files never take the
place of the game's own. The same files go into a server resource's `stream/` folder unchanged, so a server can use what
you placed.

### Textures made in the editor

**Yours** starts with **New stone blocks**, **New bricks**, **New plaster**, **New wood planks**, **New floor tiles**,
**New cobbles** and **New roof tiles**. A press makes a material of yours at once, named `stone_blocks_1`, `plaster_2`
..., in `objects/kcds/textures/`: a seamless 512-pixel texture drawn by code, each press a different one, with a normal
map from its relief so light rakes across the joints and grain. Choose it in a prop's **Material** row or in its
**Slots**. The AI assistant's `make_texture` makes the same with its own colour, size (256, 512 or 1024), repeat
(`tile`), number of blocks across (`count`), `variation`, `wear`, joint width and relief.

### A model's slots

A model's material has slots (sub-materials): a church's are `glass`, `plaster`, `plaster_b`, `planks`, `shingles`
.... **Details > Slots** lists them for the selected prop, without the model's lower details and collision, with what
each is drawn as. `-` and `+` draw a slot as one of your single materials (a texture you made or a photo), or its own
again. The change is made on every prop drawn with the same material, so the six pieces of an opened church change
together, as one Undo step: the editor writes the model's material with that slot changed as a new material of yours in
`objects/kcds/materials/` and the props draw with it. The project keeps what each slot is drawn as.

The AI assistant's `slots` answers the same list and, for your own models, how much of the model each slot draws;
`raycast` names the slot of a drawn face it meets. `set_slot` draws a slot as a material of yours, as a slot of a game
material (`lookSlot`), like another slot of the same material (`like`), or as its own again (`reset`), with `tile` to
repeat its pictures on a large face.

A photo becomes a material of your own without other tools: the AI assistant's `texture` tool takes a picture on your
PC (PNG; JPG and other kinds when [ffmpeg](https://ffmpeg.org) is installed) and writes, in your assets folder,
`objects/kcds/<folder>/textures/<name>_diff.dds` (the picture at the nearest power-of-two size, up to 2048 pixels a
side, with all its smaller levels) and `objects/kcds/<folder>/<name>.mtl` (the game's `Illum` shader drawing it,
`tile` times across a face, sounding as `stone`, `wood`, `plaster`, `metal`, `cloth` or `ground`). The game loads both
at once; `set_material` then draws a prop with it, as the Material row does:

```json
{
  "image": "D:/Pictures/tyn_wall.jpg",
  "name": "tyn_wall",
  "folder": "praha",
  "surface": "plaster",
  "tile": 2
}
```

answers `{ "material": "objects/kcds/praha/tyn_wall", "texture": "objects/kcds/praha/textures/tyn_wall_diff.dds",
"size": { "width": 1024, "height": 512 }, "from": { "width": 1600, "height": 900 } }`.

## Your work on a server

**Export for server** (Details with nothing selected, under EXPORT) writes your work as a resource folder,
`Documents\KCDS\World Architect\exports\architect_<map>`, and copies its path. Copy the folder into your server's
`resources` folder and run the server on the same map. Every player then sees it:

- from the server: your buildings and props (your [doors and gates](#doors-and-gates-of-your-own) with their use
  prompt), the level's objects you hid, moved or scaled, your zones (as areas),
  effects and lights, and your **People** as the server's NPCs (`Npc.create`), each walking back and forth between
  where it stood and where you last sent it with **Walk to** (`patrol`, ping-pong), so the walk you tried on the navmesh
  in the editor goes on on the server;
- made by each player's game when they are ready, the same for everyone: the shaped ground (with the pads under fitted
  buildings), your plants, your invisible collision and your sounds (`Audio.loopAt`).

Players appear in your **Spawn areas** (Zones), half a metre above each area's floor, one area after another. On a
custom map without a spawn area they appear on the ground under the camera as it was when you exported. Otherwise the
export leaves the spawn to the server's other resources.

The folder holds `package.json`, `server.js`, `client.js`, the work itself in `work.json`, and a `README.txt`; on a
custom map also the map, in `maps/`. Your own
models (the assets folder) go into the resource's `stream/` folder for every player to receive them. The level's grass
carpet hidden by Clear nature stays in the editor only.

## Blender

**Export for Blender** (Details with nothing selected, under EXPORT) writes your work and the ground around it into
`Documents\KCDS\World Architect\exports\<map>-<date>` and copies that folder's path:

| File             | What it is                                                                                                     |
| ---------------- | -------------------------------------------------------------------------------------------------------------- |
| `ground.obj`     | the ground as a mesh, in metres, Z up, the area's south-west corner at the origin                              |
| `scene.json`     | every thing of yours: kind, the game's model path, name, place, turn (a quaternion), scale and box; your zones |
| `import_kcds.py` | a script for Blender 3.6 or newer: Scripting tab, Open, Run Script                                             |

The script builds the ground and a box of the right size and turn for every thing, named after it, with its model path
kept in the custom property `kcds_model`, in a collection named after the map. Model new buildings and props in Blender
around them and bring them back with the AI assistant's `import_model` tool: no CryEngine exporter is needed.

### Bringing a Blender model into the game

Export the model from Blender with **File > Export > Wavefront (.obj)**, its materials on (the `.mtl` and the pictures
beside the `.obj`), and ask the AI assistant to import it. `import_model` makes, in your assets folder:

| File                                                 | What it is                                                                  |
| ---------------------------------------------------- | --------------------------------------------------------------------------- |
| `objects/kcds/<folder>/<name>.cgf`                   | the model as the game's own, drawn with your materials                      |
| `objects/kcds/<folder>/<name>_1.cgf`, ...            | a large model's further pieces (more than 65,535 vertices), placed together |
| `objects/kcds/<folder>/<name>.mtl`                   | its materials, one for each of the OBJ's                                    |
| `objects/kcds/<folder>/textures/<name>_<n>_diff.dds` | each material's picture (or its plain colour)                               |

The game loads them at once, and the assistant places the model (every piece at the same spot and turn). The model's
origin in Blender is the spot it stands on; metres, up as Blender's OBJ export writes it (`up: "y"`, or `"z"`).

What collides and where doors go comes from the names of the model's objects in Blender's Outliner:

- `collision...` (also `col_...`, `UCX_...`): not drawn; its box is solid, so players and people walk on and bump into
  it. Model walls, floors and stairs' slopes as simple boxes under these names.
- `door...`: not drawn; its place comes back in `doors` (`at`, `size`, `turn`), where the assistant puts one of the
  game's own doors (Content, Doors), which opens and closes as in the game.
- anything else is drawn. When the model names no `collision...` object, each drawn object's box is solid
  (`collision: "objects"`); `"bounds"` makes one box of the whole model and `"none"` nothing.

The game draws only a face's front, the side its normal points to. A model whose faces point either way (survey data of
a city, a mesh from a scan) comes out with holes: import it with `two_sided: true` and every face is drawn from both
sides. The pictures are kept as sRGB, as the game's own textures, so their colours look as in your picture viewer.

Positions are kept as the game keeps its own (half floats): a model may reach at most 250 m from its origin, so split a
larger area into several models in Blender, each with its own origin.

```json
{ "file": "D:/Blender/staromestske_namesti.obj", "name": "tyn_houses", "folder": "praha" }
```

answers, for example:

```json
{
  "mesh": "objects/kcds/praha/tyn_houses.cgf",
  "meshes": ["objects/kcds/praha/tyn_houses.cgf"],
  "material": "objects/kcds/praha/tyn_houses",
  "size": { "min": [-12.4, -8, 0], "max": [12.4, 8, 17.6] },
  "pieces": 1,
  "triangles": 18420,
  "materials": 6,
  "collision": { "kind": "marked", "boxes": 14 },
  "doors": [
    {
      "name": "door_front",
      "at": { "x": 0, "y": -8, "z": 0 },
      "size": { "width": 1.2, "height": 2.2 },
      "turn": 0
    }
  ]
}
```

## AI assistant (MCP)

An AI assistant can use World Architect for you through its MCP server: it places buildings and plants, moves and
deletes them, shapes the ground, clears and adds nature, sets the time and weather, moves the camera, undoes and
saves, all as Undo steps in the editor, as if you did them by hand.

- The game listens for the assistant on this PC only (127.0.0.1, port 47870, or `KCDS_ARCHITECT_AI_PORT`) and only
  in World Architect's local mode. At each start it writes a new token to `%LOCALAPPDATA%\KCDS\architect\ai-token`;
  the MCP server reads it, so only programs of your Windows account can connect.
- The server comes with the game client, in its `architect-mcp` folder (Node.js 18 or newer, nothing else to
  install). The editor's **AI** tab (beside Saves) shows whether an assistant is connected and gives, each with a
  **Copy** button: the command for Claude Code (`claude mcp add kcds-world-architect -- node "<path>\server.cjs"`), the
  `mcpServers` entry for Claude Desktop and other MCP apps, and the token (the server reads it by itself).
- Positions are world metres (x east, y north, z up); leave z out and the ground's height there is used. Turns are
  degrees. The assistant asks `state` first, `catalogue` for names to place and `list` for the ids of your things. Every thing
  it lists or places comes with its `box`: `min` and `max` in metres, in the thing's own frame before its turn, so
  the assistant can set houses front to front without gaps or overlaps.

The tools: `state`, `open`, `close`, `switch_map`, `camera`, `fly_speed`, `look` (hour in half hours, clock,
weather, wind, `freeze_time`, `freeze_weather`), `catalogue` (names, with each building's `box` and `size` in metres;
`folder`, `minWidth` ... `maxHeight` keep only what fits, `sizes: true` sizes props too), `list`, `place_many` (up to 300
in one call, `avoidOverlap` leaving out what would stand in another), `fill_row` (a street front from one point to another lined with the game's city houses side by side, each turned so its street front, with its doors, faces the street, no gaps or overlaps), `place` (also the collision shapes and the Zones: `wall`, `floor`, `ramp`, `box`, `safe`, `spawn`,
`trigger`, `private`, sized with `shape_size`), `select`, `move`,
`export_blender`, `export_server`, `delete`, `duplicate`, `scale` (steps of ×1.1 on anything that scales), `shape_size`, `pick` (what stands at a point, also the level's own objects, with an id
to move or hide; with `from` and `to`, what that line meets first, such as a door in a wall; a level entity's answer
names its class, `entity: 'AnimDoor'`), `hide_level_at`, `hide_level_named` (hides the level objects near a point
whose file name has a given text, such as `gate_a_doors`, even where your own copy stands on top; the camera must be
near, the level streams in only there; a miss lists the names nearby), `show_level` (shows a hidden one again),
`open_model` (a copy of a closed game building you can walk into, below), `door` (opens or closes a picked level door), `properties` and `set_property` (read a picked or the selected level entity's own properties and set a bool or number one in this game), `gates_near` and `gate` (step one of
the level's gates through its states), `bake_navmesh` (a custom map's navmesh from the ground as it is now, for the next start), `walk` and `npc_state` (send the selected preview NPC, placed as `npc_man` or
`npc_woman`, walking to a point, and read whether it arrived), `terrain` (raise, lower, flatten along points), `paint`
(a ground material along points, its dabs half a brush radius apart along the whole line: roads, yards, a square's
paving) and `materials` (the names it takes), `texture` (a material of your own from a photo, below), `import_model` (a Blender model as the game's own, see
[Bringing a Blender model into the game](#bringing-a-blender-model-into-the-game)), `assets` (your
models and materials, the folder looked at again) and `set_material` (a placed prop drawn with one of your materials),
`clear_nature`,
`add_nature`, `new_map` (a new flat custom map, made on this PC; then `switch_map` to it), `undo`, `redo`, `history`, `ground_height`, `raycast` (also meets
the drawn faces of your own models, such as an opened building's vault, which the game's own rays pass through: such a
hit says `drawn: true`), `screenshot` (the view as a picture, so the
assistant sees what it built, with its light and colour measured), `review` (below), `make_texture`, `slots` and `set_slot` (see [A model's slots](#a-models-slots)), `light` (a light of Effects >
Lights placed from a preset or changed by id: `color` as `[r, g, b]` or a colour row's name, `intensity`, `radius`,
`shadows`, one Undo step), `effect` (a placed effect's `size` and `repeat`), `saves`, `save`, `load`, `merge`, `delete_save`, `reset_map` (the Saves tab's
**Reset map**, called with `confirm: true`; its answer's `keptAs` names the save the work was kept in).

### Furnishing a room and judging it

`place` takes `snap` for props, so the assistant does not guess heights: `floor` rests the prop's foot on the first
floor under `at`, `ceiling` hangs it by its top from the first ceiling over `at` (give `at` with its height, inside
the room), `wall` sets its back against the first wall from `at` toward `toward` and turns it to face into the room.
A model the game has not shown yet is put down first, measured, and then moved to rest, as one Undo step. The game
loads walls and floors near the camera only: snapping more than 80 m from the camera is refused with that reason.

`review` looks at an area (`at`, `radius`, 10 m by default) as a player would. It brings the camera there, then
answers up to five pictures from inside it: from near each corner toward the middle at eye height (each camera kept
short of the wall a ray meets) and one from under the ceiling or above. The MCP server measures each picture's
brightness, contrast, colour cast, detail and flat black or textureless areas, whole and for its upper part (walls
and ceiling) and lower part (the floor), and says what a player would notice: too dark, washed out, walls lit cold by
the sky only, walls and ceiling without texture, holes. With them come whether the area is indoors and how high its
ceiling is, the things and lights in it, and Validation's issues there, each with the id to fix, and a checklist to
judge the pictures by.

### Opening a closed building

Many of the game's buildings have no inside: a box fills them so nobody walks in, and the inside walls are drawn
black. `open_model` copies such a building into your assets without those boxes:

- `mesh`: the building's model, for example
  `objects/manmade/structures/religious/churches/stjames_church.cgf`. `list_only: true` lists its boxes (index, centre,
  size) and changes nothing.
- `remove`: the indexes of the boxes to take out (by default every box at least `thinnest` metres thick, 2.5).
- `materials`: sub-materials drawn as others, `{ "void": "plaster_b" }` paints the black inside with the plaster.
- `walls` (`{ from: [x, y], to: [x, y], base, height, thickness }`) and `floors` (`{ min, max, top, thickness }`), in
  the building's own frame: collision you add where the boxes kept the player out.

It writes `objects/kcds/opened/<name>_0.cgf`, `_1.cgf` ... (the drawn parts, one each) and `<name>_walls.cgf` (the
collision: the boxes kept are written as the game has them, turned ones turned). The answer's `meshes` lists
them. A box's `size` in `list_only` is its size along the building's x, y and z. Hide the level's building (`hide_level_named`) and place each of them
at the building's place and turn. Then the building is yours to furnish.

A call answers with what it made or changed, for example a `place`:

```json
{
  "result": {
    "id": 3,
    "kind": "prefab",
    "name": "structures/living/houses/house_1s_2r_a",
    "label": "house_1s_2r_a",
    "at": { "x": 130, "y": 2070, "z": 52.4 },
    "turn": 90
  },
  "state": { "open": true, "mode": "place", "placed": 17, "steps": 41 }
}
```

and a mistake answers with its reason, for example `Error: "castle_x" is not in the content (search it with
catalogue)` or `Error: the editor is not open (command "open")`.
