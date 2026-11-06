# A legal copy of the game

To join any KCDS server, a player needs a legally owned copy of Kingdom Come: Deliverance II. This applies
to public and private servers, LAN servers, and a server on the player's own machine.

## How it works

There is no extra sign-in or browser window.

1. When a player joins, their game asks its store for proof of the licence.
2. KCDS Master checks the proof and gives the game a short-lived pass, valid for that one server and
   connection.
3. The game server checks the pass before letting the player in.

The game server never sees a password or store token, only the store and account id from the pass.

## Stores

| Store                            | Status                                                                                         |
| -------------------------------- | ---------------------------------------------------------------------------------------------- |
| Steam                            | works and is tested in the game; Steam signs an ownership ticket and Master checks the signature |
| Epic Games Store                 | ready, not yet tested with a real Epic copy                                                    |
| Microsoft Store / Xbox app on PC | ready, not yet tested with a real Store copy                                                   |
| GOG                              | not supported yet                                                                              |

If a store cannot provide proof, the player is refused on all servers. Local files, play history and earlier
confirmations do not count. The server browser explains why ("This copy cannot be verified"): no supported
store, the store is offline or signed out, or it says the account does not own the game.

## For server owners

**Always on.** Every server requires a valid pass, whether listed on Master or not, including on localhost. No
setting turns this off, and a `"required"` key in `licence.json` is not read.

**Passes are tied to a server.** A listed server's pass names its listing. An unlisted server's pass names the
address the player's game dials, which must be one of the server's own: its network interfaces, loopback, the
host of `KCDS_SERVER_PUBLIC_ADDRESS`, or the public address Master sees. A server behind NAT needs no setting. A
pass made for another server is refused, so passes cannot be passed along.

**You can be stricter.** `licence.json` in the resources folder can limit what is accepted:

- `"stores": ["steam", "epic", "xbox"]`
- `"familySharing": false` refuses shared copies. The Microsoft Store proof cannot tell a shared copy apart, so
  that proof is then refused.

**No keys needed.** The server checks passes with Master's public key, so it needs no store or publisher key.

## Identifying a player

On the server, each `player` and each `request` in `playerConnecting` has these fields:

| Field          | Comes from                                                                    | Verified                  |
| -------------- | ----------------------------------------------------------------------------- | ------------------------- |
| `steamId`      | the pass, when the store is Steam                                             | yes                       |
| `store`        | the pass: `steam`, `epic`, `xbox` or `gog`                                    | yes                       |
| `storeAccount` | the pass: the account id in that store                                        | yes                       |
| `discordId`    | the pass once the player linked their Discord account, else their Discord app | when `discordVerified`    |
| `deviceId`     | a SHA-256 of the Windows installation id and board's UUID                     | no, the client reports it |

Since every player has a pass, `steamId`, `store` and `storeAccount` are set on every server, including
whitelisted and private ones. A Steam id the client reports itself is never treated as verified.

`player.identifiers` and `request.identifiers` give all of these as a list of `{ kind, id, verified }`:

- the store account, as `steam`, `epic`, `xbox` or `gog` (verified)
- `discord` (verified once linked, else reported) and `hardware` (reported)
- `ip` (verified)

`player.getIdentifier('steam')` returns one, or `''`.

Base your own bans on a verified kind, because a changed client can report any device id, and any
Discord id that is not linked.

### A verified Discord id

KCDS never asks a player about Discord on its own. A server with `requireDiscord` on
([Verified Discord accounts](server.md#verified-discord-accounts)) turns away players who have not
verified their Discord account, and their game offers **VERIFY DISCORD**: the Discord app asks
them to allow KCDS (their browser does when the app is not running or they close its window), and
the KCDS Master checks the answer with Discord and keeps the Discord account with their store
account. The game then joins again.

From then on their pass carries it on every server: `discordId` is that account,
`discordVerified` is `true` and the `discord` identifier is verified, also when Discord is not
running. Signed in to another Discord account, the pass leaves it out until they verify that one.
A server that does not ask for it still sees who verified:

```js
kcds.core.Events.on('playerConnecting', (request) => {
  if (request.discordVerified) {
    console.info(`${request.nickname} is Discord user ${request.discordId}`);
  }
});
```

```ts
kcds.core.Events.on('playerConnecting', (request) => {
  if (request.discordVerified) {
    console.info(`${request.nickname} is Discord user ${request.discordId}`);
  }
});
```

```lua
kcds.core.Events.on("playerConnecting", function(request)
  if request.discordVerified then
    print(request.nickname .. " is Discord user " .. request.discordId)
  end
end)
```

```cs
Events.On("playerConnecting", (JsValue request) =>
{
    if (request["discordVerified"].AsBool())
    {
        Log.Info($"{request["nickname"]} is Discord user {request["discordId"]}");
    }
});
```

```js
kcds.core.Events.on('playerConnecting', (request) => {
  if (myBans.has(`steam:${request.getIdentifier('steam')}`)) {
    request.reject('You are banned from this server.');
  }
});
```

```ts
kcds.core.Events.on('playerConnecting', (request) => {
  if (myBans.has(`steam:${request.getIdentifier('steam')}`)) {
    request.reject('You are banned from this server.');
  }
});
```

```lua
kcds.core.Events.on("playerConnecting", function(request)
  if myBans["steam:" .. request:getIdentifier("steam")] then
    request:reject("You are banned from this server.")
  end
end)
```

```cs
Events.On("playerConnecting", (JsValue request) =>
{
    if (myBans.Contains($"steam:{request.Call("getIdentifier", "steam")}"))
    {
        request.Call("reject", "You are banned from this server.");
    }
});
```

## Central bans

Besides each server's own bans, KCDS keeps one central ban list on Master, for cheating and abuse across
servers. A banned player gets no licence pass, so no KCDS server admits them, and the game shows them the reason,
and the end date of a temporary ban. Servers need do nothing for this and never look up a ban.

A central ban names a store account, the hashed device ids of machines, or both. Banning an account also bans the
machines it played from, so a new account on a banned machine is refused.

Only the KCDS operators issue and lift bans, and every ban, lift and deletion goes into an audit log. To appeal,
write to the KCDS team on our Discord with the ban id the game showed you.

## What Master stores

To run the central ban list, Master keeps one record per store account it signed a pass for:

| Data                                                     | Purpose                                                                 |
| -------------------------------------------------------- | ----------------------------------------------------------------------- |
| store and account id, such as your SteamID               | matches bans to the account                                             |
| up to 8 device ids, as SHA-256 hashes your game made     | lets a ban cover the machine; Master never sees the machine's real ids  |
| first and last join time, and number of joins            | keeps records only as long as needed                                    |
| the last 16 servers joined (listing, or address if unlisted) | helps look into a report                                            |

It stores nothing else: no IP addresses, Discord ids or names.

**How long:** a record is deleted 365 days after the account's last join. You can ask the KCDS team to delete
yours sooner, which removes it from the registry. A ban on the account itself is kept while it runs. Bans are kept
while they run and in the audit log.
