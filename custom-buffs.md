# Custom buffs

A resource can create its own buffs, with one of the game's status icons, a timer, stat changes and an optional tick. The server
runs the buff, and the player sees it with its timer in their status list. `Buffs.register` has two forms.

| Form                            | Best for                                                                   |
| ------------------------------- | -------------------------------------------------------------------------- |
| `Buffs.register(id, definition)` | full buffs with a GUID, stacks and modifiers, including the game's own stats |
| `Buffs.register(definition)`    | a simple buff owned by your resource, with an `onTick` callback            |

## Full form

```js
const { Buffs } = kcds.players;
const { Events } = kcds.core;

const blessing = Buffs.register('myres:blessing', {
  name: 'Blessing',
  description: 'A priest blessed you.',
  visual: 'well_rested',
  duration: 120,
  tickInterval: 10,
  stackable: true,
  maxStacks: 3,
  modifiers: { stamina: 20, health: { operation: 'add', value: 5 } },
});

Events.on('playerCustomBuffApply', (player, buff) => {
  console.info(`${buff.displayName} on ${player.nickname}`);
});
Events.on('playerCustomBuffTick', (player, buff) => {
  player.addStat('health', 1);
});
Events.on('playerCustomBuffRemove', (player, buff, reason) => {
  console.info(`${buff.name}: ${reason}`);
});

player.addBuff(blessing); // the GUID, or use the id
```

```ts
const { Buffs } = kcds.players;
const { Events } = kcds.core;

const blessing = Buffs.register('myres:blessing', {
  name: 'Blessing',
  description: 'A priest blessed you.',
  visual: 'well_rested',
  duration: 120,
  tickInterval: 10,
  stackable: true,
  maxStacks: 3,
  modifiers: { stamina: 20, health: { operation: 'add', value: 5 } },
});

Events.on('playerCustomBuffApply', (player, buff) => {
  console.info(`${buff.displayName} on ${player.nickname}`);
});
Events.on('playerCustomBuffTick', (player, buff) => {
  player.addStat('health', 1);
});
Events.on('playerCustomBuffRemove', (player, buff, reason) => {
  console.info(`${buff.name}: ${reason}`);
});

player.addBuff(blessing); // the GUID, or use the id
```

```lua
local Buffs = kcds.players.Buffs
local Events = kcds.core.Events

local blessing = Buffs.register("myres:blessing", {
  name = "Blessing",
  description = "A priest blessed you.",
  visual = "well_rested",
  duration = 120,
  tickInterval = 10,
  stackable = true,
  maxStacks = 3,
  modifiers = { stamina = 20, health = { operation = "add", value = 5 } },
})

Events.on("playerCustomBuffApply", function(player, buff)
  print(buff.displayName .. " on " .. player.nickname)
end)
Events.on("playerCustomBuffTick", function(player, buff)
  player:addStat("health", 1)
end)
Events.on("playerCustomBuffRemove", function(player, buff, reason)
  print(buff.name .. ": " .. reason)
end)

player:addBuff(blessing) -- the GUID, or use the id
```

```cs
var blessing = Buffs.Register("myres:blessing", new
{
    name = "Blessing",
    description = "A priest blessed you.",
    visual = "well_rested",
    duration = 120,
    tickInterval = 10,
    stackable = true,
    maxStacks = 3,
    modifiers = new { stamina = 20, health = new { operation = "add", value = 5 } },
});

Events.On("playerCustomBuffApply", (Player player, JsValue buff) =>
{
    Log.Info($"{buff["displayName"]} on {player.Nickname}");
});
Events.On("playerCustomBuffTick", (Player player, JsValue buff) =>
{
    player.AddStat("health", 1);
});
Events.On("playerCustomBuffRemove", (Player player, JsValue buff, string reason) =>
{
    Log.Info($"{buff["name"]}: {reason}");
});

player.AddBuff(blessing); // the GUID, or use the id
```

It returns the buff's GUID, which comes from the id and so is the same at each start. Registering the same definition under the same id
again returns the same GUID. A different definition throws `Custom buff myres:blessing is already registered with another definition`.

