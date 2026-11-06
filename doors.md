# Doors, gates, graves and keys

## Doors

Doors belong to the level, so the server does not create them. It learns about a door when the first player's game
reports having built it, and from then on the server decides whether it is open or locked. Doors are kept per
dimension by the game entity's GUID, and their state is sent only when it changes.

```js
const { Door } = kcds.world;

kcds.core.Events.on('doorInteract', (player, door, action, keySide) => {
  if (door.guid === '00000000000000ab' && action === 'lockpick') {
    return false;
  }
});

const vaultDoor = Door.find('ab');
if (vaultDoor) {
  vaultDoor.locked = true;
  vaultDoor.setOpen(false);
}
```

```lua
local Door = kcds.world.Door

kcds.core.Events.on("doorInteract", function(player, door, action, keySide)
  if door.guid == "00000000000000ab" and action == "lockpick" then
    return false
  end
end)

local vaultDoor = Door.find("ab")
if vaultDoor then
  vaultDoor.locked = true
  vaultDoor:setOpen(false)
end
```

```cs
Events.On("doorInteract", (Player player, Door door, string action, JsValue keySide) =>
{
    if (door.Guid == "00000000000000ab" && action == "lockpick")
    {
        return false;
    }
    return (bool?)null;
});

var vaultDoor = Door.Find("ab");
if (vaultDoor != null)
{
    vaultDoor.Locked = true;
    vaultDoor.SetOpen(false);
}
```

### Finding doors

- `Door.all()` and `Door.getById(id)` return the doors the server knows, or `null` for an unknown id.
- `Door.find(guid, dimension?)` takes up to sixteen hex digits, with or without `0x`. It returns the door, or `null`
  if the server does not know it yet.
- `Door.closestTo(player)` asks the player's game which door they are at. It returns a promise of `{ found, reason,
  door }`, with `guid` and `name` when found.

### Properties

**Read only:** `id`, `guid` (sixteen lower-case hex digits), `name`, `dimension`, `lockKey`, `canBePicked`,
`builtLocked`, `position`, `open`, `side`, `lockedByScript`.

**`locked`** can also be written. Players cannot open a lock a script set.

**`lockLevel`** is the lock's difficulty, from 0 (easy) to 1 (hard). A number overrides the level's value for all
players in that dimension, and `null` restores it.

**`setOpen(open, side?)`** opens or closes it with nobody pushing, and opening also unlocks. It returns `false` if it
was already in that state.

### The `doorInteract` event

`doorInteract(player, door, action, keySide)` fires when a player uses a door, with the action `'open'`, `'close'`,
`'unlock'` or `'lockpick'`. The server first refuses what the game's rules forbid. Returning `false` refuses the
rest, and all players' copies of the door return to the server's state.

### The `doorDiscovered` event

`doorDiscovered(door, player)` fires the first time the server meets a level door in a run: a player's game in that
dimension built it and reported it. The server keeps no doors between runs, so after a restart each door is met
again. A lock or key set in the handler reaches the reporting player before they can use the door, so locks kept by
a script need no timer:

```js
const lockedDoors = new Set(['00000000000000ab']);

kcds.core.Events.on('doorDiscovered', (door) => {
  if (lockedDoors.has(door.guid)) {
    door.locked = true;
  }
});
```

```lua
local lockedDoors = { ["00000000000000ab"] = true }

kcds.core.Events.on("doorDiscovered", function(door)
  if lockedDoors[door.guid] then
    door.locked = true
  end
end)
```

```cs
var lockedDoors = new HashSet<string> { "00000000000000ab" };

Events.On("doorDiscovered", (Door door) =>
{
    if (lockedDoors.Contains(door.Guid))
    {
        door.Locked = true;
    }
});
```

### Removing a level door

`Door.remove(guid, dimension?)` takes a door and its collision out of a dimension for all players there. With no
dimension, it uses the global one, which means all dimensions. `door.destroy()` does the same in the door's own
dimension.

A removed door is not listed anywhere and cannot be used there. `Door.restore(guid, dimension?)` puts it back, and
`Door.removed(dimension?)` lists the removed ones.

```js
const { Door } = kcds.world;

const doorway = Door.find('41977678219215de');
Door.remove(doorway.guid, 7); // an open doorway in dimension 7

// Later:
Door.restore(doorway.guid, 7);
```

```lua
local Door = kcds.world.Door

