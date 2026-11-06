# Discord presence

KCDS publishes the player's current activity to their Discord profile through the Discord app on
their PC, with no work on your side. While a player is on your server, your client scripts can add
your server's name, a status line, images and one button.

## The default presence

| Player is                        | Discord shows            |
| -------------------------------- | ------------------------ |
| starting the game                | Starting up              |
| in the main menu or server list  | In the main menu         |
| joining a server                 | Joining _server name_    |
| on a server                      | Playing on _server name_ |

The presence also has the KCDS logo, the client version, a timer and a **KCDS website** button. On a
server, KCDS adds the player count, such as "Players online (5 of 32)", after your status line if you
set one. Scripts cannot change that count. A server joined by address appears as "a KCDS server", so
an IP address is never displayed.

## Adding your server's details

```js
// client
kcds.ui.Discord.setPresence({
  name: 'Rattay Roleplay', // shown as "KCDS - Rattay Roleplay"
  largeImage: 'https://example.com/rattay-logo.png', // replaces the KCDS logo
  largeText: 'Rattay Roleplay', // tooltip for that image
  button: { label: 'Join our Discord', url: 'https://discord.gg/example' },
});
```

```lua
-- client
kcds.ui.Discord.setPresence({
  name = "Rattay Roleplay", -- shown as "KCDS - Rattay Roleplay"
  largeImage = "https://example.com/rattay-logo.png", -- replaces the KCDS logo
  largeText = "Rattay Roleplay", -- tooltip for that image
  button = { label = "Join our Discord", url = "https://discord.gg/example" },
})
```

```cs
// client
Discord.SetPresence(new
{
    name = "Rattay Roleplay", // shown as "KCDS - Rattay Roleplay"
    largeImage = "https://example.com/rattay-logo.png", // replaces the KCDS logo
    largeText = "Rattay Roleplay", // tooltip for that image
    button = new { label = "Join our Discord", url = "https://discord.gg/example" },
});
```

You can also update it as things change for the player:

```js
// client
kcds.core.Events.on('job:changed', (job) => {
  kcds.ui.Discord.setPresence({
    state: `Working as a ${job}`, // the line under "Playing on ..."
    smallImage: 'https://example.com/icons/blacksmith.png',
    smallText: 'Blacksmith',
  });
});
```

```lua
-- client
kcds.core.Events.on("job:changed", function(job)
  kcds.ui.Discord.setPresence({
    state = "Working as a " .. job, -- the line under "Playing on ..."
    smallImage = "https://example.com/icons/blacksmith.png",
    smallText = "Blacksmith",
  })
end)
```

```cs
// client
Events.On("job:changed", (string job) =>
{
    Discord.SetPresence(new
    {
        state = $"Working as a {job}", // the line under "Playing on ..."
        smallImage = "https://example.com/icons/blacksmith.png",
        smallText = "Blacksmith",
    });
});
```

## What you can set

| Field                       | Limit                                     |
| --------------------------- | ----------------------------------------- |
| `name`, `state`             | up to 128 bytes                           |
| `largeImage`, `largeText`   | up to 128 bytes each                      |
| `smallImage`, `smallText`   | up to 128 bytes each                      |
| `button`                    | one; label of 1 to 32 bytes, `https` link |

These fields show only while the player is on a server and disappear when they leave. Other Discord
activity fields, such as its type, timestamps, party and secrets, are reserved for KCDS, and
`setPresence` refuses them. Resources on the same machine share one presence.

The app's name, **KCDS**, stays as the title, and your `name` reads "KCDS - _name_" on the line
below. The second button is always **KCDS website**, and a script cannot change or remove it.

## Calls

- `setPresence(fields)` sets the fields and publishes them straight away.
- `setState`, `setName`, `setLargeImage`, `setLargeText`, `setSmallImage`, `setSmallText`,
  `setAssets` and `setButton(label, url)` only prepare a field. `update()` then publishes everything
  prepared.
- `reset()` drops prepared fields without publishing. `clear()` removes your published fields, leaving
  KCDS's own presence.
- `isAvailable()` tells whether the Discord app is running and connected. While it is not, `update()`
  and `setPresence()` return `false`.
- `getUserId()` gives the Discord account signed in to the player's app, or `''` until Discord has
  answered.

> **Do not trust `getUserId()` alone.** The value comes from the player's game, so confirm it with your
> own Discord login before relying on it.

## Checking your buttons

Discord only shows an activity's buttons to other people. To see your button, view the profile from a
second account.

Reference: [Discord](/docs/api/discord).
