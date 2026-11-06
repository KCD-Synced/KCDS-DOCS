# KCDS documentation

With KCDS, you can run a multiplayer server for Kingdom Come: Deliverance II and extend it with resources. These pages are
the public documentation, published at <https://kcdsynced.com/docs> along with the API reference generated from the runtimes.
They cover how to use KCDS, not how it is built. All pages are in English, and links to API pages are written
`/docs/api/<type>`.

Found a mistake or something unclear? Open an issue or a pull request in this repository. The text is
CC BY-ND 4.0 and the code examples are free to use in your resources (MIT-0): see [LICENSE.md](LICENSE.md).

## Get started

- [Getting started](getting-started.md), [Your first resource](first-resource.md)
- [Project structure](project-structure.md), [TypeScript](typescript.md), <!-- without lua cs -->
- [Project structure](project-structure.md), [TypeScript](typescript.md), [Lua](lua.md), [C#](csharp.md), <!-- only lua cs -->
  [Trying code quickly](playground.md), [Debugging resources](debugging.md)

## Hosting

- [Running a server](server.md), [How players connect](players-connecting.md), [Join queue and whitelist](join-queue.md)
- [Server rules](server-rules.md), [A legal copy of the game](licence.md)
- [Discord presence](discord.md)

## Scripting

- [Resources](resources.md), [Client resources](client-resources.md), [API overview](api.md),
  [API changes](api-changes.md)
- [Events](events.md), [Imports and exports](imports-exports.md), [Messages](sharing.md)
- [Shared state](state.md), [Chat and translated texts](chat-locales.md)
- [Timers, logging and the runtime](timers.md), [Math types](math.md)

## Players

- [Players](players.md), [Restricting a player](player-restrictions.md), [Carrying and attaching players](carrying.md)
- [Skills, perks and needs](progression.md), [Custom perks](custom-perks.md), [Custom buffs](custom-buffs.md)
- [Animations](animations.md), [Nameplates](nameplates.md), [Voice chat](voice.md), [Following someone](follow.md)

## NPCs and animals

- [NPCs](npcs.md), [Horses and dogs](horses-dogs.md)

## World

- [Time, weather and the world](world.md), [Lights](lights.md), [Particle effects](vfx.md)
- [Areas](areas.md), [Markers and blips](markers.md)
- [Felling trees](trees.md), [Editing the level](level-editing.md)

## Objects

- [Props](props.md), [Doors, gates, graves and keys](doors.md), [Stashes](stashes.md), [Carts](carts.md),
  [Siege](siege.md)

## Gameplay

- [Inventory and items](inventory.md), [Equipment](equipment.md)
- [Quests and conversations](quests-dialogue.md), [Shops and player trade](vendors.md)
- [Crafting, cooking and gathering](crafting.md), [Dice and pickpocketing](activities.md),
  [Tabletop games](tabletop.md)

## Client and UI

- [HUD and messages](client-ui.md), ["Press [key] to ..." hints](action-hints.md), [Books and letters](books.md),
  [Map and compass](world-map.md), [Shooting contests and races](shooting-contests.md),
  [Cutscenes and the character's remarks](story-and-remarks.md)
- [Web views](web-views.md), [The game's own interface](native-ui.md)
- [Keys and controls](input.md), [Camera](client-camera.md), [What the player looks at](looking-at.md),
  [Pictures from any camera](pictures.md), [Character creator](character-creator.md),
  [Local NPCs and the local player](local-figures.md)
- [Sounds and game files](audio-and-assets.md)

## Tools

- [World Architect](world-architect.md)
- [Asset/Animation Studio](asset-studio.md) <!-- only studio -->
