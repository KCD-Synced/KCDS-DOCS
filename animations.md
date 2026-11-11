# Animations

A server script can make a player play any of the game's animations, hold a prop in their hands while doing so, change their facial
expression, or move their mouth as if talking. All players see the result.

## Finding an animation

`kcds.characters.Animations.list(search?)` lists everything the game plays on a human body, one row per fragment and tags, sorted by
fragment. `search` keeps rows whose fragment or tags contain it, ignoring case.

```js
const rows = kcds.characters.Animations.list('bakermill');
// [{ fragment: 'BakerMillFill', tags: '', duration: 4.233, oneShot: true, fullBody: true,
//    rightHand: '', leftHand: '', male: true, female: false, sequence: '', phase: '',
//    partner: '', slaveBody: false, player: true, clip: 'baker_mill_fill' }, ...]
```

```ts
const rows = kcds.characters.Animations.list('bakermill');
// [{ fragment: 'BakerMillFill', tags: '', duration: 4.233, oneShot: true, fullBody: true,
//    rightHand: '', leftHand: '', male: true, female: false, sequence: '', phase: '',
//    partner: '', slaveBody: false, player: true, clip: 'baker_mill_fill' }, ...]
```

```lua
local rows = kcds.characters.Animations.list("bakermill")
-- { { fragment = "BakerMillFill", tags = "", duration = 4.233, oneShot = true, fullBody = true,
--     rightHand = "", leftHand = "", male = true, female = false, sequence = "", phase = "",
--     partner = "", slaveBody = false, player = true, clip = "baker_mill_fill" }, ... }
```

```cs
var rows = Animations.List("bakermill");
// [{ fragment: "BakerMillFill", tags: "", duration: 4.233, oneShot: true, fullBody: true,
//    rightHand: "", leftHand: "", male: true, female: false, sequence: "", phase: "",
//    partner: "", slaveBody: false, player: true, clip: "baker_mill_fill" }, ...]
Log.Info(rows[0]["fragment"] + " " + rows[0]["duration"]);
```

| Field                          | Meaning                                                                      |
| ------------------------------ | ---------------------------------------------------------------------------- |
| `fragment`, `tags`             | what to pass to `playAnimation`                                              |
| `duration`                     | seconds, for the male body's clip                                            |
| `oneShot`                      | ends by itself; otherwise it loops until stopped                             |
| `fullBody`                     | moves the legs too; otherwise only the upper body                            |
| `rightHand`, `leftHand`        | the tool the game puts in that hand, the station slot it picks an item from (`GetMash`: `'scoopSlot'`), or `''` |
| `male`, `female`               | which of the game's people, men or women, have it                            |
| `player`                       | a player's body has it as a fragment                                         |
| `sequence`, `phase`, `partner` | parts of a longer action: `sequence` names it, `phase` is `in`, `loop`, `out` or `variation`, and `partner` is the other body's fragment in a two-person animation |
| `slaveBody`                    | the second body of a two-person animation                                    |
| `clip`                         | its first clip by name, or `''`; this is what `upperBody` plays              |

`Animations.props()` lists the tools animations put in people's hands, as `[{ name, model, hand }]`, where `hand` is the kind of grip, such
as `light` or `pouch`.

`Animations.clipLength(name)` answers how long one of the game's clips runs, in seconds, by its name or path, or `null` for a
clip no game row plays, such as your own `.caf`. With `playerAnimationEnd` it times a scene without reading the game's files.

### Who can play what

A man plays the rows with `male: true`. A woman plays the rows with `player: true` or `female: true`. All players use the player's own
animations, so a woman also has the men's work animations. Those only the game's women have, such as `Bartender_ServeBeer` and `PickingStuff`,
play as their clip on all screens. A fragment that repeats in the game keeps repeating until `stopAnimation`, unless you pass `loop`.

## Playing an animation