| Field          | Meaning                                                                  |
| -------------- | ------------------------------------------------------------------------ |
| `id`           | 1 to 128 characters of `A-Z a-z 0-9 : _ . -`; not the name of a game buff |
| `name`         | required; the text the player sees, up to 256 bytes                      |
| `description`  | optional text, up to 4096 bytes                                          |
| `visual`       | required; the game buff, by name or GUID, whose status icon it borrows   |
| `image`        | optional; a `.dds` your resource ships to clients, shown in place of the visual's icon |
| `duration`     | seconds, above 0 and up to 7 days; `-1` (default) has no end             |
| `tickInterval` | 0.1 to 86400 seconds; `0` (default) has no ticks                         |
| `stackable`    | whether a player can have several instances; default `false`             |
| `maxStacks`    | 1 to 16 (default 1): how many instances a stackable buff keeps           |
| `refresh`      | at the limit, `true` (default) replaces the oldest; `false` ignores the new one |
| `modifiers`    | `{ stat: points }` or `{ stat: { operation, value } }`; `Buffs.modifiers` lists the stats |

An unknown key, wrong value, or `visual` that is not a game buff throws `Invalid custom buff: ...`.

### Your own picture

`image` gives the buff a picture of its own in the status list. It is the path of a `.dds` that your resource
ships to clients, listed in the manifest's `files`, such as `"files": ["icons/*.dds"]`. The game's buff icons are
64 x 64 pixels; a picture of another size must be an uncompressed 32-bit `.dds`, which clients scale to 64 x 64.
A file the resource does not ship, or one that does not fit, throws when the buff is registered.

The player's client loads the picture the first time the buff shows. If it has not arrived after five
seconds, the buff shows with the icon of its `visual` instead.

```js
const thirst = kcds.players.Buffs.register('needs:thirst', {
  name: 'Thirst',
  visual: 'hangover',
  image: 'icons/thirst.dds',
  duration: 300,
  modifiers: { staminaRegeneration: { operation: 'multiply', value: 0.8 } },
});

player.addBuff(thirst);
```

```ts
const thirst = kcds.players.Buffs.register('needs:thirst', {
  name: 'Thirst',
  visual: 'hangover',
  image: 'icons/thirst.dds',
  duration: 300,
  modifiers: { staminaRegeneration: { operation: 'multiply', value: 0.8 } },
});

player.addBuff(thirst);
```

```lua
local thirst = kcds.players.Buffs.register("needs:thirst", {
  name = "Thirst",
  visual = "hangover",
  image = "icons/thirst.dds",
  duration = 300,
  modifiers = { staminaRegeneration = { operation = "multiply", value = 0.8 } },
})

player:addBuff(thirst)
```

```cs
var thirst = Buffs.Register("needs:thirst", new
{
    name = "Thirst",
    visual = "hangover",
    image = "icons/thirst.dds",
    duration = 300,
    modifiers = new { staminaRegeneration = new { operation = "multiply", value = 0.8 } },
});

player.AddBuff(thirst);
```

### Modifiers

| Operation          | Effect                                                      |
| ------------------ | ----------------------------------------------------------- |
| `'add'` (a number) | adds the points                                             |
| `'multiply'`       | multiplies the stat by `value`, 0 or more; `1.5` adds half again |
| `'set'`            | sets the stat to `value`                                    |

Modifiers work on three groups of stats.

**Direct stats:** `health`, `stamina`, `exhaust` and `hunger`. An instance works out its change from the player's last reported value when
it lands, and takes back exactly that when it ends. `stamina: { operation: 'multiply', value: 1.5 }` on 50 stamina adds 25, then later takes
25 away. Three stacks of `stamina: 20` give 60, and leave nothing behind.

**Rates per second:** `healthRegeneration` gives health, `hungerDrain` takes hunger, and `energyDrain` adds exhaust. These accept a number of 0
or more, or `'add'`, and apply each second while an instance lasts. What they changed stays changed.

**The game's own stats:** `maxStamina`, `carryCapacity`, `staminaRegeneration`, `runSpeed`, `attackSpeed`, `charisma`, `strength`, `agility`,
`vitality`, `speech`, and skills by track name, such as `weapon_sword`, `fencing` and `stealth`. These go through the game's buff system. The
game applies them once, however many stacks there are, while the buff is on the status list. Derived values follow, so more strength lets a
player carry more.

**Getting dirty:** `clothDirtying` scales how fast walking dirties the player's clothes and body, as the game does it.
`multiply` 2 is twice as fast and 0 stops it, as the game's own no-dirt buff does. For a lasting rate with no status
entry, use `player.setGrimeRate(rate)`.

