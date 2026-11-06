# Custom perks

A resource can add perks of its own to the game's perk tree: a name, a level, the skill or stat it belongs to, an icon
and what the perk does. Players see them on the perk screen like the game's own, and your scripts give, take and check
them with the same members as the game's perks.

The game reads its perk tables once, when it starts, and every learned perk points into them, so a new perk cannot be
added while a player's game runs. A resource therefore ships its perks as the game's own table patches, and they reach
a player's game at its next start.

## The files

```text
my-perks/
  package.json
  server.js
  replace/Libs/Tables/rpg/perk__my_server.xml        new rows of the perk table
  replace/Libs/Tables/rpg/perk_buff__my_server.xml   what each perk does
  replace/Libs/UI/Textures/Icons/Perks/my_perk_0_icon.dds   its icon (also _1_ and _2_, and
                                                         Icons/Buffs/my_perk_icon.dds)
```

## The perk table

`perk__my_server.xml` holds rows in the game's own form, with a perk id of your own (a GUID nobody else uses):

```xml
<?xml version="1.0" encoding="us-ascii"?>
<database xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="barbora" xsi:noNamespaceSchemaLocation="../database.xsd">
	<perks version="1">
		<perk autolearnable="false" icon_id="perk_onslaught" level="1" perk_id="6b1f3c2e-951a-4d7e-9a51-c0de00000951" perk_name="MyServerPerk" perk_ui_name="perk_onslaught_name" perk_ui_desc="perk_onslaught_desc" skill_selector="17" visibility="2" />
	</perks>
</database>
```

| Attribute        | Meaning                                                                       |
| ---------------- | ----------------------------------------------------------------------------- |
| `perk_id`        | the perk's GUID                                                               |
| `perk_name`      | the name your scripts use (`player.addPerk('MyServerPerk')`)                  |
| `skill_selector` | the skill or stat whose tree the perk sits in                                 |
| `level`          | the level of that skill or stat the perk needs                                |
| `perk_ui_name`, `perk_ui_desc` | the localization keys of its name and description on the perk screen |
| `icon_id`        | its icon (see below)                                                          |
| `visibility`     | `2` shows it on the perk screen                                               |
| `autolearnable`  | `true` gives it to every player who reaches the level                         |

To give the perk a name of its own without shipping localization files, use `Progression.setPerkText` (see
[Skills, perks and needs](progression.md)).

## What the perk does

A perk does something only through one of the game's perk effect tables, and the game gives a player no perk that has
none: `player.addPerk` then changes nothing (the player's game logs `PROGRESSION perk add refused by the game`). The
simplest is a buff the perk holds while learned, `perk_buff__my_server.xml` (a buff of the game's, or a
[custom buff](custom-buffs.md)):

```xml
<?xml version="1.0" encoding="us-ascii"?>
<database xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="barbora" xsi:noNamespaceSchemaLocation="../database.xsd">
	<perk_buffs version="1">
		<perk_buff buff_id="44d83d12-9709-4e54-9b7d-9b6bcfb630be" perk_id="6b1f3c2e-951a-4d7e-9a51-c0de00000951" />
	</perk_buffs>
</database>
```

## How players get it

When a player joins, their game keeps every `perk*__*.xml` it receives, for that server, in
`%LOCALAPPDATA%\KCDS\startup-tables\`. If these are not the patches the game started with, the player sees "This
server has perks of its own. Restart the game to have them." After the restart the game loads them with its own
tables, as it loads its own `perk__kcd2.xml`. A game started for a server (a join link, `--connect`) loads that
server's perks; any other start loads those of the last server joined. A file outside `Libs/Tables/rpg/perk*__*.xml`
takes effect at once, as any `replace/` file does.

## Its icon

Give the row `icon_id="my_perk"` and ship the pictures (64 x 64 `.dds`, as the game's) in `replace/` at the game's
icon paths: `Libs/UI/Textures/Icons/Perks/my_perk_0_icon.dds`, `_1_icon.dds` and `_2_icon.dds` (the perk's three
looks on the perk screen) and `Libs/UI/Textures/Icons/Buffs/my_perk_icon.dds`. They arrive when the player joins, with
no restart. A `stream/` folder cannot hold them: streamed files go under `<folder>/kcds/<resource>/` only.

## Using it in scripts

The rows of a resource's `perk__<name>.xml` join the server's catalog when the resource starts, and the server log
says `perks MyServerPerk added from perk__my_server.xml`. `Progression.perks()`, `Progression.perkNamed`,
`player.addPerk`, `player.removePerk`, `player.hasPerk` and `player.perks` then know it like the game's own perks:

```js
const { Progression } = kcds.players;

const perk = Progression.perkNamed('MyServerPerk');
kcds.core.Events.on('playerReady', (player) => {
  if (perk && !player.hasPerk(perk.id)) {
    player.addPerk(perk.id);
  }
});
```

```lua
local Progression = kcds.players.Progression

local perk = Progression.perkNamed("MyServerPerk")
kcds.core.Events.on("playerReady", function(player)
  if perk and not player:hasPerk(perk.id) then
    player:addPerk(perk.id)
  end
end)
```

```cs
var perk = Progression.PerkNamed("MyServerPerk");
Events.On("playerReady", (Player player) =>
{
    if (!perk.IsNull && !player.HasPerk(perk["id"]))
    {
        player.AddPerk(perk["id"]);
    }
});
```

A client script sees another player's perk of your own by its GUID (its `name` is the GUID, `track` null).

Reference: [Progression](/docs/api/progression), [Player](/docs/api/player).