```js
// A deep bow.
player.playAnimation('Greetings', { tags: 'bowBig' });

// Reading on a loop, while the player cannot walk away.
player.playAnimation('', {
  clip: 'reading_stand_inv_in',
  loop: true,
  lockMovement: true,
});

kcds.core.Events.on('playerAnimationEnd', (player, fragment, reason) => {
  console.info(`${player.nickname}: ${fragment} ${reason}`);
});

player.stopAnimation(); // or { standUp: true } to get up the game's way
```

```ts
// A deep bow.
player.playAnimation('Greetings', { tags: 'bowBig' });

// Reading on a loop, while the player cannot walk away.
player.playAnimation('', {
  clip: 'reading_stand_inv_in',
  loop: true,
  lockMovement: true,
});

kcds.core.Events.on('playerAnimationEnd', (player, fragment, reason) => {
  console.info(`${player.nickname}: ${fragment} ${reason}`);
});

player.stopAnimation(); // or { standUp: true } to get up the game's way
```

```lua
-- A deep bow.
player:playAnimation("Greetings", { tags = "bowBig" })

-- Reading on a loop, while the player cannot walk away.
player:playAnimation("", {
  clip = "reading_stand_inv_in",
  loop = true,
  lockMovement = true,
})

kcds.core.Events.on("playerAnimationEnd", function(player, fragment, reason)
  print(player.nickname .. ": " .. fragment .. " " .. reason)
end)

player:stopAnimation() -- or { standUp = true } to get up the game's way
```

```cs
// A deep bow.
player.PlayAnimation("Greetings", new { tags = "bowBig" });

// Reading on a loop, while the player cannot walk away.
player.PlayAnimation("", new
{
    clip = "reading_stand_inv_in",
    loop = true,
    lockMovement = true,
});

Events.On("playerAnimationEnd", (Player player, string fragment, string reason) =>
{
    Log.Info($"{player.Nickname}: {fragment} {reason}");
});

player.StopAnimation(); // or new { standUp = true } to get up the game's way
```