**Being noticed:** `noise`, `visibility` and `conspicuousness` are how loud the player is, how well NPCs see them and how much
they stand out. The game's own crouch is `noise: { operation: 'multiply', value: 0.5 }`, and its Slim fit perk, which quiets
worn armour, is `0.8`. Rattling armour is a buff that multiplies `noise` above 1 while the player wears it:

```js
const rattling = kcds.players.Buffs.register('myres:rattling_armour', {
  name: 'Rattling armour',
  visual: 'well_rested',
  modifiers: { noise: { operation: 'multiply', value: 1.4 } },
});
```

```ts
const rattling = kcds.players.Buffs.register('myres:rattling_armour', {
  name: 'Rattling armour',
  visual: 'well_rested',
  modifiers: { noise: { operation: 'multiply', value: 1.4 } },
});
```

```lua
local rattling = kcds.players.Buffs.register("myres:rattling_armour", {
  name = "Rattling armour",
  visual = "well_rested",
  modifiers = { noise = { operation = "multiply", value = 1.4 } },
})
```

```cs
var rattling = Buffs.Register("myres:rattling_armour", new
{
    name = "Rattling armour",
    visual = "well_rested",
    modifiers = new { noise = new { operation = "multiply", value = 1.4 } },
});
```

**Wounds:** `attackSpeed: { operation: 'multiply', value: 0.8 }` makes every swing take 80 % of its speed, as the game's own arm wound does.
`noSprint: true` keeps the player from sprinting while the buff lasts, as the game's leg wounds do, without touching the movement mode that
other scripts set.

```js
const { Buffs } = kcds.players;

const brokenArm = Buffs.register('myres:broken_arm', {
  name: 'Broken arm',
  visual: 'well_rested',
  modifiers: { attackSpeed: { operation: 'multiply', value: 0.8 } },
});
const brokenLeg = Buffs.register('myres:broken_leg', {
  name: 'Broken leg',
  visual: 'well_rested',
  modifiers: { noSprint: true, runSpeed: -0.5 },
});

kcds.core.Events.on('playerDamage', (player, attacker, amount, bodyPart) => {
  if (amount > 40 && (bodyPart === 'leftArm' || bodyPart === 'rightArm')) {
    player.addBuff(brokenArm);
  }
  if (amount > 40 && (bodyPart === 'leftLeg' || bodyPart === 'rightLeg')) {
    player.addBuff(brokenLeg);
  }
});
```

```ts
const { Buffs } = kcds.players;

const brokenArm = Buffs.register('myres:broken_arm', {
  name: 'Broken arm',
  visual: 'well_rested',
  modifiers: { attackSpeed: { operation: 'multiply', value: 0.8 } },
});
const brokenLeg = Buffs.register('myres:broken_leg', {
  name: 'Broken leg',
  visual: 'well_rested',
  modifiers: { noSprint: true, runSpeed: -0.5 },
});

kcds.core.Events.on('playerDamage', (player, attacker, amount, bodyPart) => {
  if (amount > 40 && (bodyPart === 'leftArm' || bodyPart === 'rightArm')) {
    player.addBuff(brokenArm);
  }
  if (amount > 40 && (bodyPart === 'leftLeg' || bodyPart === 'rightLeg')) {
    player.addBuff(brokenLeg);
  }
});
```

```lua
local Buffs = kcds.players.Buffs

local brokenArm = Buffs.register("myres:broken_arm", {
  name = "Broken arm",
  visual = "well_rested",
  modifiers = { attackSpeed = { operation = "multiply", value = 0.8 } },
})
local brokenLeg = Buffs.register("myres:broken_leg", {
  name = "Broken leg",
  visual = "well_rested",
  modifiers = { noSprint = true, runSpeed = -0.5 },
})

kcds.core.Events.on("playerDamage", function(player, attacker, amount, bodyPart)
  if amount > 40 and (bodyPart == "leftArm" or bodyPart == "rightArm") then
    player:addBuff(brokenArm)
  end
  if amount > 40 and (bodyPart == "leftLeg" or bodyPart == "rightLeg") then
    player:addBuff(brokenLeg)
  end
end)
```

