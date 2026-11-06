# Map and compass

The game's map and compass on this player's screen, from client scripts.

**Waypoints and events:** `WorldMap.getWaypoint()`, `shiftWaypoint(position)`, `clearWaypoint()` and
`setMapCategory('Shrine', false)`, with the events `mapOpened`, `mapClosed` and `mapWaypointSet`.
`WorldMap.mapCategoryShown(type)` reads a mark category.

**Fog of war:** by default the server lifts the fog and discovers all locations as the world loads.
`WorldMap.clearAllFog()` clears fog again if a level load brought it back. A server with `revealMap` off keeps the
fog, and `WorldMap.clearFog(position)` clears it around a point with the game's reveal radius, as if walking there. A
resource tracking where a character has been calls this after loading the character.

**Discovery:** `WorldMap.discover(id)` silently marks a place as discovered, with no toast or XP. The id comes from
`locationDiscovered` or `poiDiscovered`. Call this after loading the character, if your resource keeps their
discoveries. Marking a point of interest fires `poiDiscovered`, but a location fires nothing.

**Compass and travel:** `kcds.ui.Compass.visible = false` hides the compass and `Compass.markerTotal()` counts the
compass marks. `kcds.world.Travel.active` is true during fast travel.

Map icons for everyone come from the server; see [Markers and blips](markers.md). `blip.legend = false` keeps a blip
shown when its type is off in the map legend.

Reference: [WorldMap](/docs/api/worldmap), [Compass](/docs/api/compass), [Travel](/docs/api/travel).
