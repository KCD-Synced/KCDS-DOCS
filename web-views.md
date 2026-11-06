# Web views

A web view draws an HTML page over the game, which suits menus, shops, phones and scoreboards. It can
load a page shipped with your resource, at `kcds://resources/<resource>/<path>`, or an `https://`
address.

<!-- only lua cs -->
The page itself is HTML, CSS and JavaScript in every resource, whichever language its scripts are
in: a Lua or C# client script opens the same page and exchanges the same messages with it as a
JavaScript one, with the same calls.
<!-- end -->

## A simple example

List the page's folder under `files` so that players download it:

```json
{
  "name": "shop_menu",
  "kcds": {
    "clientScripts": ["client.js"],
    "files": ["ui/**"]
  }
}
```

The client script opens the page and exchanges messages with it:

```js
// client
const { Web, Controls } = kcds.ui;

const shop = Web.open('kcds://resources/shop_menu/ui/index.html', {
  width: 800,
  height: 600,
  focus: true,
});
Controls.showCursor();

Web.on(shop, 'buy', (item) => kcds.core.Events.toServer('menuBuy', item));
Web.emit(shop, 'gold', { amount: 120 });
```

```lua
-- client
local Web, Controls = kcds.ui.Web, kcds.ui.Controls

local shop = Web.open("kcds://resources/shop_menu/ui/index.html", {
  width = 800,
  height = 600,
  focus = true,
})
Controls.showCursor()

Web.on(shop, "buy", function(item) kcds.core.Events.toServer("menuBuy", item) end)
Web.emit(shop, "gold", { amount = 120 })
```

```cs
// client
var shop = Web.Open("kcds://resources/shop_menu/ui/index.html", new
{
    width = 800,
    height = 600,
    focus = true,
});
Controls.ShowCursor();

Web.On(shop, "buy", (JsValue item) => Events.ToServer("menuBuy", item));
Web.Emit(shop, "gold", new { amount = 120 });
```

In the page, `window.sendToGame(name, json)` sends a message. What the script sends with `Web.emit`
arrives as a window event, with the data in `detail`:

```html
<!-- ui/index.html -->
<button id="buy">Buy a sword</button>
<script>
  window.addEventListener('gold', (event) => console.log(event.detail.amount));
  document.getElementById('buy').onclick = () => window.sendToGame('buy', JSON.stringify('sword'));
</script>
```

## Controlling a view

- `show`, `hide`, `focus`, `resize`, `move`, `navigate` and `close` do what their names say.
- `setBackground` keeps a view running without drawing it.
- `setViewWorldPosition(view, point)` pins a view to a world position, such as a sign over a stall.
- A resource can reach only its own views.

**Size:** a view without `width` and `height` fills the screen, as does a view at 0, 0 with the size
from `Web.screenSize()`. Both follow the game's resolution when the player changes it, so a responsive
page lays itself out again. A smaller view keeps its size, so read `Web.screenSize()` again to place it.

## Keyboard and mouse

While a view has focus, the player types into it and key binds pause. The third-person camera also stops
turning with the mouse, as it does during `Controls.showCursor()`, `Controls.disable()` or
`Controls.blockAction('rotateyaw')`.

The page receives normal browser key events, with `key`, `code` (the physical key, such as `KeyW`,
`Digit1` or `Enter`), `keyCode` and the modifiers Shift, Ctrl and Alt. So `event.code` checks work as in
any browser.

## When a page is slow

If a focused page has not finished loading after 6 seconds, the player gets the mouse and keyboard back,
and `webViewFailed` fires with `ERR_TIMED_OUT` (`errorCode` -7). Your script can then try `navigate`
again or close the view. If the page finishes after all, it takes input again.

`Web.isBackground(viewId)` says whether a view runs without being drawn.

Reference: [Web](/docs/api/web), [LoadingScreen](/docs/api/loadingscreen).

## A loading screen for your server

By default, players see the game's loading screen. To show your own, name a page in `package.json`. It
appears as soon as its own files have arrived, while the rest of your resources download.

```json
{
  "name": "my-server",
  "kcds": {
    "clientScripts": ["client.js"],
    "loadingScreen": { "page": "ui/loading.html", "files": ["ui/loading/**"] }
  }
}
```

`loadingScreen` is either the page's path or `{ page, files }`, with patterns for the pictures, videos
and fonts it needs. These download first and need no entry in the main `files` list.

**Limits:** up to 32 MB together with the page, and 32 MB per file (a page opened later with `Web.open`
can load files up to 512 MB). Above that, the server logs an error
and players get the game's screen. Nothing shows until the player has all of it, so keep it small. If
several resources name a page, the first in the catalogue is used.

A client script can also show a page once resources have started, which replaces the one named in
`package.json`:

```js
// client: needs "files": ["ui/**"] in package.json
kcds.ui.LoadingScreen.show('kcds://resources/my-server/ui/loading.html');
```

```lua
-- client: needs "files": ["ui/**"] in package.json
kcds.ui.LoadingScreen.show("kcds://resources/my-server/ui/loading.html")
```

```cs
// client: needs "files": ["ui/**"] in package.json
LoadingScreen.Show("kcds://resources/my-server/ui/loading.html");
```

Pictures and video are ordinary HTML. Video must be WebM (VP8 or VP9), and it can be muted or have sound:

```html
<body style="margin: 0; background: #000">
  <video src="loading/intro.webm" autoplay muted loop style="width: 100%"></video>
  <img src="loading/logo.png" style="position: fixed; left: 40px; top: 40px" />
</body>
```

### Showing progress

The page receives `kcdsLoading` window events:

```html
<!-- ui/loading.html -->
<div id="fill"></div>
<script>
  window.addEventListener('kcdsLoading', (event) => {
    const { stage, percent } = event.detail;
    document.getElementById('fill').style.width = `${percent}%`;
  });
</script>
```

| `stage`       | What `percent` shows                                                                            |
| ------------- | ----------------------------------------------------------------------------------------------- |
| `'download'`  | share of bytes; the event also has `resourcesDone`, `resourcesTotal`, `bytesDone`, `bytesTotal` |
| `'resources'` | scripts can run from here on                                                                    |
| `'level'`     | counts the game's load sections, staying at 95 at most until the world is ready, then 100       |
| `'ready'`     | the page closes                                                                                 |

The percentage at `'level'` is an estimate and does not move evenly, because the game has no single
number for its whole load.

Client scripts hear these stages as the event `loadingProgress(stage, percent)`.

### How the page behaves

- It is a full-screen web view with no keyboard or mouse.
- While it shows, the server browser's connecting window is gone, so the player cannot cancel. If the
  join fails, the page goes and the window returns with the reason.
- `LoadingScreen.hold()` keeps the page up after `'ready'` until `LoadingScreen.release()`, for at most
  30 seconds, so a broken script cannot trap a player.
- If the world is ready before a long download finishes, the page stays until the download ends, for at
  most 10 minutes.