```cs
var brokenArm = Buffs.Register("myres:broken_arm", new
{
    name = "Broken arm",
    visual = "well_rested",
    modifiers = new { attackSpeed = new { operation = "multiply", value = 0.8 } },
});
var brokenLeg = Buffs.Register("myres:broken_leg", new
{
    name = "Broken leg",
    visual = "well_rested",
    modifiers = new { noSprint = true, runSpeed = -0.5 },
});

Events.On("playerDamage", (Player player, Player attacker, double amount, string bodyPart) =>
{
    if (amount > 40 && (bodyPart == "leftArm" || bodyPart == "rightArm"))
    {
        player.AddBuff(brokenArm);
    }
    if (amount > 40 && (bodyPart == "leftLeg" || bodyPart == "rightLeg"))
    {
        player.AddBuff(brokenLeg);
    }
});
```

A custom buff cannot change maximum health or armor rating. The game has no buff modifier that raises them. Its buff table never changes them,
and its `mhs` and `bba` codes moved neither stat when tested.

```js
const { Buffs } = kcds.players;

Buffs.register('myres:giant', {
  name: 'Giant',
  visual: 'well_rested',
  duration: 600,
  modifiers: { maxStamina: { operation: 'multiply', value: 1.5 }, carryCapacity: 100, charisma: 5 },
});

Buffs.register('myres:marathon', {
  name: 'Marathon',
  visual: 'well_rested',
  duration: 600,
  modifiers: { hungerDrain: 0.2, energyDrain: 0.1 },
});

Buffs.register('myres:feast', {
  name: 'Feast',
  visual: 'well_rested',
  duration: 300,
  modifiers: { hunger: { operation: 'set', value: 100 }, healthRegeneration: 0.5 },
});
```

```ts
const { Buffs } = kcds.players;

Buffs.register('myres:giant', {
  name: 'Giant',
  visual: 'well_rested',
  duration: 600,
  modifiers: { maxStamina: { operation: 'multiply', value: 1.5 }, carryCapacity: 100, charisma: 5 },
});

Buffs.register('myres:marathon', {
  name: 'Marathon',
  visual: 'well_rested',
  duration: 600,
  modifiers: { hungerDrain: 0.2, energyDrain: 0.1 },
});

Buffs.register('myres:feast', {
  name: 'Feast',
  visual: 'well_rested',
  duration: 300,
  modifiers: { hunger: { operation: 'set', value: 100 }, healthRegeneration: 0.5 },
});
```

```lua
local Buffs = kcds.players.Buffs

Buffs.register("myres:giant", {
  name = "Giant",
  visual = "well_rested",
  duration = 600,
  modifiers = { maxStamina = { operation = "multiply", value = 1.5 }, carryCapacity = 100, charisma = 5 },
})

Buffs.register("myres:marathon", {
  name = "Marathon",
  visual = "well_rested",
  duration = 600,
  modifiers = { hungerDrain = 0.2, energyDrain = 0.1 },
})

Buffs.register("myres:feast", {
  name = "Feast",
  visual = "well_rested",
  duration = 300,
  modifiers = { hunger = { operation = "set", value = 100 }, healthRegeneration = 0.5 },
})
```

```cs
Buffs.Register("myres:giant", new
{
    name = "Giant",
    visual = "well_rested",
    duration = 600,
    modifiers = new { maxStamina = new { operation = "multiply", value = 1.5 }, carryCapacity = 100, charisma = 5 },
});

Buffs.Register("myres:marathon", new
{
    name = "Marathon",
    visual = "well_rested",
    duration = 600,
    modifiers = new { hungerDrain = 0.2, energyDrain = 0.1 },
});

Buffs.Register("myres:feast", new
{
    name = "Feast",
    visual = "well_rested",
    duration = 300,
    modifiers = new { hunger = new { operation = "set", value = 100 }, healthRegeneration = 0.5 },
});
```

### Events and reading

- `playerCustomBuffApply` fires for each instance.
- `playerCustomBuffTick` fires for each instance and interval.
- `playerCustomBuffRemove` gives a reason: `expired`, `server` (from `removeBuff`, which takes all instances), `refreshed` (a new stack replaced
  the oldest) or `disconnect`.
- `addBuff` returns `false` when the buff is at its limit and does not refresh.
- In `player.buffs`, each instance has its own entry with `custom: true`, `displayName` and `description`. Game buffs have `custom: false` and
  `null` for those texts.

The buff these events receive is what `Buffs.find` returns:

```json
{
  "name": "myres:blessing",
  "class": "custom",
  "aiTag": null,
  "duration": 120,
  "custom": true,
  "displayName": "Blessing",
  "description": "A priest blessed you."
}
```

## Simple form

