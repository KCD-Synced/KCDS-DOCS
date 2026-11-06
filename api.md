# API overview

All SDK features are in one global object, `kcds`, which every script has without an import. It is
divided into six domains:

| Domain            | Contents                                                     |
| ----------------- | ------------------------------------------------------------ |
| `kcds.core`       | events, chat, sharing between resources, runtime information |
| `kcds.math`       | vectors, quaternions and colours                             |
| `kcds.players`    | players, their stats, inventories and progression            |
| `kcds.characters` | NPCs, animals, dialogue, quests and shops                    |
| `kcds.world`      | places, objects and world systems                            |
| `kcds.ui`         | screens, input and the camera                                |

So you write `kcds.world.Door` for a door and `kcds.core.Events.on(...)` for an event listener. Text in
these docs often shortens a type to its name, such as `Door`, but code always needs the full path. To
shorten your code, unpack what you need at the top of a file:

```js
const { Events, Chat } = kcds.core;
const { Door } = kcds.world;
```

```lua
local Events, Chat = kcds.core.Events, kcds.core.Chat
local Door = kcds.world.Door
```

```cs
// The namespaces, which the C# template imports for every file:
using Kcds.Core; // Events, Chat
using Kcds.World; // Door
```

Standard JavaScript and web features such as `console`, `setTimeout` and `fetch` are still global.

Each type has a reference page listing its members, parameters and examples. A type may work in
`serverScripts`, in `clientScripts` or on both sides.
Coming from an older version? See [API changes](api-changes.md). <!-- only apichanges -->

## Scripting basics

| Type                                                                                       | Purpose                                         |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------- |
| [Events](/docs/api/events)                                                                 | game events and your own, in both directions    |
| [Chat](/docs/api/chat)                                                                     | chat messages and the default relay             |
| [Exports](/docs/api/exports), [Imports](/docs/api/imports), [Messages](/docs/api/messages) | sharing between resources on one side           |
| [SharedState](/docs/api/sharedstate)                                                       | values on entities and the server, sent to players |
| [Timers](/docs/api/timers), [Runtime](/docs/api/runtime)                                   | scheduling and runtime details                  |
| [Locales](/docs/api/locales)                                                               | text in each player's language                  |
| [process](/docs/api/process)                                                               | the part of Node's `process` server scripts can read |

## Players and characters

| Type                                                                                                                                | Purpose                                     |
| ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| [Player](/docs/api/player)                                                                                                          | a connected player, with body, stats, inventory and animations |
| [Entity](/docs/api/entity)                                                                                                          | what players, NPCs and horses have in common |
| [Npc](/docs/api/npc), [Horse](/docs/api/horse), [HorseLooks](/docs/api/horselooks), [Dog](/docs/api/dog), [PatrolRoute](/docs/api/patrolroute) | NPCs and animals                            |
| [Appearances](/docs/api/appearances), [PlayerStat](/docs/api/playerstat), [DerivedPlayerStat](/docs/api/derivedplayerstat)          | looks and stat names                        |
| [Animations](/docs/api/animations)                                                                                                  | the game's animations and their held tools  |

## World

| Type                                                                                                                                 | Purpose                               |
| ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------- |
| [World](/docs/api/world)                                                                                                             | time of day and weather               |
| [Area](/docs/api/area), [Marker](/docs/api/marker), [Blip](/docs/api/blip)                                                           | regions, ground markers and map icons |
| [Prop](/docs/api/prop), [PropGhost](/docs/api/propghost), [PropPlacer](/docs/api/propplacer), [GroundItem](/docs/api/grounditem), [Vfx](/docs/api/vfx) | objects and effects                   |
| [Door](/docs/api/door), [Gate](/docs/api/gate), [Stash](/docs/api/stash), [Cart](/docs/api/cart)                                     | things players use                    |
| [LevelEdit](/docs/api/leveledit), [WorldResource](/docs/api/worldresource), [Gizmo](/docs/api/gizmo)                                 | changing the level                    |
| [SiegeEngine](/docs/api/siegeengine), [SiegeLadder](/docs/api/siegeladder), [StonePile](/docs/api/stonepile)                         | siege                                 |

## Gameplay

| Type                                                                                                                                                                    | Purpose                               |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| [Inventory](/docs/api/inventory), [Items](/docs/api/items)                                                                                                              | what players carry                    |
| [Buffs](/docs/api/buffs), [Needs](/docs/api/needs), [Progression](/docs/api/progression)                                                                                | effects, needs and skills             |
| [Quest](/docs/api/quest), [Dialogue](/docs/api/dialogue), [Vendor](/docs/api/vendor)                                                                                    | quests, conversations and shops       |
| [Crafting](/docs/api/crafting), [Recipe](/docs/api/recipe), [CookPot](/docs/api/cookpot), [Gathering](/docs/api/gathering), [Dice](/docs/api/dice), [Pickpocketing](/docs/api/pickpocketing) | activities                            |
| [Trade](/docs/api/trade)                                                                                                                                                | one player's inventory as a shop for another |

## Hosting

- [Queue](/docs/api/queue): the join queue on a full server
- [Http](/docs/api/http): a resource's HTTP routes, next to the server's status API

## Server switches

[Camera](/docs/api/camera) for scripted shots, [ThirdPerson](/docs/api/thirdperson),
[Freecam](/docs/api/freecam), [Interactions](/docs/api/interactions),
[CombatMarker](/docs/api/combatmarker) and [RestrictedAreaMarker](/docs/api/restrictedareamarker).

## Client only

| Type                                                                                                                                                 | Purpose                                          |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| [Hud](/docs/api/hud), [ActionHint](/docs/api/actionhint), [Book](/docs/api/book), [Compass](/docs/api/compass), [WorldMap](/docs/api/worldmap), [Travel](/docs/api/travel) | on-screen information, the map and travel        |
| [Key](/docs/api/key), [Controls](/docs/api/controls)                                                                                                 | input                                            |
| [Web](/docs/api/web)                                                                                                                                 | HTML pages over the game                         |
| [NativeUI](/docs/api/nativeui), [NativeScreen](/docs/api/nativescreen), [NativeElement](/docs/api/nativeelement), [NativeClip](/docs/api/nativeclip) | the game's own interface                         |
| [NoClip](/docs/api/noclip), [CharacterCreator](/docs/api/charactercreator), [LocalNpc](/docs/api/localnpc), [LocalPlayer](/docs/api/localplayer), [Gizmos](/docs/api/gizmos) | the free camera, character creator and local objects |
| [Render](/docs/api/render)                                                                                                                           | pictures from any camera, unseen by the player   |
| [Follow](/docs/api/follow)                                                                                                                           | makes the local player follow someone            |
| [Audio](/docs/api/audio), [Assets](/docs/api/assets)                                                                                                 | game sounds and files a resource ships           |
| [Nameplates](/docs/api/nameplates), [EmoteWheel](/docs/api/emotewheel), [Discord](/docs/api/discord)                                                 | nameplate display, the emote wheel, Discord status |

## Math

[Vector2](/docs/api/vector2), [Vector3](/docs/api/vector3), [Vector4](/docs/api/vector4),
[Quaternion](/docs/api/quaternion) and [Color](/docs/api/color). See also [Math types](math.md).
