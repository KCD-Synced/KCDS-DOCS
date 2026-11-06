# Time, weather and the world

`kcds.world.World` holds the time of day, sky, rain and wind on the server, and all players' games
follow it. A new server starts at 9:00, and by default 15 game seconds pass per real second. The clock
never goes backwards.

```js
const { World } = kcds.world;

World.setTimeScale(30);
World.setHour(18); // the next 18:00, which may be tomorrow
World.setWeather('cloudless_sunny', 60); // fade over 60 game seconds
World.setRain(0.5);
World.setWind({ x: 3, y: 0, z: 0 });

kcds.core.Events.on('worldDayChange', (day) => {
  console.info(`Day ${day} begins`);
});
```

```ts
const { World } = kcds.world;

World.setTimeScale(30);
World.setHour(18); // the next 18:00, which may be tomorrow
World.setWeather('cloudless_sunny', 60); // fade over 60 game seconds
World.setRain(0.5);
World.setWind({ x: 3, y: 0, z: 0 });

kcds.core.Events.on('worldDayChange', (day) => {
  console.info(`Day ${day} begins`);
});
```

```lua
local World = kcds.world.World

World.setTimeScale(30)
World.setHour(18) -- the next 18:00, which may be tomorrow
World.setWeather("cloudless_sunny", 60) -- fade over 60 game seconds
World.setRain(0.5)
World.setWind({ x = 3, y = 0, z = 0 })

kcds.core.Events.on("worldDayChange", function(day)
  print("Day " .. day .. " begins")
end)
```

```cs
World.SetTimeScale(30);
World.SetHour(18); // the next 18:00, which may be tomorrow
World.SetWeather("cloudless_sunny", 60); // fade over 60 game seconds
World.SetRain(0.5);
World.SetWind(new { x = 3, y = 0, z = 0 });

Events.On("worldDayChange", (double day) =>
{
    Log.Info($"Day {day} begins");
});
```

## Changing the world

Each setter returns `true` when it worked and `false` when a value is out of range.

| Call                               | Accepted values                                                                 |
| ---------------------------------- | ------------------------------------------------------------------------------- |
| `setTime(day, hour)`               | whole day from 0, hour from 0 up to (not including) 24; a past time returns `false` |
| `setHour(hour)`                    | winds the clock on to that hour, which can be tomorrow                          |
| `setTimeScale(scale)`              | 0 to 200; 0 stops the clock                                                     |
| `setWeather(preset, seconds?)`     | name of 1 to 64 characters from `A-Z a-z 0-9 _ . -`; fade of 0 to 21600 game seconds |
| `setRain(intensity, amount?)`      | both 0 to 1; `amount` defaults to 1                                             |
| `setWind({ x, y, z })`             | metres per second; a missing component is 0; above 50 m/s is refused            |

`setWeather` also returns `false` while an earlier fade is still running.

## Reading the world

| Group    | Properties                                                     |
| -------- | -------------------------------------------------------------- |
| Time     | `day`, `hour`, `timeScale`                                     |
| Weather  | `weather` (target preset), `lastWeather`, `weatherFade`, `weatherLeft` |
| Rain     | `rainStrength`, `rainLevel`, `wetness` (0 to 3), `puddles` (0 to 1) |
| Wind     | `wind` (a `Vector3`), `windSpeed`                              |

## Events

- `worldDayChange(day)`: midnight passed.
- `worldWeatherChange(preset, previous, seconds)`: a weather fade started.

## Items placed by the level

Tankards, tools, goods on shop counters and similar items are shared between all players. The first
player in reach to pick one up gets it, and it then disappears for everyone, including those who join
later. Taken items come back when the server restarts.

Return `false` from `worldItemPickup(player, { item, name, position })` to refuse a pick-up.

You can also remove these items altogether:

```js
kcds.world.World.setLevelItemsEnabled(false); // hidden for all, nothing can be picked up
```

```ts
kcds.world.World.setLevelItemsEnabled(false); // hidden for all, nothing can be picked up
```

```lua
kcds.world.World.setLevelItemsEnabled(false) -- hidden for all, nothing can be picked up
```

```cs
World.SetLevelItemsEnabled(false); // hidden for all, nothing can be picked up
```

