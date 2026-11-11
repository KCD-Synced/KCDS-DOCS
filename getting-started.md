# Getting started

KCDS (KCD2-Synced) adds multiplayer to Kingdom Come: Deliverance II. People host servers, and each
server runs resources, which are JavaScript or TypeScript packages that define its rules, commands and
content. This page takes you from installation to playing.

> This is an early test build, so expect bugs. Please report them in the game or on our Discord.

## What you need

- 64-bit Windows 10 or 11
- A legal copy of Kingdom Come: Deliverance II, game version 1.5.5
- Microsoft Edge WebView2 Runtime, which Windows 11 already includes

Steam copies are tested. Epic Games Store and Microsoft Store copies are supported but have not yet been
tested with a real copy. GOG is not supported yet. See [Licence](licence.md).

## Installing

1. Download `KCDS-Setup-<version>.exe` from the download page and run it. It installs for your Windows
   account only, needs no administrator rights, and adds KCDS to the Start menu. It goes in
   `%LOCALAPPDATA%\Programs\KCDS`. If you prefer to install by hand into any folder, the zip on the same
   page has the same files.
2. Open your store's launcher (Steam), then start KCDS from the Start menu. The launcher keeps itself
   and the client updated, and adds a desktop shortcut the first time.
3. The launcher finds games installed through Steam or in a default folder. If it shows "Kingdom Come:
   Deliverance II not found", press **LOCATE GAME FOLDER** and select the folder once.

The first start after a game or KCDS update takes a few seconds longer, because the client looks up the
game's functions once and stores them in `%LOCALAPPDATA%\KCDS\signatures.txt`.

### Updating and removing

Running a newer installer over an older one keeps your settings. To uninstall, use Windows' installed
apps or the Start menu entry. Your logs and crash reports stay in `%LOCALAPPDATA%\KCDS`.

### Launcher settings

`kcds-launcher.json` in the install folder holds two settings. `gameExecutable` is the full path to
`KingdomCome.exe`, if you want to set it yourself, with doubled backslashes. `serverAddress` is described
below.

```json
{
  "gameExecutable": "D:\\Games\\KingdomComeDeliverance2\\Bin\\Win64MasterMasterSteamPGO\\KingdomCome.exe",
  "serverAddress": ""
}
```

## Joining a server

Leave `serverAddress` empty and choose a server in the game's server browser.

To go straight to one server, set `serverAddress` to its address, such as `"203.0.113.7:7777"`. The game
opens KCDS's menu and joins from there. You see the connecting screen and any password field, and
leaving the server takes you to the server list.

## Reporting a bug

On a server, press Esc and choose **Report a bug**, above Leave server. Describe what happened and what
you were doing, tick the box to agree to sending your logs, and choose **Send report**. The game tells
you whether it was sent, and the team replies on Discord.

The report becomes a private ticket on our Discord. It contains:

- the end of your client log, the launcher's log and the game's `kcd.log`
- your nickname and the server
- your player ids: Steam, Epic, Discord and device id

Use this form for bugs in KCDS itself. Problems with a server's scripts or rules belong with that
server's team. Abusing reports gets a player banned from reporting.

When the game does not reach a server (or an editor stops answering), the folder button next to the gear
in the main menu opens `%LOCALAPPDATA%\KCDS\logs`: your client logs and the launcher's log are all there,
ready to attach to a ticket. The game's own `kcd.log` is in the game's folder.

## Settings for crowded areas

These settings are environment variables. Set them before starting the game, and restart it after a
change. None of them guarantees a player count. Outfits, attachments, NPCs, horses, graphics hardware
and frame time all matter too.

### Character capacity

`KCDS_SKINNING_CAPACITY_SCALE` reserves extra graphics memory for animated characters at startup.

- Accepts whole numbers from 1 to 8. The default is 4, and 1 uses the game's original settings.
- The default adds about 24 MiB of graphics-buffer storage per rendering pipeline.
- An invalid value or an unsupported game binary turns the override off, leaving the game's capacity.
- A higher value does not fix CPU overload, and does not ensure that each character renders correctly.

Server owners can limit how many characters each client receives with the optional [character
budget](server.md#capacity-and-tuning).

### Material capacity

`KCDS_MATERIAL_CAPACITY_SCALE` reserves extra material storage.

- Accepts whole numbers from 1 to 8. The default is 8, and 1 keeps the game's configured capacity.
- The default adds about 2.19 MiB of system memory, plus 2.19 MiB of graphics-buffer storage per
  rendering pipeline, compared with the game's default.
- Larger existing settings are kept. An invalid value or an unsupported game binary turns the override
  off.
- This default is prepared for crowd testing. Correct appearance at 1,000 players has not been verified.

### Clothing colour capacity

`KCDS_MATERIAL_FEATURE_SCALE` reserves extra storage for the colours and patterns of clothing. When
the game's own storage is full, every further piece of clothing is drawn white; with the game's
default this starts at roughly 200 dressed characters in view.

- Accepts whole numbers from 1 to 16. The default is 8, and 1 keeps the game's capacity.
- The default adds about 47 MiB of system memory, plus 47 MiB of graphics-buffer storage per
  rendering pipeline, compared with the game's default.
- 500 players in army outfits used about 30 % of the default.
- An invalid value or an unsupported game binary turns the override off.

### Distance detail (experimental)

With `KCDS_REMOTE_DISTANCE_LOD=1`, the game can use lower detail for idle remote players at least 80
metres from your character.

- Normal detail returns within 60 metres, or when the player moves, fights, rides or interacts.
- It is off by default. Set `0` or remove the variable to turn it off.
- Your existing custom detail settings are kept.
- Its performance benefit and visual quality still need testing in realistic crowded scenes.

## Next steps

- Host a server: [Running a server](server.md)
- Write a resource: [Your first resource](first-resource.md), then [Resources](resources.md) for
  everything a manifest can contain
