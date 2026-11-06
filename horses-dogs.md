# Horses and dogs

## Horses

`kcds.characters.Horse` is a horse the server spawns. Its coat and tack look the same in all games. It
can have an owner and a rider, follow orders, and has its health kept by the server.

```js
const { Horse } = kcds.characters;
const { Vector3 } = kcds.math;

const spot = player.position.clone().add(player.rotation.rotateVector(new Vector3(0, 3, 0)));
const pebbles = Horse.spawn(spot, player.rotation, '', 'Pebbles', 'horse_noble05');

pebbles.setDimension(player.dimension);
pebbles.giveTo(player);
player.mount(pebbles);
```

```lua
local Horse = kcds.characters.Horse
local Vector3 = kcds.math.Vector3

local spot = player.position:clone():add(player.rotation:rotateVector(Vector3.new(0, 3, 0)))
local pebbles = Horse.spawn(spot, player.rotation, "", "Pebbles", "horse_noble05")

pebbles:setDimension(player.dimension)
pebbles:giveTo(player)
player:mount(pebbles)
```

```cs
var spot = player.Position.Clone().Add(player.Rotation.RotateVector(new Vector3(0, 3, 0)));
var pebbles = Horse.Spawn(spot, player.Rotation, "", "Pebbles", "horse_noble05");

pebbles.SetDimension(player.Dimension);
pebbles.GiveTo(player);
player.Mount(pebbles);
```

### Spawning

`Horse.spawn(position, orientation, soul?, name?, gear?, options?)`

- `soul` is a breed from `Horse.breeds()`, which comes from the server's horse config, or a soul GUID.
- `gear` is its tack (see below). Without it, the horse wears one of the game's generic sets.
- `{ knockOut: true }` makes a fatal blow knock it out instead.

Also available are `Horse.all()`, `Horse.getById(id)` and `destroy()`.

### Owner and rider

`giveTo(player)` sets the owner, and `null` removes it. `owner`, `rider` and `mounted` read the current
state. An owner mounts their own horse, and for anyone else it counts as a stranger's horse, as in the
game. With `removedWithOwner`, which is true by default, the horse goes when its owner leaves.

`player.mount(horse, { instant })` puts a player standing nearby in the saddle, and `player.dismount()`
takes them out. Return `false` from `horseMounting` to refuse a ride.

`horse.allowRider(player, allowed?)` lends the horse: that player sees the game's plain mount instead of stealing it, and the
saddlebags stay the owner's. `horse.riders` lists those players, and `allowRider(player, false)` takes it back.

`World.addInteraction({ entity: horse, text })` puts a hint on the horse that moves with it, such as feed or groom.
`setPlayers([owner])` shows it to the owner only.

`player.whistle()` calls the player's horse like the game's companion key, and fires `playerWhistle`.

### Pace

`speedScale` is the share of its pace the horse moves at in every gait, walking to galloping, ridden or led: 0.1 to 2, or
`null` for the game's own. Everyone sees it. At 0.5 a horse covers half the ground at the same gait, for an old, sick or
neglected one:

```js
horse.speedScale = 0.5;
```

```lua
horse.speedScale = 0.5
```

```cs
horse.SpeedScale = new JsValue(0.5);
```

`speed` (30 to 60) is only what the inspect screen shows.

### Dirt

`dirt` is how dirty the coat and tack are, 0 to 1, or `null` for the game's own; everyone sees it, mostly on the legs
and the tack. `soil(amount)` adds to it, and `clean()` brushes it off.

### Health

`stamina` is its current stamina as the game that runs it reports it, next to `maxStamina`.

`health`, `alive`, `knockedOut` (also `unconscious`), `kill()`, `knockDown()`, `wake(health?)` and
`revive(health?)`.

### Orders

`moveTo(position, { speed })`, `patrol(points, { speed, loop, waitSeconds })` and `hold()`.
`travelStatus` tells you how it is going. A rider stays in control.

`follow(target, { radius, speed })` leads it after a player, an NPC or another horse, keeping `radius` metres (3) behind,
at a pace by the distance unless you give one. With `rope: true` everyone sees a lead rope from the horse's head to the
leader's right hand, hanging when they are close and pulling straight as they walk off. `following` reads the target,
and `hold()` ends it, rope included.

### Tack

`gear` is `{ saddle, head, torso, shoe }`, with item class names or `null` for empty. It is the same on
all clients, including the rider's.

| Call                                  | Effect                                                     |
| ------------------------------------- | ---------------------------------------------------------- |
| `setTack('horse_noble05')`            | puts on a named set                                        |
| `setTack({ saddle, head, torso, shoe })` | sets exactly these pieces; `{}` leaves it bare          |
| `fitGear(item)`                       | adds one piece                                             |
| `takeOffGear(slot)`                   | removes one piece; the caparison comes off with the saddle |
| `Horse.gearSets()`, `Horse.gearList(slot?)` | list the game's sets and pieces                      |