`setLevelItemsEnabled(true)` brings them back and `World.levelItemsEnabled` reads the setting. To start
the server with them off, put `{ "enabled": false }` in `resources/level-items.json`.

To take the items from one spot only, such as the bowls by a camp fire or the goods on one stall, give a
place. Every game hides the level's items within `radius` metres of `position`, with the pile or stack
look the level draws for them, also for players who join later, so nobody can pick them up. `item`, a
class name or GUID, takes only that class. The returned handle's `restore()` brings them back; places
last until then or until the server restarts, up to 1024 at once.

```js
const { World } = kcds.world;

const apples = World.removeLevelItems({ position: stall, radius: 2, item: 'apple' });
// later
apples.restore();
```

```ts
const { World } = kcds.world;

const apples = World.removeLevelItems({ position: stall, radius: 2, item: 'apple' });
// later
apples.restore();
```

```lua
local World = kcds.world.World

local apples = World.removeLevelItems({ position = stall, radius = 2, item = "apple" })
-- later
apples:restore()
```

```cs
var apples = World.RemoveLevelItems(new { position = stall, radius = 2, item = "apple" });
// later
apples.Call("restore");
```

`setLevelItemsEnabled(false)` also hides the pile and stack looks of the level's items.

To hide only some of them, such as the sausages on one drying rack, ask a player's game which level
entities are near and hide those with a level edit. `World.levelEntitiesAround(player, center, radius,
{ max })`, up to 64 m and 32 entities, answers each with its `guid`, `name`, `class`, `position`,
`distance` and the `levelObject` that `LevelEdit.apply` takes. That game must have the place streamed
in, so ask a player standing near it:

```js
const { World, LevelEdit } = kcds.world;

const { entities } = await World.levelEntitiesAround(player, rack, 2);
for (const found of entities) {
  if (found.name.startsWith('sausage') && found.levelObject) {
    LevelEdit.apply({ ...found.levelObject, hidden: true });
  }
}
```

```ts
const { World, LevelEdit } = kcds.world;

const { entities } = await World.levelEntitiesAround(player, rack, 2);
for (const found of entities) {
  if (found.name.startsWith('sausage') && found.levelObject) {
    LevelEdit.apply({ ...found.levelObject, hidden: true });
  }
}
```

```lua
local World, LevelEdit = kcds.world.World, kcds.world.LevelEdit

local around = World.levelEntitiesAround(player, rack, 2):await()
for _, found in ipairs(around.entities) do
  if string.sub(found.name, 1, 7) == "sausage" and found.levelObject then
    -- the level object as it is, hidden
    local hidden = {}
    for key, value in pairs(found.levelObject) do
      hidden[key] = value
    end
    hidden.hidden = true
    LevelEdit.apply(hidden)
  end
end
```

```cs
var around = await World.LevelEntitiesAround(player, rack, 2);
foreach (var found in around["entities"])
{
    if (found["name"].AsString()!.StartsWith("sausage") && !found["levelObject"].IsNull)
    {
        // the level object as it is, hidden
        var hidden = new Dictionary<string, object?>();
        foreach (var key in found["levelObject"].Keys)
        {
            hidden[key] = found["levelObject"][key];
        }
        hidden["hidden"] = true;
        LevelEdit.Apply(hidden);
    }
}
```

## Renaming places

`setPlaceName(place, name)` renames a town or place for all current and future players, and `null`
restores the game's name. `World.places()` lists the ids.

```js
kcds.world.World.setPlaceName('troskovice', 'Zahradka');
```

```ts
kcds.world.World.setPlaceName('troskovice', 'Zahradka');
```

```lua
kcds.world.World.setPlaceName("troskovice", "Zahradka")
```

```cs
World.SetPlaceName("troskovice", "Zahradka");
```

This replaces the place's quest location and settlement name, with the same name in every language.
Names written inside a sentence, such as a dialogue line or quest text, stay as the game has them.

Also: `World.landmarks()` lists the level's points of interest as `{ name, group, level, position }`, and
`World.replicasAround(position, radius, dimension)` gives the server's entities around a point, closest first (players,
horses, NPCs, dogs, props, dropped items, doors and stashes), with no client needed.

Reference: [World](/docs/api/world).
