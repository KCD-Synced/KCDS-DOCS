# Running a server

A KCDS server is a small native program with a Node.js host for your resources. It uses one port number, UDP for the game and TCP for
the resources' files, and appears in the in-game server browser.

## Requirements

The server runs on Linux and in Docker. There is no Windows server package; players still play on Windows.

- Linux x64 with glibc 2.36 or newer and libcurl
- Node.js 22 or newer on `PATH`, with 26 LTS recommended (<https://nodejs.org/>)
- The server from the KCDS release your players use

Client and server must come from the same release. Each proves to the other that it is an official build of that release, so an older or
self-built client is refused at connection.

## Starting it

1. Download the Linux server, `KCDS-Server-<version>-linux.tar.gz`, from the download page and unpack it.
2. Run `./start-server.sh`. It listens on UDP port 7777 on all network interfaces, and on TCP 7777 for the resources' files.
3. For players outside your network, open 7777 for UDP and TCP in the firewall and forward both on your router. Without TCP the game
   still gets the files, over the game connection, only slower.

You can then type [console commands](#console-commands) into the server's terminal.

**In Docker**, the server image runs the same server. Publish 7777 for UDP and TCP (`-p 7777:7777/udp -p 7777:7777/tcp`), set the settings with the `KCDS_SERVER_*`
variables or mount your `server.json` at `/opt/kcds/server.json`, and mount your resources over `/opt/kcds/resources`.

## Settings: server.json

All settings live in `server.json`, next to `start-server.sh`. Change it, then start the server again.

```json
{
  "port": 7777,
  "bind": "0.0.0.0",
  "level": "trosecko",
  "name": "Rattay Roleplay",
  "maxPlayers": 64,
  "password": "only for friends",
  "httpPort": 30120,
  "adminPort": 30121,
  "adminPassword": "a long owner password",
  "restartTimes": ["06:00", "18:00"]
}
```

If a setting is given in several places, the first of these wins:

1. the command line: `--port`, `--bind`, `--level`, `--node`, `--resource-host`, `--resources`, `--config <file>`. The older form,
   `./kcd2s-server <port> <bind> <level> [<node> <main.cjs> <resources>]`, still works.
2. its environment variable
3. `server.json`
4. the default

| Key                                   | Variable                           | Default                                    | Meaning                                                  |
| ------------------------------------- | ---------------------------------- | ------------------------------------------ | -------------------------------------------------------- |
| `port`                                | `KCDS_SERVER_PORT`                 | `7777`                                     | the UDP port, and TCP for the resources' files           |
| `bind`                                | `KCDS_SERVER_BIND`                 | `127.0.0.1` (`0.0.0.0` in the shipped file) | the interface; `0.0.0.0` means all                      |
| `level`                               | `KCDS_SERVER_LEVEL`                | `trosecko`                                 | `trosecko`, `kutnohorsko`, `klaster` or a World Architect custom map, `kcds_map_<name>` ([Custom maps](world-architect.md#custom-maps)); any other name stops the server at once. `klaster` needs a DLC, and a custom map comes to players with the resource World Architect exports; a player without the level is told in the browser |
| `node`, `resourceHost`, `resources`   | `KCDS_NODE`, -, `KCDS_RESOURCES`   | from the start script                      | Node.js, the resource host (`runtime/.../main.cjs`) and the resources folder; without all three, no resources run |
| `name`                                | `KCDS_SERVER_NAME`                 | `KCDS server`                              | the browser name, up to 64 characters                    |
| `maxPlayers`                          | `KCDS_SERVER_MAX_PEERS`            | `32`                                       | most players at once, 1 to 1024                          |
| `password`                            | `KCDS_SERVER_PASSWORD`             | none                                       | see [Password](#password)                                |
| `requireDiscord`                      | `KCDS_REQUIRE_DISCORD`             | `off`                                      | see [Discord](#verified-discord-accounts)                |
| `master`                              | `KCDS_MASTER_URL`                  | `https://master.kcdsynced.com`             | `off` keeps the server private                           |
| `publicAddress`                       | `KCDS_SERVER_PUBLIC_ADDRESS`       |                                            | the `host:port` players dial                             |
| `image`                               | `KCDS_SERVER_IMAGE_URL`            |                                            | the browser's card art, an `https://` address            |
| `httpPort`, `httpBind`                | `KCDS_HTTP_PORT`, `KCDS_HTTP_BIND` | off, all interfaces                        | resources' HTTP routes and the [status API](#status-api) |
| `publicPlayerList`                    | `KCDS_PUBLIC_PLAYER_LIST`          | `true`                                     | `false` keeps `/players.json` private                    |
| `revealMap`                           | `KCDS_SERVER_REVEAL_MAP`           | `true`                                     | lifts the fog and discovers all places for all players. With `false`, players find places by walking there, firing `locationDiscovered` and `poiDiscovered` in client scripts |
| `levelPeople`                         | `KCDS_SERVER_LEVEL_PEOPLE`         | `false`                                    | the game's own people of the level, with their daily routines, the same for all players; see [The level's people](npcs.md#the-levels-people) |
| `npcHouses`                           | `KCDS_SERVER_NPC_HOUSES`           | `true`                                     | the level's people own houses; `false`: at night they walk out of the map along its roads and come back in the morning; see [Where the people spend the night](npcs.md#where-the-people-spend-the-night) |
| `adminPort`, `adminPassword`, `adminBind`, `adminTlsCert`, `adminTlsKey` | `KCDS_ADMIN_*` | off, -, `127.0.0.1`      | [KCDS-Admin](#kcds-admin)                                |
| `restartTimes`                        | `KCDS_RESTART_TIMES`               | none                                       | daily restarts, such as `["06:00", "18:00"]`             |
| `voiceMode`, `voiceUrl`, `voiceServerId`, `voiceSecret`, `voicePublicAddress` | `KCDS_SERVER_VOICE_MODE`, `_URL`, `_ID`, `_SECRET`, `_PUBLIC` | `embedded` | voice chat on [a separate voice server](voice.md#a-separate-voice-server) |
| `tickRate`, `interestRadius`, `interestHysteresis`, `maxCharacters`, `snapshotWorkers`, `metrics` | `KCDS_SERVER_TICK_HZ` and others | see [Capacity and tuning](#capacity-and-tuning) |          |

An unknown key, or a value of the wrong type, is reported at startup and ignored. The log line `CONFIG server.json: N settings` says how many the
file set.

## The server browser

The server lists itself through <https://master.kcdsynced.com>, with its name, map, players, version and address. The listing refreshes every 15
seconds and is removed when the server stops.

| Variable                     | Meaning                                                                                           |
| ---------------------------- | ------------------------------------------------------------------------------------------------- |
| `KCDS_SERVER_NAME`           | the name players see, up to 64 characters                                                         |
| `KCDS_SERVER_PUBLIC_ADDRESS` | the `host:port` players dial. Without it, the listing uses your public IP and the server's port. Needed behind a proxy or for a DNS name |
| `KCDS_SERVER_IMAGE_URL`      | optional `https://` address of your card picture                                                  |
| `KCDS_MASTER_URL`            | `off` means not listed; another address lists it on that master instead                           |

A server bound to `127.0.0.1` is listed only if `KCDS_SERVER_PUBLIC_ADDRESS` is set. Each of these can also be set in `server.json`.

## KCDS-Admin

Each server has a built-in admin page for your browser. It stays off until you set both the port and the password:

| Variable                                    | Meaning                                                                                                     |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `KCDS_ADMIN_PORT`                           | the TCP port, such as `30121`                                                                               |
| `KCDS_ADMIN_PASSWORD`                       | the owner's password, at least 8 characters; you sign in as `owner`                                         |
| `KCDS_ADMIN_BIND`                           | optional interface; `127.0.0.1`, only this computer, by default; `0.0.0.0` for all                          |
| `KCDS_ADMIN_TLS_CERT`, `KCDS_ADMIN_TLS_KEY` | optional PEM certificate and key files, to serve over HTTPS                                                 |

If the page can be reached from the internet, use HTTPS, or a reverse proxy with it.

Open `http://127.0.0.1:30121/` and sign in.

### Tabs

**Status:** players, level, version, uptime, password and join link.

**Players:** everyone online, with ping and identifiers. You can message, teleport, kick or ban them. A ban covers all identifiers except the address,
for a set time or permanently.

**Bans:** this server's bans, from `bans.json` in the resources folder. Add one by identifiers such as `steam:7656...` or `hardware:...`, or lift one. A
banned player is told the reason and end time.

**Console:** the live log. The owner can also run console commands, such as `restart my-gamemode` or your resources' own.

**Resources** (owner): start, stop or restart each one.

**Accounts** (owner): add moderators. A moderator can see everything and act on players and bans, but cannot run commands or change resources or accounts.
Passwords are stored only as a scrypt hash in `admin-accounts.json`.

**Settings** (owner) has several parts:

- **Editor:** change `server.json`, then save, or save and restart in a minute. It must be one JSON object, and it applies at the next start.
- **Where the server listens:** sets the game's port and interface (`port`, `bind`), the admin's port and interface (`adminPort`, `adminBind`), and the status API's
  port (`httpPort`). A change is written into the editor, ready to save. After changing the admin's port or address, the page answers there once the server has
  restarted, so open it and sign in again. A command-line option or environment variable with the same setting wins over `server.json`.
- **Backups:** copies the resources folder with its data, `server.json` and the bans to `backups/<date-time>/` next to the resources folder, either now or before each
  planned stop or restart. It leaves out `node_modules` and files over 64 MiB, and keeps the newest 10.
- **Crash reports:** a run that ended abnormally, through a crash or killed process, is written to `crashes/<date-time>.json` next to the resources folder, with its last
  200 log lines. Each opens from the list, and the newest 20 are kept.
- **Updates:** checks the KCDS update feed and installs a newer server with one click. Changed files (the server, `runtime/`, `README.txt`) are downloaded and checked
  against the release's signature, and the server restarts a minute later, with players warned. `server.json`, resources and the start script are never changed.
  `KCDS_UPDATE_CHANNEL=preview` follows preview releases. This is for the Linux server package; it is off in Docker, where you pull the new image.

### Sign-in and sessions

Five wrong passwords from one address lock it out for 5 minutes. The sign-in page shows the tries left and when it unlocks. A session lasts 12 hours, and the header shows
until when.

A planned restart or stop appears as a banner with a countdown on all screens, including for moderators, but only the owner can cancel it. The page also works on a phone,
where the sections fold into one menu. All admin actions go to the server log with the account that did them.

### The admin API

The page uses a JSON API on the same port, which you can also script, for a Discord bot or cron job for example. Sign in once and send the session cookie with each
request. A request that changes something (POST, PUT, DELETE) must have the header `X-KCDS-Admin: 1`, which pages from other sites cannot add. The body is JSON of up to 64
KiB.

```sh
curl -c admin.txt -H "X-KCDS-Admin: 1" -H "content-type: application/json"   -d '{"name":"owner","password":"..."}' http://127.0.0.1:30121/api/login
curl -b admin.txt http://127.0.0.1:30121/api/players
curl -b admin.txt -H "X-KCDS-Admin: 1" -H "content-type: application/json"   -d '{"text":"Server restarts at 20:00"}' http://127.0.0.1:30121/api/players/1/message
```

| Route                                                  | Who       | Body                             | Answer                                                                          |
| ------------------------------------------------------ | --------- | -------------------------------- | ------------------------------------------------------------------------------- |
| `POST /api/login`                                      | anyone    | `{name, password}`               | the session, as `GET /api/session`, and the `kcds_admin` cookie (HttpOnly, SameSite=Strict, 12 hours) |
| `POST /api/logout`                                     | signed in |                                  | `{ok: true}`                                                                    |
| `GET /api/session`                                     | signed in |                                  | `{name, role, expires, listen}`; `role` is `owner` or `moderator`, and `listen` is where the admin answers, `{scheme, bind, port}` |
| `GET /api/status`                                      | signed in |                                  | the [status API](#status-api)'s `info.json`                                     |
| `GET /api/players`                                     | signed in |                                  | `[{id, name, ping, ip, identifiers, position}]`                                 |
| `POST /api/players/{id}/message`                       | signed in | `{text}`                         | `{ok: true}`; a chat line from "Admin"                                          |
| `POST /api/players/{id}/teleport`                      | signed in | `{x, y, z}`                      | `{ok: true}`                                                                    |
| `POST /api/players/{id}/kick`                          | signed in | `{reason}`                       | `{ok: true}`; the reason, by default "Kicked by an admin.", is cut to 200 bytes |
| `POST /api/players/{id}/ban`                           | signed in | `{reason, minutes}`              | `{ok: true}`; bans all identifiers except the address, which could hit strangers behind a shared router, and kicks them with the ban's text; `minutes: 0` is permanent |
| `GET /api/bans`                                        | signed in |                                  | `[{id, identifiers, reason, by, at, until, active}]`                            |
| `POST /api/bans`                                       | signed in | `{identifiers, reason, minutes}` | `{ok: true}`; 1 to 8 identifiers such as `steam:7656...`                        |
| `DELETE /api/bans/{id}`                                | signed in |                                  | `{ok: true}`                                                                    |
| `GET /api/log?after=N`                                 | signed in |                                  | `{lines: [{seq, time, source, text}]}`, the lines after `seq` N                 |
| `POST /api/command`                                    | owner     | `{text}`                         | `{ok: true}`; runs a [console command](#console-commands)                       |
| `GET /api/resources`                                   | owner     |                                  | `[{name, version, running}]`                                                    |
| `POST /api/resources/{name}/start` (`stop`, `restart`) | owner     |                                  | `{ok: true}`                                                                    |
| `GET /api/config`                                      | owner     |                                  | `{path, text, exists}`, with `server.json` as text                              |
| `PUT /api/config`                                      | owner     | `{text}`                         | `{ok: true}`; refused unless the text is one JSON object                        |
| `GET /api/backups`, `POST /api/backups`                | owner     |                                  | the backups; a new one gives `{name, bytes, skipped}`                           |
| `GET /api/crashes`, `GET /api/crashes/{name}`          | owner     |                                  | `[{name, kind, startedAt, endedAt}]`; one report with its last log lines        |
| `GET /api/updates`                                     | owner     |                                  | `{available, reason, current, channel, staged}`                                 |
| `POST /api/updates/check`                              | owner     |                                  | `{release, sequence, newer}`                                                    |
| `POST /api/updates/install`                            | owner     | `{minutes}`                      | downloads the release, then restarts into it in `minutes`                       |
| `GET /api/server`                                      | signed in |                                  | `{pending}`: the planned stop or restart, or `null`                             |
| `POST /api/server/restart` (`stop`)                    | owner     | `{minutes}`                      | `{pending: {restart, at, by, daily}}`                                           |
| `POST /api/server/cancel`                              | owner     |                                  | `{ok: true}`                                                                    |
| `GET /api/accounts`, `POST /api/accounts`              | owner     | `{name, password}`               | `[{name, role}]`; `{ok: true}`                                                  |
| `DELETE /api/accounts/{name}`                          | owner     |                                  | `{ok: true}`; their sessions end at once                                        |

### Example answers

Times in `at`, `until`, `time` and `expires` are milliseconds since 1970.

`POST /api/login {"name": "owner", "password": "..."}`, and `GET /api/session`:

```json
{
  "name": "owner",
  "role": "owner",
  "expires": 1793068145011,
  "listen": { "scheme": "http", "bind": "127.0.0.1", "port": 30121 }
}
```

A wrong password:

```json
{ "error": "wrong name or password", "triesLeft": 1 }
```

The sixth try from one address within 5 minutes, also with the header `Retry-After: 141`, the seconds until it unlocks:

```json
{ "error": "too many failed sign-ins; wait 5 minutes", "retryAfter": 141 }
```

`GET /api/players`:

```json
[
  {
    "id": 1,
    "name": "Henry",
    "ping": 42,
    "ip": "203.0.113.7",
    "identifiers": [
      { "kind": "steam", "id": "76561198000000001", "verified": true },
      { "kind": "hardware", "id": "3f2a3f2a...", "verified": false },
      { "kind": "ip", "id": "203.0.113.7", "verified": true }
    ],
    "position": { "x": 1234.5, "y": 820.25, "z": 31 }
  }
]
```

`GET /api/bans`, after `POST /api/players/1/ban {"reason": "Cheating", "minutes": 60}`:

```json
[
  {
    "id": "2ad9433eed394c0c",
    "identifiers": ["steam:76561198000000001", "hardware:3f2a3f2a..."],
    "reason": "Cheating",
    "by": "owner",
    "at": 1793010558544,
    "until": 1793014158544,
    "active": true
  }
]
```

`GET /api/log?after=4`:

```json
{
  "lines": [
    {
      "seq": 5,
      "time": 1793010492309,
      "source": "host",
      "text": "KCDS-Admin: owner signed in (owner) from 127.0.0.1"
    },
    {
      "seq": 6,
      "time": 1793010492314,
      "source": "host",
      "text": "KCDS-Admin: owner messages Henry (1): Server restarts at 20:00"
    }
  ]
}
```

`GET /api/resources`:

```json
[{ "name": "my-gamemode", "version": "1.2.0", "running": true }]
```

`POST /api/backups`:

```json
{ "name": "20261026-102812", "bytes": 360, "skipped": [] }
```

`GET /api/updates`, on a server not started from a KCDS server package:

```json
{
  "available": false,
  "reason": "this server does not run from a KCDS server package",
  "current": "",
  "channel": "server-linux-stable",
  "staged": null
}
```

`POST /api/server/restart {"minutes": 5}`:

```json
{ "pending": { "restart": true, "at": 1793010792348, "by": "owner", "daily": false } }
```

### Refusals

A refused request gets an HTTP status with `{error}`:

| Status | `error`                                                                                                                       |
| ------ | ----------------------------------------------------------------------------------------------------------------------------- |
| 400    | the request's fault, such as `not valid JSON: ...` (a `PUT /api/config`), `a message of 1 to 512 bytes`, `a position x, y, z in the world`, `A password of 8 to 128 bytes`, `A ban needs 1 to 8 identifiers, a reason and its length in minutes` |
| 401    | `sign in first`; at sign-in, `wrong name or password` with `triesLeft`                                                        |
| 403    | `missing X-KCDS-Admin header`; `only the owner may do this` for a moderator                                                   |
| 404    | `no such player`, `no such ban`, `not found`                                                                                  |
| 429    | `too many failed sign-ins; wait 5 minutes` after 5 failures from one address in 5 minutes, with `retryAfter` and the header `Retry-After` in seconds |

### Restarts

The owner can restart or stop the server from the Status tab, immediately or in 1, 5 or 15 minutes. `KCDS_RESTART_TIMES=06:00,18:00` restarts it daily at those times,
by the server computer's clock.

Players are told in chat 15, 5 and 1 minute before, and 10 seconds before. A planned restart can be cancelled.

A restart ends with exit code 75, and `start-server.sh` and the Docker image then start the server again. Any other exit, or a stop signal, stops it
for good.

## Private servers and join links

A private server is one the browser does not list, because of `KCDS_MASTER_URL=off` or a bind to `127.0.0.1`. Players join by address through Direct connect, or by a join
link. It still admits only legally owned games, like any KCDS server.

A join link opens KCDS and joins from the browser, so the password prompt and refusals appear there as usual:

| Link                                | Joins                                                                                             |
| ----------------------------------- | ------------------------------------------------------------------------------------------------- |
| `kcds://join/play.example.com:7777` | that address, listed or not                                                                       |
| `kcds://server/<id>`                | the listed server with that id; the server log shows `join link kcds://server/<id>` once listed, and `info.json` has it as `joinLink` |

Put a link on your website or Discord: `<a href="kcds://join/play.example.com:7777">`. The launcher registers `kcds://` links for the player's Windows account at each start,
so they work once KCDS has run once. A link only names a server and possibly its version. It never carries a password or other setting.

### Staying on an older release

A link can name the KCDS version your server runs: `kcds://join/play.example.com:7777?version=0.3.4.1`. The launcher then starts that version, installing it from the update
feed if needed, so a server on an older release can still be joined after a new one comes out. The next start without a version returns to the newest.

Every release from 0.3.4.1 on is in the feed. For any other version, the player is told the name, and no client starts. Other launchers can do the same with
`kcds-launcher.exe --connect play.example.com:7777 --version 0.3.4.1`. With `--join-token <text>` the server's resources also get a text
from that launcher as `request.joinToken` ([How players connect](players-connecting.md#a-token-from-your-own-launcher)).

## Verified Discord accounts

`"requireDiscord": true` (`KCDS_REQUIRE_DISCORD=on`) admits only players who verified their Discord account. A player who has not
sees "The gatekeeper asks for your Discord" with **VERIFY DISCORD**. The Discord app asks them to allow KCDS, or their browser does when
the app is not running or they close its window; once they allow it, the game joins again by itself. Players verify once: every server that asks for it lets them in from then on.

The check comes after the password and the server's bans, before the whitelist and your `playerConnecting` handlers. Off, nobody is
asked to verify anything, and `discordVerified` tells your resources who did ([Identifying a player](licence.md#a-verified-discord-id)).

## Password

`KCDS_SERVER_PASSWORD`, 1 to 64 characters, admits only players who type it. The browser marks the server PASSWORD and asks before the first try. A player joining by address
without it, or with a wrong one, sees "The gate is locked" with a password field, and joins once they enter the right one.

The password is checked before the whitelist and before your `playerConnecting` handlers, and `info.json` says `"password": true`.

**How it is protected:** the game sends a SHA-256 proof made with the server's one-time handshake number, not the password, so a proof works for only that connection. The
password is never written to the log or passed to your resources' events. An invalid value, too long or with control characters, lets nobody in and says so in the log, so the
server is not left open.

## Status API

With `KCDS_HTTP_PORT` set, which is also the port of your resources' routes (see [Http](/docs/api/http)), the server answers three read-only JSON addresses, for your website,
a Discord bot or a server list. `KCDS_HTTP_BIND` picks the interface, by default all of them.

| Address             | Answer                                                                                                     |
| ------------------- | ---------------------------------------------------------------------------------------------------------- |
| `GET /info.json`    | `name`, `image`, `level`, `players`, `maxPlayers`, `version`, `protocol`, `password` (whether joining asks for one), `playerList` (whether `/players.json` is public), `licence` (`stores` and `familySharing`), `resources` (those running), `joinLink` (`""` while unlisted), `uptime` (seconds) |
| `GET /dynamic.json` | `name`, `level`, `players`, `maxPlayers`: what changes, for polling                                        |
| `GET /players.json` | the scoreboard, `[{ "id": 3, "name": "Hans" }, ...]`; `KCDS_PUBLIC_PLAYER_LIST=off` makes it private (403) |

```text
> curl http://my-server:30120/dynamic.json
{"name":"Rattay Roleplay","level":"trosecko","players":12,"maxPlayers":64}
```

`info.json` for a server with two players:

```json
{
  "name": "Rattay Roleplay",
  "level": "trosecko",
  "players": 2,
  "maxPlayers": 64,
  "image": "https://example.com/card.png",
  "version": "0.3.2",
  "protocol": 2,
  "password": false,
  "playerList": true,
  "licence": { "stores": ["steam", "gog", "epic", "xbox"], "familySharing": true },
  "resources": ["my-gamemode"],
  "joinLink": "kcds://server/6180b06a1f1bdefb",
  "uptime": 5400
}
```

No answer contains anything secret, such as a password, a player's address or identifier, or any other setting. Any website can read them
(`Access-Control-Allow-Origin: *`). Answers are never cached, and one address can ask at most 20 times per second.

## Capacity and tuning

All of these are optional. An invalid value is reported at startup and ignored.

| Variable                          | Default                              | Meaning                                                                                   |
| --------------------------------- | ------------------------------------ | ----------------------------------------------------------------------------------------- |
| `KCDS_SERVER_MAX_PEERS`           | 32                                   | most players at once, 1 to 1024; others are refused as "server full"                      |
| `KCDS_SERVER_TICK_HZ`             | 60                                   | the server's tick rate, 10 to 240                                                         |
| `KCDS_SERVER_INTEREST_RADIUS`     | 150                                  | metres within which a player receives other players and entities                          |
| `KCDS_SERVER_INTEREST_HYSTERESIS` | 25                                   | extra metres before something leaves a player's view                                      |
| `KCDS_SERVER_MAX_CHARACTERS`      | 0                                    | per-player budget for players, NPCs and horses, including their own character; 0 turns it off, otherwise 1 to 4096 |
| `KCDS_SERVER_SNAPSHOT_WORKERS`    | CPU threads minus one (2 to 16)      | threads building world updates; 0 or 1 uses one thread                                    |
| `KCDS_SERVER_METRICS`             | off                                  | prints server metrics at this interval in seconds                                         |

### The character budget

`maxCharacters` in `server.json` sets the same budget. It changes which characters a player receives, not how many people can connect. It is opt-in, because no value has been
measured as safe for every area, outfit, hardware setup and mix of players, NPCs and horses. A larger number does not mean that many visible characters are supported. The
level's own population comes on top of it.

**Who is chosen:** a player's own character always stays. Current combat opponents, owned characters and characters simulated by that client come first, and the rest go by
distance. Characters already visible get the `interestHysteresis` distance as an advantage, which reduces changes near the boundary. A mounted player and their horse compete
together when both are in view. A budget of 1 leaves room only for the local player. Dimension and private visibility rules still apply, including to owned characters.

**Who is left out:** characters beyond the budget remain on the server and return with current state when selected again. NPCs and horses with no eligible client to simulate them
stay at their last reported state.

**Departures:** a departing character keeps its slot until the client confirms removal, so under packet loss, fewer characters can briefly be visible.

**Measuring:** with `KCDS_SERVER_METRICS` on, the `KCDS_METRICS_CONFIG` line records `max_characters`, and each `KCDS_METRICS` line has `characters`: `candidates`, `admitted`,
`resident`, `retiring` and `deferred`. These are totals across clients, so a character seen by two players counts twice. `resident` counts reserved slots, including removals
awaiting confirmation, and is not a count of loaded models. `deferred` shows characters held back by the budget, separately from bandwidth deferrals. Measure a typical busy area
before choosing a budget for production.

### Idle players

An unchanged, verified idle player sends a complete owner report every 100 ms. The server shares its idle clock and animation progress every second. Advancing playback in the same
idle animation does not cause extra reports, and refreshes include the current progress.

Movement, turning, animations starting or stopping, other state changes and authority changes skip this interval. Combat, riding, aiming and incomplete activity reports do not
qualify as idle. Complete refreshes still recover lost reports, and transport acknowledgements and connection liveness continue independently.

### Update cache metrics

The `snapshots` metrics include `update_cache_hits`, `update_cache_misses` and `update_cache_fallbacks`, counting update encodings that were reused, newly cached or handled without
the cache during the interval. To get the reuse rate, divide hits by the sum of the three. Fallbacks do not discard updates or change delivery rules.

`update_cache_busy_fallbacks`, `update_cache_variant_fallbacks`, `update_cache_memory_fallbacks` and `update_cache_phase_fallbacks` separate concurrent work, cache entry limits,
memory limits and an unavailable cache phase. They add up to `update_cache_fallbacks`, and a high rate helps explain reduced reuse.

Snapshot entries of equal priority keep a stable order across worker counts, so changing `snapshotWorkers` does not change visibility, ownership or acknowledgement requirements.

### Snapshot bandwidth

Bandwidth limits defer entries that do not fit. After enough credit or waiting time, the highest-priority entry can exceed the limit as the only optional entry in that snapshot, so
smaller updates cannot hold up a large new character forever. Later snapshots account for the extra.

`snapshots.deferred_constructs` counts deferred attempts, including repeats for one character, and not distinct missing characters. When measuring a crowded scene, check delivered
character counts as well.

## Console commands

| Command                                   | Effect                                                                                     |
| ----------------------------------------- | ------------------------------------------------------------------------------------------ |
| `start <resource>`                        | starts a resource                                                                          |
| `stop <resource>`                         | stops a resource and those depending on it                                                 |
| `restart <resource>`, `ensure <resource>` | restarts a resource and its running dependents; players switch client parts without restarting the game |
| `refresh`                                 | finds new and changed manifests, but starts nothing new                                    |
| `refreshall`                              | stops all resources, reads the manifests again, and restarts those that were running       |

Any other command reaches your resources as the `consoleCommand` event; see [Timers, logging and the runtime](timers.md).

## Resources

Each folder in `resources/` whose `package.json` has a `"kcds"` section starts with the server.
<!-- only examples -->
Two examples come with it:

- `resources/freeroam`: greetings, `/help`, `/players`, `/tp`, `/horse`, appearance commands, the character creator, and NoClip on F3
- `resources/dice`: two players at a dice table get a key hint to start a match
<!-- end -->

Change them or add your own; see [Resources](resources.md).

> **Run only resources you trust.** The resource host runs them with the server's rights.

The resource command inbox keeps messages in order, and holds up to 16,384 messages or 16 MiB of encoded payload, which handles join bursts without silently losing spawn or other
commands. If either limit is exceeded, the server reports `RESOURCE IPC inbox capacity exceeded` and exits with an error. Treat that run as failed, and look for excessive resource
output before restarting. The limits bound the queued payload, not the server's total memory use.

## Optional server files

- `resources/item-config.json` changes the game's items on your server: weight, price, the most units a player can hold, and stats. See [Items](/docs/api/items).
- `resources/level-items.json` with `{ "enabled": false }` starts the server with the level's own items, such as tankards, tools and shop goods, removed for all players. Scripts can
  switch this with `kcds.world.World.setLevelItemsEnabled`.
- World exports, `<name>.world.json`, spawn props, effects, level edits, areas and patrol routes at startup. List them in `KCDS_WORLD_RESOURCES`; see
  [WorldResource](/docs/api/worldresource).
