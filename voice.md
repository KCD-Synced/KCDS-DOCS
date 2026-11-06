# Voice chat

Players near each other can talk. Voice chat is built into the server: it is on by default, uses the
game's port and needs no extra program. Nothing is recorded.

## Who hears whom

A player talks in one of three tiers, and only players within its range and in the same dimension
hear them. Each player hears at most the 8 clearest talkers at once.

| Tier | Default range |
|---|---|
| `whisper` | 2 m |
| `normal` | 10 m |
| `shout` | 30 m |

```js
const { Voice } = kcds.players;

Voice.setRange('shout', 50);
```

```ts
const { Voice } = kcds.players;

Voice.setRange('shout', 50);
```

```lua
local Voice = kcds.players.Voice

Voice.setRange("shout", 50)
```

```cs
Voice.SetRange("shout", 50);
```

## How a voice sounds

A voice sounds different with what is around and on its talker. Walls and doors muffle it, and a closed
helmet visor takes its highs. A drunk talker's voice wanders, a hurt or breathless one is weaker, and a knocked-out one is
only a faint mumble. Heavy rain and riding cover a voice some. None of this changes who hears whom.

## Switching it off

`Voice.setEnabled(false)` switches voice chat off for the whole server. Every player's game then closes
its microphone and voice output, and nothing goes over the network.

```js
kcds.players.Voice.setEnabled(false);
```

```ts
kcds.players.Voice.setEnabled(false);
```

```lua
kcds.players.Voice.setEnabled(false)
```

```cs
Voice.SetEnabled(false);
```

## The voice indicator

Each player's game shows their voice chat in the bottom left corner, right of the KCD2 Synced watermark (the
watermark shows the server's name and the player's ping under it, or `localhost` for a server on the same
computer). The indicator is a small pill with a microphone, the range the player talks in (whisper, normal or
shout) and one to three pips for it. While the player is heard it turns green, with level bars that follow
their microphone, or amber on shout. With push to talk, the key replaces the pips. When the server silences
the player, it turns red and reads MUTED.

