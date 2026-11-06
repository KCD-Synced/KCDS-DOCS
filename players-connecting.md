# How players connect

This page follows a player from choosing your server to standing in its world, including what can refuse them and what you can change.

## Reaching the server

| Method             | Details                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------- |
| Server browser     | the server lists itself on KCDS Master unless `master` is `off`, or it binds to `127.0.0.1` with no `publicAddress` ([Running a server](server.md#the-server-browser)) |
| Direct connect     | by address in the browser, such as `play.example.com:7777` or `203.0.113.7:7777`; works for listed and private servers |
| Join link          | `kcds://join/play.example.com:7777`, or `kcds://server/<id>` for a listed one, on your website or Discord ([Private servers and join links](server.md#private-servers-and-join-links)) |
| Launcher           | `serverAddress` in the player's `kcds-launcher.json` joins one server directly ([Getting started](getting-started.md)) |
| Your own launcher  | `kcds-launcher.exe --connect play.example.com:7777 --join-token <text>` joins directly and hands the server a text, such as a one-time ticket ([below](#a-token-from-your-own-launcher)) |

The server listens on one UDP port, 7777 by default. For players outside your network, it must be open in the firewall and forwarded on the
router.

## Checks on the way in

Any of these can refuse a player, and the server browser tells them why. The password comes before the whitelist, and both before your
`playerConnecting` handlers.

1. **KCDS version.** It must match the server's. Whichever way they joined, a player on another version is told which one the server runs, such
   as "This server runs KCDS 0.3.4.1. Your KCDS is another version: join with KCDS 0.3.4.1." The browser then offers the update, or says the
   server is on an older version.
2. **Level.** A server on `klaster` needs the game's DLC, and a player without it is told before anything loads.
3. **Legal copy.** Their game proves to KCDS Master that its store owns the game, and brings a pass for this server ([A legal copy of the
   game](licence.md)). Without one they see "This copy cannot be verified". A player on the central ban list gets no pass.
4. **Password,** if the server has one: "The gate is locked", with a password field ([Password](server.md#password)).
5. **This server's bans,** from `bans.json` and the Bans tab of [KCDS-Admin](server.md#kcds-admin). A ban on any of the player's identifiers refuses
   them, showing its reason and end.
6. **Free place.** A full server puts the player in its line ([Join queue and whitelist](join-queue.md)).
7. **Whitelist,** if there is one: only those listed by Steam id or nickname.
8. **Your resources.** All `playerConnecting` handlers run, and any can refuse with a reason or show the waiting player a line:

   ```js
   kcds.core.Events.on('playerConnecting', async (request) => {
     request.update('One moment, looking you up...');
     if (bans.has(request.steamId)) {
       request.reject('You are banned from this server.');
     }
   });
   ```

9. **Loading.** The game downloads the resources' client files, only changed ones after the first time, then loads the level behind the loading
   screen.

## A token from your own launcher

A community launcher that starts KCDS with `--connect` can add `--join-token <text>`: 1 to 255 letters, digits and `- _ . ~ + / = :`, so
base64, JWT and UUID texts fit. The game sends it to that server only, never to one picked later in the browser, and your
`playerConnecting` handlers read it as `request.joinToken`, `''` without one. The game does not check it, so your resource does,
for example against the tickets your launcher's backend issued:

```js
kcds.core.Events.on('playerConnecting', async (request) => {
  if (!(await launcherTickets.redeem(request.joinToken, request.steamId))) {
    request.reject('Start the game from our launcher.');
  }
});
```

```lua
kcds.core.Events.on("playerConnecting", function(request)
  -- a handler cannot wait, so it answers the promise: the join waits for it
  return launcherTickets:redeem(request.joinToken, request.steamId):next(function(redeemed)
    if not redeemed then
      request:reject("Start the game from our launcher.")
    end
  end)
end)
```

```cs
Events.On("playerConnecting", async (JsValue request) =>
{
    if (!await launcherTickets.Redeem(request["joinToken"].AsString(), request["steamId"].AsString()))
    {
        request.Call("reject", "Start the game from our launcher.");
    }
});
```

## Events on arrival

Once admitted, a player's connection goes through these events in order:

| Event              | When                                                                  | Typical use                                      |
| ------------------ | --------------------------------------------------------------------- | ------------------------------------------------ |
| `playerConnect`    | the connection exists and the inventory too, but the body is loading  | restore a saved inventory                        |
| `playerSpawning`   | their game waits behind the loading screen for a place                | choose it with `player.spawn(position, rotation)` |
| `playerSpawned`    | the body stands in the world                                          | dress them, set their nameplate                  |
| `playerReady`      | the loading screen is down and they can play; also fires in their game | greet them, open a menu                         |
| `playerDisconnect` | they left, but the object still has its last values                   | save what you keep                               |

```js
const { Events, Chat } = kcds.core;

Events.on('playerSpawning', (player) => {
  player.spawn(new kcds.math.Vector3(1652.5, 2151.5, 37.5), { z: 90 });
});

Events.on('playerReady', (player) => {
  Chat.sendToPlayer(player, `Welcome, ${player.nickname}!`);
});

Events.on('playerDisconnect', (player) => {
  savedInventories.set(player.steamId, kcds.players.Inventory.get(player));
});
```

```lua
local Events, Chat = kcds.core.Events, kcds.core.Chat

Events.on("playerSpawning", function(player)
  player:spawn(kcds.math.Vector3.new(1652.5, 2151.5, 37.5), { z = 90 })
end)

Events.on("playerReady", function(player)
  Chat.sendToPlayer(player, "Welcome, " .. player.nickname .. "!")
end)

Events.on("playerDisconnect", function(player)
  savedInventories[player.steamId] = kcds.players.Inventory.get(player)
end)
```

```cs
Events.On("playerSpawning", (Player player) =>
{
    player.Spawn(new Vector3(1652.5, 2151.5, 37.5), new { z = 90 });
});

Events.On("playerReady", (Player player) =>
{
    Chat.SendToPlayer(player, $"Welcome, {player.Nickname}!");
});

Events.On("playerDisconnect", (Player player) =>
{
    savedInventories[player.SteamId.AsString()!] = Inventory.Get(player);
});
```

## Identifying the player

`player.steamId`, or `epicId` for a player who owns the game on Epic, `store` and `storeAccount` come from the licence pass and are verified.
`discordId` is verified when `discordVerified` is true (the player linked their Discord account through the KCDS
Master), else it is what the player's game reports, as `deviceId` always is. `player.identifiers` lists them all ([Identifying a
player](licence.md#identifying-a-player)). Use a verified one for your bans and saved data.

## Troubleshooting

| The player sees                  | Likely cause                                                                                                  |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| the server is not in the browser | `master` is `off`, it binds to `127.0.0.1`, or Master cannot reach it; check for `MASTER listed` in the server log |
| a time-out when joining          | the UDP port is not open or forwarded, or the address is wrong                                                |
| "This copy cannot be verified"   | their store is offline or signed out, or the copy is not supported ([Stores](licence.md#stores))              |
| "The gate is locked"             | the server has a password                                                                                     |
| a place in a queue               | the server is full (`maxPlayers`)                                                                             |
| a refusal with your text         | one of your `playerConnecting` handlers refused them                                                          |

## The server's refusal texts

These are the exact texts, in the order the server checks. The browser's join popup shows a title for the kind of refusal, such as "The gate is
locked" with a password field, or "Turned away at the gates". The server's text appears below it, in a box headed THE REALM SAYS.

| Check                                                 | The player is told                                                                     |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------- |
| No licence pass                                       | This server only lets in a legally owned game: join through the KCDS launcher.         |
| A broken pass                                         | Your licence pass is not valid. Rejoin through the KCDS launcher.                      |
| A pass the Master did not sign                        | Your licence pass was not signed by the KCDS Master. Rejoin through the launcher.      |
| An old pass                                           | Your licence pass expired or has an invalid lifetime. Rejoin through the launcher.     |
| A pass for another server                             | Your licence pass is for another server. Rejoin through the KCDS launcher.             |
| A pass for another connection                         | Your licence pass is for another connection. Rejoin through the KCDS launcher.         |
| A pass used twice                                     | Your licence pass was already used. Rejoin through the KCDS launcher.                  |
| A store the server does not take (`licence.stores`)   | This server does not let in games from your store.                                     |
| A store that could not confirm the licence            | Your store could not confirm a current game licence.                                   |
| A family-shared copy (`licence.familySharing: false`) | This server does not let in family-shared copies.                                      |
| Sharing not known for the store                       | This server cannot confirm unshared ownership for this store.                          |
| The server's licence settings are broken              | This server cannot validate game licences because its configuration is invalid.        |
| Too many passes at once                               | This server cannot validate another licence pass yet. Please try again shortly.        |
| A password and none given                             | This server asks for a password.                                                       |
| A wrong password                                      | Wrong password.                                                                        |
| A ban                                                 | You are banned from this server: Cheating. Until 2026-10-26 21:00 UTC. (a permanent ban has no "Until") |
| Not on the whitelist                                  | The whitelist's `reason`, or: This server only admits listed players.                  |
| `request.reject(reason)`                              | Your `reason`, cut to 512 bytes.                                                       |
| A `playerConnecting` handler that throws              | No text; the server log has the error.                                                 |

Each refusal goes to the server log with its cause, for example `playerConnecting refused request 3: licence wrongServer`, `... : wrong password`,
`... : banned (2ad9433eed394c0c)` or `... : not on the whitelist`.

Reference: [Bans](/docs/api/bans).