local doorway = Door.find("41977678219215de")
Door.remove(doorway.guid, 7) -- an open doorway in dimension 7

-- Later:
Door.restore(doorway.guid, 7)
```

```cs
var doorway = Door.Find("41977678219215de")!;
Door.Remove(doorway.Guid, 7); // an open doorway in dimension 7

// Later:
Door.Restore(doorway.Guid, 7);
```

A removal is a level edit ([Level editing](level-editing.md)). It lasts while the server runs and the resource that
made it is started. Remember that the server only knows a door once a player's game has shown it.

### Adding a new door

`Door.spawn(definition, dimension?)` hangs a new door, in the global dimension by default. Players there can open,
close, unlock and pick it, under the same checks as level doors. New doors start closed, and later joiners get them
too.

```js
const guildDoor = kcds.world.Door.spawn({
  model: 'lvl1_door_a_left', // from Door.models(), by path or file name
  position: [1650.5, 2160, 35.4], // the hinge pivot
  rotation: [0, 0, 0.7071, 0.7071], // optional: x, y, z, w
  name: 'guild hall',
  locked: true,
  lock: 'boltAndKey', // 'none', 'bolt', 'boltAndKey' (default) or 'key'
  canBePicked: true,
  lockLevel: 0.8, // 0 to 1; leave out for the game's value
});
guildDoor.setOpen(true);
```

```lua
local guildDoor = kcds.world.Door.spawn({
  model = "lvl1_door_a_left", -- from Door.models(), by path or file name
  position = { 1650.5, 2160, 35.4 }, -- the hinge pivot
  rotation = { 0, 0, 0.7071, 0.7071 }, -- optional: x, y, z, w
  name = "guild hall",
  locked = true,
  lock = "boltAndKey", -- "none", "bolt", "boltAndKey" (default) or "key"
  canBePicked = true,
  lockLevel = 0.8, -- 0 to 1; leave out for the game's value
})
guildDoor:setOpen(true)
```

```cs
var guildDoor = Door.Spawn(new
{
    model = "lvl1_door_a_left", // from Door.Models(), by path or file name
    position = new[] { 1650.5, 2160, 35.4 }, // the hinge pivot
    rotation = new[] { 0, 0, 0.7071, 0.7071 }, // optional: x, y, z, w
    name = "guild hall",
    locked = true,
    @lock = "boltAndKey", // "none", "bolt", "boltAndKey" (default) or "key"
    canBePicked = true,
    lockLevel = 0.8, // 0 to 1; leave out for the game's value
});
guildDoor.SetOpen(true);
```

- `Door.models()` lists each leaf used in the shipped levels, as `{ path, name, hinge, placements }`. The hinge side
  is part of the mesh.
- `door.spawned` is true for a new door, and `door.model` gives its leaf. A level door has `model` too.
- `door.toJSON()` gives the spawn definition with the current lock state, or `null` for a level door. The server keeps nothing
  after a restart, so save this and hang the door again at startup.
- `door.destroy()` takes it down for good. The resource that hung it owns it, so it also goes when that resource
  stops.
- Its GUID is the server's own, `6b636473` plus a number, and stays the same on all machines until the server stops.
  Use `Door.find(door.guid, dimension)` to find it again.
- NPCs do not walk through a new door, because the level's navigation has no link for it.

Reference: [Door](/docs/api/door).

## Gates

Level gates, such as the drawbridge, portcullises and palisade gates of Trosky and Kuttenberg, are the same for all
players. The server owns each gate's state. Players' games move the gate there smoothly, and latecomers see it where it
is.

```js
const { Gate } = kcds.world;

for (const gate of Gate.all('trosecko')) {
  console.info(gate.key, gate.states);
}

const [firstGate] = Gate.all('trosecko');
firstGate.open(); // a barred gate has its bar removed first
firstGate.close({ bar: false }); // closes without replacing the bar
```

```lua
local Gate = kcds.world.Gate

for _, gate in ipairs(Gate.all("trosecko")) do
  print(gate.key, json.encode(gate.states))
end

local firstGate = Gate.all("trosecko")[1]
firstGate:open() -- a barred gate has its bar removed first
firstGate:close({ bar = false }) -- closes without replacing the bar
```

```cs
foreach (var gate in Gate.All("trosecko"))
{
    Log.Info($"{gate.Key} {string.Join(", ", gate.States)}");
}