`Voice.setIndicator(false)` hides it in every player's game, so a client resource can show its own instead
from `kcds.ui.LocalVoice` (see [In a client script](#in-a-client-script)). Voice chat keeps working. The
watermark always stays.

```js
kcds.players.Voice.setIndicator(false);
```

```ts
kcds.players.Voice.setIndicator(false);
```

```lua
kcds.players.Voice.setIndicator(false)
```

```cs
Voice.SetIndicator(false);
```

## What a player can set

The voice chat settings are in the game's own **Settings > Sound Settings** page, below the game's volume
sliders, in the ESC menu and in the main menu. A player chooses push to talk or voice activation and its
sensitivity, the microphone and output devices, how loud others hear them, and how loud they hear others.
On a server, **Microphone test** moves while they speak. A player on speakers turns on echo cancellation, so others do not
hear their own voices come back. A change is heard at once; **Confirm** keeps it, leaving with **No** puts
the settings back, and **Reset** sets the defaults, as for the game's own volumes.

A microphone shows beside the name of whoever is talking, and the player's own voice indicator lights up while
what they say goes out. Either can be turned off in the same settings. These are the player's own choices; a server does
not set them.

## Rules for players

```js
const { Voice } = kcds.players;
const { Events } = kcds.core;

// Silence a player for everyone, or make them hear nobody.
Events.on('playerConnect', (player) => {
  Voice.setPlayerMuted(player, false);
  Voice.setPlayerDeaf(player, false);
});

// Players may not shout here.
Events.on('voiceTierRequest', (player, tier) => {
  return tier !== 'shout';
});

Events.on('voiceTalkingChanged', (player, talking) => {
  console.info(`${player.nickname} ${talking ? 'talks' : 'is silent'}`);
});
```

```ts
const { Voice } = kcds.players;
const { Events } = kcds.core;

// Silence a player for everyone, or make them hear nobody.
Events.on('playerConnect', (player) => {
  Voice.setPlayerMuted(player, false);
  Voice.setPlayerDeaf(player, false);
});

// Players may not shout here.
Events.on('voiceTierRequest', (player, tier) => {
  return tier !== 'shout';
});

Events.on('voiceTalkingChanged', (player, talking) => {
  console.info(`${player.nickname} ${talking ? 'talks' : 'is silent'}`);
});
```

```lua
local Voice = kcds.players.Voice
local Events = kcds.core.Events

-- Silence a player for everyone, or make them hear nobody.
Events.on("playerConnect", function(player)
  Voice.setPlayerMuted(player, false)
  Voice.setPlayerDeaf(player, false)
end)

-- Players may not shout here.
Events.on("voiceTierRequest", function(player, tier)
  return tier ~= "shout"
end)

Events.on("voiceTalkingChanged", function(player, talking)
  print(tostring(player.nickname) .. (talking and " talks" or " is silent"))
end)
```

```cs
// Silence a player for everyone, or make them hear nobody.
Events.On("playerConnect", (Player player) =>
{
    Voice.SetPlayerMuted(player, false);
    Voice.SetPlayerDeaf(player, false);
});

// Players may not shout here.
Events.On("voiceTierRequest", object? (Player player, string tier) => tier != "shout");

Events.On("voiceTalkingChanged", (Player player, bool talking) =>
{
    Log.Info($"{player.Nickname} {(talking ? "talks" : "is silent")}");
});
```

`Voice.setListenerMuted(listener, talker, true)` stops one player hearing another, and nobody else
is affected.

`Voice.setPlayerMuffled(player, 0.8)` makes a player sound muffled to everyone, as through a gag. They are
still heard; `0` gives them a clear voice again.

Each of these has its reading twin: `Voice.getRange(tier)`, `Voice.getPlayerTier(player)` and
`Voice.setPlayerTier(player, tier)`, `Voice.isPlayerMuted`, `Voice.isPlayerDeaf`, `Voice.getPlayerMuffled`,
`Voice.isListenerMuted(listener, talker)` and `Voice.isTalking(player)`.

## In a client script

`kcds.ui.LocalVoice` is the voice chat of the player's own game: whether the server has it on
(`isEnabled`), whether a microphone opened (`hasMicrophone`), whether their voice goes out now
(`isTransmitting`), and whether the server silenced them (`isMuted`) or made them hear nobody (`isDeaf`).
`getTier()` is the tier they talk in, and `requestTier(tier)` asks the server for another, which its
`voiceTierRequest` handlers may refuse. `getMode()` and `setMode('push' | 'voice')` are push to talk or voice
activation, kept on that computer. The `localVoiceTransmitting` event comes when their voice starts or stops
going out.

`LocalVoice.setZones(zones)` gives places their own sound in this game: a zone that covers the listener gives
its room's echo, and a zone between a talker and the listener muffles the talker by its `occlusion`.

```js
const { LocalVoice } = kcds.ui;

// A church that echoes, and walls that muffle a voice by half.
LocalVoice.setZones([
  {
    min: { x: 100, y: 200, z: 10 },
    max: { x: 120, y: 230, z: 25 },
    reverb: 'church',
    occlusion: 0.5,
  },
]);

kcds.core.Events.on('localVoiceTransmitting', (transmitting) => {
  console.log(transmitting ? `on air, ${LocalVoice.getTier()}` : 'quiet');
});
```

```ts
const { LocalVoice } = kcds.ui;

// A church that echoes, and walls that muffle a voice by half.
LocalVoice.setZones([
  {
    min: { x: 100, y: 200, z: 10 },
    max: { x: 120, y: 230, z: 25 },
    reverb: 'church',
    occlusion: 0.5,
  },
]);

kcds.core.Events.on('localVoiceTransmitting', (transmitting) => {
  console.log(transmitting ? `on air, ${LocalVoice.getTier()}` : 'quiet');
});
```

```lua
local LocalVoice = kcds.ui.LocalVoice

-- A church that echoes, and walls that muffle a voice by half.
LocalVoice.setZones({
  {
    min = { x = 100, y = 200, z = 10 },
    max = { x = 120, y = 230, z = 25 },
    reverb = "church",
    occlusion = 0.5,
  },
})

kcds.core.Events.on("localVoiceTransmitting", function(transmitting)
  print(transmitting and ("on air, " .. LocalVoice.getTier()) or "quiet")
end)
```

```cs
// A church that echoes, and walls that muffle a voice by half.
LocalVoice.SetZones(new[]
{
    new
    {
        min = new { x = 100, y = 200, z = 10 },
        max = new { x = 120, y = 230, z = 25 },
        reverb = "church",
        occlusion = 0.5,
    },
});

Events.On("localVoiceTransmitting", (bool transmitting) =>
{
    Log.Info(transmitting ? $"on air, {LocalVoice.GetTier()}" : "quiet");
});
```

## What it costs a server

Voice runs on one thread of its own, so it cannot slow the game. Upload per talking player is about 53 kbps
with its headers, and nothing while they are silent. A server's upload depends on how close players stand:

| 1000 players, 100 talking | One core | Upload |
|---|---|---|
| spread over the map | 2 % | under 1 Mbps |
| in a town | 6 % | 18 Mbps |
| all in one crowd | 44 % | 250 Mbps |

A connection of at least 100 Mbps is enough for most servers, and 1 Gbps for large ones with crowds. When a
very dense crowd would send more than about 250 Mbps, or voice falls behind, each player in it hears fewer of
the clearest talkers at once, until it eases.

## When it does not work

- **Nobody hears anybody:** check `kcds.players.Voice.isEnabled()`, and that players are in the same dimension
  and range. With a separate voice server, players must reach its UDP port.
- **The log says `voice: the voice server refused this server`:** the id or secret differs between the two
  servers.
- **The log says `voice: the link to the voice server was lost`:** voice stops 5 seconds later and comes back
  when the link does. Check that the game server can reach the control port over TCP.

## A separate voice server

Voice chat needs no separate server. A host with many players can still move it off the game server's
machine, so voice traffic takes nothing from the game. `kcds-voice` is that program. Voice then works
exactly as above, and the rules and events stay in the game server's resources. One voice server can
serve several game servers; their players never hear each other.

`kcds-voice` comes with the Linux server package, and as its own Docker image (`Dev/docker/voice/Dockerfile`). Give
the voice server a settings file, `kcds-voice.json`, with an id and a long random secret for each
game server it serves:

```json
{
  "bind": "0.0.0.0",
  "voicePort": 7777,
  "controlPort": 7778,
  "tenants": {
    "main": "a long random secret"
  }
}
```

The same settings can come from the environment: `KCDS_VOICE_BIND`, `KCDS_VOICE_PORT`,
`KCDS_VOICE_CONTROL_PORT` and `KCDS_VOICE_TENANTS` (`main=secret,second=secret`). Keep the secret out of
anything you publish.

Open `voicePort` (UDP) for the players, and `controlPort` (TCP) for your game servers only. When a voice
server shares a machine with a game server, give it other ports than the game's.

Then point the game server at it in `server.json`:

```json
{
  "voiceMode": "external",
  "voiceUrl": "voice.example.com:7778",
  "voiceServerId": "main",
  "voiceSecret": "a long random secret"
}
```

If players reach the voice server at another address than the game server does, also set
`voicePublicAddress` to the `host:port` they use. The address must be IPv4.

Everything between the two servers is encrypted with a key made from the secret. A wrong id or secret is
refused, and the game server's log says so: `voice: the voice server refused this server; check the id
and the secret`. It tries again every ten seconds.

If the link between them is lost, voice carries on with the last known positions and stops after 5
seconds. The game itself is not affected. When the link returns, voice comes back by itself. If the voice
server itself was restarted, every player's game makes a new encrypted session with it, without the
player doing anything.


Reference: [Voice](/docs/api/voice), [LocalVoice](/docs/api/localvoice).