| Option         | Meaning                                                                       |
| -------------- | ----------------------------------------------------------------------------- |
| `tags`         | the fragment's tags, from the catalogue                                       |
| `clip`         | a clip by name, with an empty fragment: one of the game's, or a `.caf` your resource ships |
| `loop`         | repeats directly, with no idle stance between passes, until `stopAnimation` or the next `playAnimation` |
| `upperBody`    | plays on arms, spine and head only, so the legs keep walking and moving does not end it; plays the fragment's `clip`, so a fragment without one is refused; takes no `lockMovement` |
| `gravity`      | `false` keeps the body where the script puts it; when the animation ends, the body is put on the ground below it and falls again |
| `lockRotation` | `true` keeps the body's heading; the mouse turns only the head and camera, up to 120 degrees either side and 70 up and down, all players see the head turn, and the player cannot walk. `{ yaw, up, down }` sets those limits in degrees (`yaw` 1 to 180, `up` and `down` 0 to 89; a missing one keeps its default), and the view then begins facing the body's heading, in the middle of its up and down range |
| `speed`        | `1` is as authored and the default, `0.5` is half speed, `0` holds still, up to `10` |
| `time`         | seconds into it to begin; the body plays the earlier part at up to 20 times speed |
| `lockMovement` | the player cannot walk away while it plays                                    |
| `props`        | up to two things to hold; see [Holding props](#holding-props)                 |

With `gravity: false`, for example, you can lift a hanged player into the noose with `teleport`, then play `DeadBody_Hanged`. With `lockRotation`,
a player tied to a stake looks around without turning away from it. A player sat at a game board looks
a little to the sides and mostly down at it, as the game's dice players do:

```js
player.playAnimation('IngameDialogPose_SittingWithOrWithoutTable_Loop', {
  tags: 'sitting',
  loop: true,
  lockRotation: { yaw: 45, up: 10, down: 60 },
});
```

```ts
player.playAnimation('IngameDialogPose_SittingWithOrWithoutTable_Loop', {
  tags: 'sitting',
  loop: true,
  lockRotation: { yaw: 45, up: 10, down: 60 },
});
```

```lua
player:playAnimation("IngameDialogPose_SittingWithOrWithoutTable_Loop", {
  tags = "sitting",
  loop = true,
  lockRotation = { yaw = 45, up = 10, down = 60 },
})
```

```cs
player.PlayAnimation("IngameDialogPose_SittingWithOrWithoutTable_Loop", new
{
    tags = "sitting",
    loop = true,
    lockRotation = new { yaw = 45, up = 10, down = 60 },
});
```

### Changing a running animation

These apply on all screens without restarting it. They return `false` if no `playAnimation` is running. A loop keeps its speed between passes.

- `setAnimationSpeed(speed)`, and `animationSpeed` to read it, which is `0` while paused and `null` when nothing is running
- `pauseAnimation()` holds the current frame, and `resumeAnimation()` continues at its speed
- `setAnimationProps(props)` changes the props, which helps tune a held prop while the hand moves

```js
player.playAnimation('PrayStand', { loop: true, speed: 0.5, time: 2 });
player.pauseAnimation();
player.setAnimationSpeed(2); // takes effect once resumed
player.resumeAnimation();
player.setAnimationProps({ model: 'wooden_scoop', position: { x: 0.02, y: 0, z: 0 } });
```

```ts
player.playAnimation('PrayStand', { loop: true, speed: 0.5, time: 2 });
player.pauseAnimation();
player.setAnimationSpeed(2); // takes effect once resumed
player.resumeAnimation();
player.setAnimationProps({ model: 'wooden_scoop', position: { x: 0.02, y: 0, z: 0 } });
```

```lua
player:playAnimation("PrayStand", { loop = true, speed = 0.5, time = 2 })
player:pauseAnimation()
player:setAnimationSpeed(2) -- takes effect once resumed
player:resumeAnimation()
player:setAnimationProps({ model = "wooden_scoop", position = { x = 0.02, y = 0, z = 0 } })
```

```cs
player.PlayAnimation("PrayStand", new { loop = true, speed = 0.5, time = 2 });
player.PauseAnimation();
player.SetAnimationSpeed(2); // takes effect once resumed
player.ResumeAnimation();
player.SetAnimationProps(new { model = "wooden_scoop", position = new { x = 0.02, y = 0, z = 0 } });
```

### When it refuses or ends

`playAnimation` returns `false` before the body exists, while cuffed, and when the body lacks the fragment.

`playerAnimationEnd` fires once per request, with one of these reasons:

| Reason        | Meaning                                |
| ------------- | -------------------------------------- |
| `finished`    | a one-shot ended                       |
| `interrupted` | a hit or fall took the body            |
| `refused`     | the body has no such animation         |
| `stopped`     | `stopAnimation` was called             |
| `replaced`    | another animation replaced it          |

NPCs have the same `playAnimation`, with `npcAnimationEnd`; see [NPCs](npcs.md).

### Seated players

A player on a bench or chair (`player.seated`) plays the animation within the seat and stays there. For example, the tavern guests'
`Guest_DrawTankard`, `Guest_DrinkBeer` and `Guest_CheersBeer` play to their end at the table. `stopAnimation()` ends it and leaves them seated,
even for a loop such as `EatingMash` with `sitting+player`, and `{ standUp: true }` stands them up.

```js
if (player.seated) {
  player.playAnimation('Guest_DrinkBeer');
}
```

```ts
if (player.seated) {
  player.playAnimation('Guest_DrinkBeer');
}
```

```lua
if player.seated then
  player:playAnimation("Guest_DrinkBeer")
end
```

```cs
if (player.Seated)
{
    player.PlayAnimation("Guest_DrinkBeer");
}
```

## Holding props

`props` takes up to two things to hold. Each is `{ item }`, an item class, or `{ model }`, a mesh from the props catalogue, with these options:

| Option                 | Meaning                                                                            |
| ---------------------- | ---------------------------------------------------------------------------------- |
| `hand`                 | `'right'` (default) or `'left'`                                                    |
| `attach`               | `'bone'` (default) or `'game'`                                                     |
| `joint`                | another joint to hang it on                                                        |
| `position`, `rotation` | an offset from the hand                                                            |
| `fromAnimation`        | follows the animation's own pick-up and put-down                                   |
| `untilPutDown`         | in the hand from the start, gone at the animation's put-down (`place`, `detach`)   |
| `rest`                 | a `Prop` lying where the item lies                                                 |
| `lit`                  | burns like the game's torches                                                      |

```js
player.playAnimation('', {
  clip: 'reading_stand_inv_in',
  loop: true,
  props: { model: 'wooden_scoop', hand: 'right' },
});
```

```ts
player.playAnimation('', {
  clip: 'reading_stand_inv_in',
  loop: true,
  props: { model: 'wooden_scoop', hand: 'right' },
});
```

```lua
player:playAnimation("", {
  clip = "reading_stand_inv_in",
  loop = true,
  props = { model = "wooden_scoop", hand = "right" },
})
```

```cs
player.PlayAnimation("", new
{
    clip = "reading_stand_inv_in",
    loop = true,
    props = new { model = "wooden_scoop", hand = "right" },
});
```

### Grip

With `attach: 'bone'`, the prop hangs on the hand's bone, and `position` and `rotation` place it. With `attach: 'game'`, it sits in the game's own
grip, on the hand's slot joint (`RightHandSlot` or `LeftHandSlot`). The game's animations move that joint to where the hand holds its item, just as
NPCs hold a tankard, cup or scoop. `position` and `rotation` then only fine-tune it.

```js
// An upright tankard, raised to the mouth and set down between sips.
player.playAnimation('Guest_DrinkBeer', { props: { item: 'tankard_beer', attach: 'game' } });
```

```ts
// An upright tankard, raised to the mouth and set down between sips.
player.playAnimation('Guest_DrinkBeer', { props: { item: 'tankard_beer', attach: 'game' } });
```

```lua
-- An upright tankard, raised to the mouth and set down between sips.
player:playAnimation("Guest_DrinkBeer", { props = { item = "tankard_beer", attach = "game" } })
```

```cs
// An upright tankard, raised to the mouth and set down between sips.
player.PlayAnimation("Guest_DrinkBeer", new { props = new { item = "tankard_beer", attach = "game" } });
```

### Other joints

`joint` hangs the prop elsewhere on the body. These are the game's joint names:

- **Core:** `Head`, `Neck`, `Neck1`, `Spine` to `Spine4`, `Hips`
- **Limbs:** `LeftShoulder`, `LeftArm`, `LeftForeArm`, `LeftHand`, `LeftHandSlot`, `LeftUpLeg`, `LeftLeg`, `LeftFoot`, and the same with `Right`
- **Fingers:** such as `RightHandIndex1`
- **Slots:** such as `DaggerSlot`, `BackSlot_bow`, `BackSlot_shield` and `LeftWeaponSheath`

Biped names also work, so `Bip01 Neck` means `Neck` and `Bip01 R Hand` means `RightHand`. A joint the body lacks hangs the prop at the feet, and the
client's log then lists the body's joints.

```js
// A noose rope on the neck while hanging.
player.playAnimation('DeadBody_Hanged', {
  tags: 'slaveHalter+hanged_01',
  loop: true,
  props: { item: 'poi_hroby_noose', joint: 'Neck' },
});
```

```ts
// A noose rope on the neck while hanging.
player.playAnimation('DeadBody_Hanged', {
  tags: 'slaveHalter+hanged_01',
  loop: true,
  props: { item: 'poi_hroby_noose', joint: 'Neck' },
});
```

```lua
-- A noose rope on the neck while hanging.
player:playAnimation("DeadBody_Hanged", {
  tags = "slaveHalter+hanged_01",
  loop = true,
  props = { item = "poi_hroby_noose", joint = "Neck" },
})
```

```cs
// A noose rope on the neck while hanging.
player.PlayAnimation("DeadBody_Hanged", new
{
    tags = "slaveHalter+hanged_01",
    loop = true,
    props = new { item = "poi_hroby_noose", joint = "Neck" },
});
```

### Props that follow the animation

With `fromAnimation: true`, the prop appears when the animation picks it up, through its `pickup`, `spawnitem` or `attach` event for that hand. It goes
when the animation puts it down, through `place`, `despawnitem` or `detach`. An animation whose `rightHand` or `leftHand` names a station slot works this
way in the game, and all screens see it at the same moments.

```js
// The scoop appears as GetMash reaches for it, and goes as it is laid back.
player.playAnimation('GetMash', { props: { item: 'Scoop', attach: 'game', fromAnimation: true } });
```

```ts
// The scoop appears as GetMash reaches for it, and goes as it is laid back.
player.playAnimation('GetMash', { props: { item: 'Scoop', attach: 'game', fromAnimation: true } });
```

```lua
-- The scoop appears as GetMash reaches for it, and goes as it is laid back.
player:playAnimation("GetMash", { props = { item = "Scoop", attach = "game", fromAnimation = true } })
```

```cs
// The scoop appears as GetMash reaches for it, and goes as it is laid back.
player.PlayAnimation("GetMash", new { props = new { item = "Scoop", attach = "game", fromAnimation = true } });
```

### At a horse or another body

`target` plays an animation at a horse, a player or an NPC wherever it stands: the body is first put beside it,
turned to it, as the game's camp places a groom, and the animation starts a moment later. `side` is `'left'` (default),
`'right'`, `'head'` or `'rear'`, and `distance` overrides how far from the target's middle it stands.

```js
player.playAnimation('CampHorseGroomingLoopLeft', { target: horse, side: 'left', loop: true });
```

```ts
player.playAnimation('CampHorseGroomingLoopLeft', { target: horse, side: 'left', loop: true });
```

```lua
player:playAnimation("CampHorseGroomingLoopLeft", { target = horse, side = "left", loop = true })
```

```cs
player.PlayAnimation("CampHorseGroomingLoopLeft", new { target = horse, side = "left", loop = true });
```

With `untilPutDown: true`, the prop is in the hand from the start and goes when the animation puts it down through its
`place`, `despawnitem` or `detach` event, for animations that hand an item over without picking one up first:

```js
player.playAnimation('AlchemyReturn', {
  tags: 'alchemyBowl',
  props: { item: 'Scoop', attach: 'game', untilPutDown: true },
});
```

```ts
player.playAnimation('AlchemyReturn', {
  tags: 'alchemyBowl',
  props: { item: 'Scoop', attach: 'game', untilPutDown: true },
});
```

```lua
player:playAnimation("AlchemyReturn", {
  tags = "alchemyBowl",
  props = { item = "Scoop", attach = "game", untilPutDown = true },
})
```

```cs
player.PlayAnimation("AlchemyReturn", new
{
    tags = "alchemyBowl",
    props = new { item = "Scoop", attach = "game", untilPutDown = true },
});
```

Such a prop can also lie in the world before and after. Set `rest` to a `Prop` you placed there, such as a scoop by a pot. All screens hide it while the
animation holds the item, and show it again afterwards. The events `propPickedUp` and `propPlaced`, each with (player, prop), report the moments, where
`prop` is `{ index, fragment, hand, item, model, rest }`.

```js
const potScoop = kcds.world.Prop.spawn('wooden_scoop', { x: 512.4, y: 1288.1, z: 31.02 });

player.playAnimation('GetMash', {
  tags: 'FrontIn',
  props: { item: 'Scoop', attach: 'game', fromAnimation: true, rest: potScoop },
});

kcds.core.Events.on('propPickedUp', (who, prop) => {
  if (prop.rest === potScoop) {
    console.info(`${who.nickname} scoops soup from the pot`);
  }
});
```

```ts
const potScoop = kcds.world.Prop.spawn('wooden_scoop', { x: 512.4, y: 1288.1, z: 31.02 });

player.playAnimation('GetMash', {
  tags: 'FrontIn',
  props: { item: 'Scoop', attach: 'game', fromAnimation: true, rest: potScoop },
});

kcds.core.Events.on('propPickedUp', (who, prop) => {
  if (prop.rest === potScoop) {
    console.info(`${who.nickname} scoops soup from the pot`);
  }
});
```

```lua
local potScoop = kcds.world.Prop.spawn("wooden_scoop", { x = 512.4, y = 1288.1, z = 31.02 })

player:playAnimation("GetMash", {
  tags = "FrontIn",
  props = { item = "Scoop", attach = "game", fromAnimation = true, rest = potScoop },
})

kcds.core.Events.on("propPickedUp", function(who, prop)
  if prop.rest == potScoop then
    print(who.nickname .. " scoops soup from the pot")
  end
end)
```

```cs
var potScoop = Prop.Spawn("wooden_scoop", new { x = 512.4, y = 1288.1, z = 31.02 });

player.PlayAnimation("GetMash", new
{
    tags = "FrontIn",
    props = new { item = "Scoop", attach = "game", fromAnimation = true, rest = potScoop },
});

Events.On("propPickedUp", (Player who, JsValue prop) =>
{
    if (prop["rest"].Is(potScoop.AsValue()))
    {
        Log.Info($"{who.Nickname} scoops soup from the pot");
    }
});
```

### Burning props

With `lit: true`, a prop burns like the game's torches, with a flame and warm light 0.45 m along it from where it is held. All screens see this until the
animation is stopped or replaced, or its props change.

```js
player.playAnimation('CrimeSurrender', {
  tags: 'l_torch',
  loop: true,
  props: { model: 'torch_player', hand: 'left', attach: 'game', lit: true },
});
```

```ts
player.playAnimation('CrimeSurrender', {
  tags: 'l_torch',
  loop: true,
  props: { model: 'torch_player', hand: 'left', attach: 'game', lit: true },
});
```

```lua
player:playAnimation("CrimeSurrender", {
  tags = "l_torch",
  loop = true,
  props = { model = "torch_player", hand = "left", attach = "game", lit = true },
})
```

```cs
player.PlayAnimation("CrimeSurrender", new
{
    tags = "l_torch",
    loop = true,
    props = new { model = "torch_player", hand = "left", attach = "game", lit = true },
});
```

## Two-person animations

`player.playPairedAnimation(other, fragment, options)` plays one of the game's animations for two characters at once, with this player as the master and
`other` as the slave. The master stays put, and plays `fragment`. The other is moved to where the pair needs them, and plays its `Slave` fragment. The two
bodies do not push each other while it plays.

```js
guard.playPairedAnimation(condemned, 'Quest_StandHlbrdKill');
```

```ts
guard.playPairedAnimation(condemned, 'Quest_StandHlbrdKill');
```

```lua
guard:playPairedAnimation(condemned, "Quest_StandHlbrdKill")
```

```cs
guard.PlayPairedAnimation(condemned, "Quest_StandHlbrdKill");
```

Examples are `Quest_StandHlbrdKill` with `Quest_StandHlbrdKillSlave`, `Quest_GroundDaggerKill`, `Quest_HangmanCutoff`, `TiedUpOut_Master` with
`TiedUpOut_Slave`, `LootIn`, and the other `...Slave` rows in the catalogue. Some need the master's weapon in hand, as in the game. `Quest_StandHlbrdKill` needs
a halberd, and bare-handed the master's part ends at once.

**Options:** `slaveFragment` for a pair the catalogue does not name, `tags`, `slaveTags`, and `lockMovement`, which holds both still and is `true` by default.

**It returns `false`** for a player with no body, a cuffed player, the same player twice, players in different dimensions, a fragment either body lacks, or a
master with no `Slave`. `playerAnimationEnd` follows for each.

## Pointing

While a player holds **B**, by key position, their right arm points where their camera looks and follows it. The legs keep walking, and other players see the arm
move. It works while no server `playAnimation` is in place, either before the first or after `stopAnimation`. While pointing is on, the game's own
quick-access menu on B is off, so B does not draw the quick-access weapon; turning pointing off gives B back to the game. It is on by default, and you can
turn it off:

```js
kcds.players.Interactions.set('pointing', false); // for everyone
player.setInteraction('pointing', false); // for one player
```

```ts
kcds.players.Interactions.set('pointing', false); // for everyone
player.setInteraction('pointing', false); // for one player
```

```lua
kcds.players.Interactions.set("pointing", false) -- for everyone
player:setInteraction("pointing", false) -- for one player
```

```cs
Interactions.Set("pointing", false); // for everyone
player.SetInteraction("pointing", false); // for one player
```

## Facial expressions

```js
player.setExpression('FE_Default', { tags: 'angry' });
player.setExpression('FE_DialogueSpeaking', { tags: 'happy' });
player.setExpression(''); // back to the game's own face
```

```ts
player.setExpression('FE_Default', { tags: 'angry' });
player.setExpression('FE_DialogueSpeaking', { tags: 'happy' });
player.setExpression(''); // back to the game's own face
```

```lua
player:setExpression("FE_Default", { tags = "angry" })
player:setExpression("FE_DialogueSpeaking", { tags = "happy" })
player:setExpression("") -- back to the game's own face
```

```cs
player.SetExpression("FE_Default", new { tags = "angry" });
player.SetExpression("FE_DialogueSpeaking", new { tags = "happy" });
player.SetExpression(""); // back to the game's own face
```

The fragments are `FE_Default`, `FE_DialogueIdle` and `FE_DialogueSpeaking`. Put a mood in `tags`, such as `angry`, `arogant`, `drunk`, `happy`, `nervous`,
`pensive`, `sad`, `injured`, `combat`, or `FE_Sleeping` for closed eyes. `Animations.list('FE_')` lists them all.

The face keeps the expression alongside whatever the body plays, until the next one or `''`. A player who reconnects to the world gets it again.

## Talking

```js
player.setSpeaking(true); // the mouth moves on all screens
player.setSpeaking(true, 'happy'); // this time only: 'neutral', 'happy', 'drunk' or 'chew'
player.setTalkingStyle('drunk'); // their style from now on
player.setSpeaking(false);

console.info(player.talking); // the current mouth style, or null
console.info(player.talkingStyle); // 'drunk'
```

```ts
player.setSpeaking(true); // the mouth moves on all screens
player.setSpeaking(true, 'happy'); // this time only: 'neutral', 'happy', 'drunk' or 'chew'
player.setTalkingStyle('drunk'); // their style from now on
player.setSpeaking(false);

console.info(player.talking); // the current mouth style, or null
console.info(player.talkingStyle); // 'drunk'
```

```lua
player:setSpeaking(true) -- the mouth moves on all screens
player:setSpeaking(true, "happy") -- this time only: "neutral", "happy", "drunk" or "chew"
player:setTalkingStyle("drunk") -- their style from now on
player:setSpeaking(false)

print(player.talking) -- the current mouth style, or nil
print(player.talkingStyle) -- "drunk"
```

```cs
player.SetSpeaking(true); // the mouth moves on all screens
player.SetSpeaking(true, "happy"); // this time only: "neutral", "happy", "drunk" or "chew"
player.SetTalkingStyle("drunk"); // their style from now on
player.SetSpeaking(false);

Log.Info(player.Talking.ToString()); // the current mouth style, or null
Log.Info(player.TalkingStyle); // "drunk"
```

This loops the game's face clips and is not lip-sync, so it follows no words. Use it while someone talks over voice or reads out a line from your resource. The
nameplate shows a speaking icon during it; see [Nameplates](nameplates.md).

Other players do not see the mouth movements for a player's grunts and cries in a fight, since they do not hear those sounds.

## The emote wheel

Client scripts use `kcds.ui.EmoteWheel` to manage the emote catalogue and the six slots of the player's wheel. A gesture plays for everyone through the server's
`playAnimation`. Each entry names its animation in `fragment`, `tags` and, for a woman's body, `femaleTags`, so an NPC can play the same emote with
`npc.playAnimation(emote.fragment, { tags: emote.tags })`.

```js
// client
const { EmoteWheel } = kcds.ui;

const catalogue = EmoteWheel.getCatalog(); // [{ id, name, duration, fullBody, fragment, tags, femaleTags }]
EmoteWheel.assignSlot(5, 8); // the last slot now points
console.log(JSON.stringify(EmoteWheel.getSlots())); // [1, 2, 3, 4, 5, 8]
```

```ts
// client
const { EmoteWheel } = kcds.ui;

const catalogue = EmoteWheel.getCatalog(); // [{ id, name, duration, fullBody, fragment, tags, femaleTags }]
EmoteWheel.assignSlot(5, 8); // the last slot now points
console.log(JSON.stringify(EmoteWheel.getSlots())); // [1, 2, 3, 4, 5, 8]
```

```lua
-- client
local EmoteWheel = kcds.ui.EmoteWheel

local catalogue = EmoteWheel.getCatalog() -- { { id, name, duration, fullBody, fragment, tags, femaleTags } }
EmoteWheel.assignSlot(5, 8) -- the last slot now points
print(json.encode(EmoteWheel.getSlots())) -- [1,2,3,4,5,8]
```

```cs
// client
var catalogue = EmoteWheel.GetCatalog(); // [{ id, name, duration, fullBody, fragment, tags, femaleTags }]
EmoteWheel.AssignSlot(5, 8); // the last slot now points
Log.Info(string.Join(",", EmoteWheel.GetSlots())); // 1,2,3,4,5,8
```

| id | Emote          | `playAnimation` arguments                                                                 | Seconds | Full body |
| -- | -------------- | ----------------------------------------------------------------------------------------- | ------- | --------- |
| 1  | Wave           | `'GreetingsUpperBody', { tags: 'l_noweapon+waveSmall' }`                                  | 2.8     | no        |
| 2  | Big wave       | `'GreetingsUpperBody', { tags: 'l_noweapon+waveBig' }`                                    | 4.2     | no        |
| 3  | Nod            | `'GreetingsUpperBody', { tags: 'l_noweapon+nod' }` (a woman: `'nod'`)                     | 2.2     | no        |
| 4  | Respectful nod | `'GreetingsUpperBody', { tags: 'l_noweapon+nodSubmissive' }` (a woman: `'nodSubmissive'`) | 1.8     | no        |
| 5  | Bow            | `'Greetings', { tags: 'bowBig' }`                                                         | 3.0     | yes       |
| 6  | Cheer          | `'ApplauseHappy'`                                                                         | 8.3     | yes       |
| 7  | Sad applause   | `'ApplauseSad'`                                                                           | 8.2     | yes       |
| 8  | Point          | `'Pointing'`                                                                              | 8.6     | yes       |

The layout is kept on the player's machine for the session. `EmoteWheel.reset()` restores the default wheel, slots 1 to 6.

Also: `Animations.undefine(name)` forgets a name `Animations.define` gave (players playing it keep playing it until
`stopAnimation` or their next `playAnimation`). `EmoteWheel.assignSlots(emoteIds)` sets all of the wheel's
`EmoteWheel.slotTotal()` slots (6) together, and changes nothing with a wrong length or id.

Reference: [Animations](/docs/api/animations), [Player](/docs/api/player), [EmoteWheel](/docs/api/emotewheel).