var firstGate = Gate.All("trosecko")[0];
firstGate.Open(); // a barred gate has its bar removed first
firstGate.Close(new { bar = false }); // closes without replacing the bar
```

- **Finding:** `Gate.all(level?)` and `Gate.find(key)`. A gate's `key` stays the same across server starts.
- **State:** `state`, `states`, `isOpen`, `moving`, `secondsTo(state)`, `setState(state)`, `open()`, `close()`.
- **Bars:** `hasBar`, `barred`, `removeBar()`, and `placeBar()`, which leaves the gate shut.
- **Events:** `gateChanged`, `gateBarChanged`.

Reference: [Gate](/docs/api/gate).

## Graves

The level's graves, where a player digs with a shovel, are also the same for all players. Once dug with the game's
digging, a grave is open for everyone, including those joining later. Scripts can dig or refill one.

Players' games tell the server where graves are, so `Grave.all()` lists the ones in the server's level once a player has
been near them. Nothing is saved, and all graves are full again after a server start.

```js
const { Grave } = kcds.world;

kcds.core.Events.on('graveDug', (player, grave) => {
  kcds.core.Chat.sendToAll(`${player.nickname} dug out a grave.`);
  setTimeout(() => grave.reset(), 10 * 60 * 1000); // refills after ten minutes
});

const closest = Grave.nearest({ x: 3187.1, y: 495.9, z: 40.5 }, 50);
if (closest && !closest.dug) {
  closest.dig();
}
```

```lua
local Grave = kcds.world.Grave

kcds.core.Events.on("graveDug", function(player, grave)
  kcds.core.Chat.sendToAll(player.nickname .. " dug out a grave.")
  setTimeout(function() grave:reset() end, 10 * 60 * 1000) -- refills after ten minutes
end)

local closest = Grave.nearest({ x = 3187.1, y = 495.9, z = 40.5 }, 50)
if closest and not closest.dug then
  closest:dig()
end
```

```cs
Events.On("graveDug", (Player player, Grave grave) =>
{
    Chat.SendToAll($"{player.Nickname} dug out a grave.");
    Timers.SetTimeout(() => grave.Reset(), 10 * 60 * 1000); // refills after ten minutes
});

var closest = Grave.Nearest(new { x = 3187.1, y = 495.9, z = 40.5 }, 50);
if (closest != null && !closest.Dug)
{
    closest.Dig();
}
```

- **Static:** `Grave.all(near?)`, nearest first when `near` is given, `Grave.get(id)`, `Grave.nearest(position, radius?)`
  and `Grave.resetAll()`.
- **On a grave:** `id` (its location in whole centimetres, `"x,y,z"`, stable across starts), `position`, `dug`,
  `distanceTo(position)`, `dig()`, `reset()`.
- **Event:** `graveDug`.

Also: `Door.linkedTo(areaId)` lists the doors the level links to an area, each with `link` `'entrance'`, `'basic'` or
`'storage'`, even with no players near. On a gate, `toggle()` opens a shut one or shuts an open one, `openness` goes from
0 shut to 1 open while it moves, `openingTime` and `closingTime` are its seconds, and `barState` is the state in which
it is barred, or null.

Reference: [Door](/docs/api/door), [Gate](/docs/api/gate), [Grave](/docs/api/grave).

## Keys for a house

`area.lockKey` makes one key work on all the doors and chests in an area. Whoever carries it gets the game's Unlock
prompt, and a Lock hint on the U key at an unlocked door or chest. The server checks the key on each use.

```js
const house = kcds.world.Area.getById('006588875df00335');
if (house) {
  house.lockKey = 'key_home';
  house.setLocked(true);
}
```

```lua
local house = kcds.world.Area.getById("006588875df00335")
if house then
  house.lockKey = "key_home"
  house:setLocked(true)
end
```

```cs
var house = Area.GetById("006588875df00335");
if (house != null)
{
    house.LockKey = "key_home";
    house.SetLocked(true);
}
```

A door or chest can have its own key that takes priority, such as a supervisor's key for one room. Use `door.setKey(item)`
or `stash.keyClass`, and `null` or `''` returns to the area's key.

`player.keyBypass = true` lets one player, such as an admin, open everything the server holds without a key. Their game
shows those locks as unlocked, and nobody else's game changes.