```js
const { Buffs } = kcds.players;

const warmth = Buffs.register({
  name: 'warmth',
  label: 'Warm',
  icon: 'hangover',
  duration: 120,
  effects: { stats: { stamina: 20 }, tickStats: { health: 1 } },
  tickSeconds: 5,
  onTick: (player, buff) => console.info(player.nickname, buff.name),
});

kcds.core.Events.on('playerBuffRemoved', (player, buff, reason) => {
  if (buff === 'warmth') {
    kcds.core.Chat.sendToAll(`The warmth wore off (${reason})`);
  }
});

player.addBuff(warmth); // or 'warmth'
player.hasBuff(warmth);
player.removeBuff(warmth);
```

```ts
const { Buffs } = kcds.players;

const warmth = Buffs.register({
  name: 'warmth',
  label: 'Warm',
  icon: 'hangover',
  duration: 120,
  effects: { stats: { stamina: 20 }, tickStats: { health: 1 } },
  tickSeconds: 5,
  onTick: (player: kcds.players.Player, buff: { name: string }) => console.info(player.nickname, buff.name),
});

kcds.core.Events.on('playerBuffRemoved', (player, buff, reason) => {
  if (buff === 'warmth') {
    kcds.core.Chat.sendToAll(`The warmth wore off (${reason})`);
  }
});

player.addBuff(warmth); // or 'warmth'
player.hasBuff(warmth);
player.removeBuff(warmth);
```

```lua
local Buffs = kcds.players.Buffs

local warmth = Buffs.register({
  name = "warmth",
  label = "Warm",
  icon = "hangover",
  duration = 120,
  effects = { stats = { stamina = 20 }, tickStats = { health = 1 } },
  tickSeconds = 5,
  onTick = function(player, buff) print(player.nickname, buff.name) end,
})

kcds.core.Events.on("playerBuffRemoved", function(player, buff, reason)
  if buff == "warmth" then
    kcds.core.Chat.sendToAll("The warmth wore off (" .. reason .. ")")
  end
end)

player:addBuff(warmth) -- or "warmth"
player:hasBuff(warmth)
player:removeBuff(warmth)
```

```cs
var warmth = Buffs.Register(new
{
    name = "warmth",
    label = "Warm",
    icon = "hangover",
    duration = 120,
    effects = new { stats = new { stamina = 20 }, tickStats = new { health = 1 } },
    tickSeconds = 5,
    onTick = new Action<Player, JsValue>((player, buff) => Log.Info($"{player.Nickname} {buff["name"]}")),
});

Events.On("playerBuffRemoved", (Player player, string buff, string reason) =>
{
    if (buff == "warmth")
    {
        Chat.SendToAll($"The warmth wore off ({reason})");
    }
});

player.AddBuff(warmth); // or "warmth"
player.HasBuff(warmth);
player.RemoveBuff(warmth);
```

| Field                  | Meaning                                                                            |
| ---------------------- | ---------------------------------------------------------------------------------- |
| `name`                 | 1 to 64 characters of `A-Z a-z 0-9 : _ . -`; not a game buff or one already registered |
| `icon`                 | the name of a game buff, as `Buffs.find` spells it, whose status icon it uses      |
| `label`                | the text in the status list; `name` by default                                     |
| `duration`             | seconds, above 0 and up to 7 days; `-1` has no end                                 |
| `effects.stats`        | stat changes, as for `player.addStat`, applied each time it is added               |
| `effects.tickStats`    | stat changes on each tick                                                          |
| `tickSeconds`          | the tick interval, at least 1 second; needs `onTick` or `effects.tickStats`        |
| `onTick(player, buff)` | called every `tickSeconds` while the buff lasts; missed ticks are not made up      |

It returns a frozen description, with `name`, `icon`, `label`, `duration` and `tickSeconds`, which `player.addBuff`, `removeBuff` and `hasBuff` take
in place of the name. The buff belongs to your resource and goes when it stops, including from players.

**How it behaves:**

- `addBuff` applies at once, so `hasBuff` is `true` straight away, and `playerBuffAdded` fires with the source `server`.
- Adding a buff the player already has restarts its timer and applies its effects again, with no events.
- When it ends, `playerBuffRemoved` fires with the reason `expired`, or `server` for `removeBuff`.
- In `player.buffs` it has `class: 'custom'` and `source: 'server'`.
- It ends when the player disconnects and does not carry over.

To decide the game's own effects on the server, see `Buffs.claim` in [Buffs](/docs/api/buffs).