An owner can also change tack on the Horse tab of the game's inventory screen. A piece put on comes from
the saddlebags or the owner's inventory. A piece taken off goes to the saddlebags, or to the owner's
inventory when the saddlebags refuse it.

Two things can stop an owner's change. `gearFixed = true` refuses them all, and `horseGearChanging`, given
the horse, the player and the resulting gear, can return `false`. Neither affects `setTack`, `fitGear` or
`takeOffGear` called from scripts.

### Buffs

`addBuff(buff)` puts a game buff, by GUID or table name, on the horse's body in all games. Examples are
`konskeZavody_horseBoost_diff3` (a race boost) and `item_horse_shoe`. `removeBuff(buff)`, `hasBuff(buff)`,
`clearBuffs()` and `buffs` work on the server's list of up to sixteen names. A buff that wears off is
applied again while it remains on the list. Custom buffs from `kcds.players.Buffs.register` are only for
players and throw here.

### Looks

`look` and `setLook({ body, head, hair, tail })` take names from [HorseLooks](/docs/api/horselooks).
`body` is the coat, `head` the breed and `hair` the mane.

### Events

`horseSpawn`, `horseOwnerChanged`, `horseMounting`, `horseMount`, `horseDismount`, `horseGearChanging`,
`horseGearChanged`, `horseDamage`, `horseDeath`, `horseKnockedOut`, `horseWake`, `horseRevive`,
`horseDestroy`.

### Which game simulates a horse

A game can run a horse's AI only when it has finished loading the current world and started gameplay, so
connections that only receive world data do not run it. While a mounted rider reloads, the simulation
waits for them, and a nearby player does not take over. A horse with no rider can move to another eligible
game during loading.

## Dogs

`kcds.characters.Dog` is a companion belonging to a player. It is an NPC of the game's dog class with an
owner and a mode.

```js
const { Dog } = kcds.characters;

const mutt = Dog.spawn(player, player.position, player.rotation, '', 'Mutt');
mutt.mode = 1; // follow
```

```lua
local Dog = kcds.characters.Dog

local mutt = Dog.spawn(player, player.position, player.rotation, "", "Mutt")
mutt.mode = 1 -- follow
```

```cs
var mutt = Dog.Spawn(player, player.Position, player.Rotation, "", "Mutt");
mutt.Mode = 1; // follow
```

### Modes

| `mode` | Name       | What the dog does                                                                                                |
| ------ | ---------- | ---------------------------------------------------------------------------------------------------------------- |
| 0      | Wait       | stays where it is                                                                                                |
| 1      | Follow     | the default; see below                                                                                           |
| 2      | Free       | not implemented yet, it stays where it is                                                                        |
| 3      | Aggressive | not implemented yet, it stays where it is                                                                        |
| 4      | Search     | roams to places within 12 m of its owner                                                                         |
| 5      | Hunt       | runs at the nearest living animal within 40 m, chases it up to 80 m and bites for 12 health per second; with no animal, it follows the owner |
| 6      | Guard      | not implemented yet, it stays where it is                                                                        |
| 7      | Ambush     | not implemented yet, it stays where it is                                                                        |

When hunting, the dog counts as the attacker in `npcDamage`.

`objective` reports what it is doing: `'waiting'`, `'following'`, `'searching'`, `'hunting'`, or `'idle'`
in a mode not yet implemented.

In Follow, the dog trots three metres behind its owner and runs if it falls over four metres further back.
Since it is an NPC, `moveTo(position, { speed })` also works, with `walk`, `jog` or `run`.

### Owner

`giveTo(player)` (or `null`), `owner`, `removedWithOwner`, `destroy()`. `player.dog` gives the player's dog.

### Fighting dogs

If a player locks on to a dog that is not their own, the dog fights them, since the game's lock-on only
accepts a body that fights, and their blows then land (`npcDamage`). The owner's lock-on does nothing. The
same applies to a `WildDog`, `Wolf` or `Boar`, while other animals flee.

### Events

As with other NPCs, `npcDamage` and `npcDeath` fire for its body. Its own events are `dogSpawn`,
`dogOwnerChanged`, `dogModeChanged` and `dogDestroy`.

Also: `horse.buckRider()` rears the horse to throw its rider with the game's animation, `horse.dragRider(attacker)` has
a player pull the rider down with the game's move, and `horse.unseatRider()` dismounts the rider and fires
`horseDismount`. `horse.hitPoints` is how much a blow's damage is divided across (higher loses less health per blow),
and `horse.ownerId`, `horse.riderId` and `dog.ownerId` are player ids, 0 for none.

Reference: [Horse](/docs/api/horse), [HorseLooks](/docs/api/horselooks), [Dog](/docs/api/dog).
