# C#

You can write a resource's server scripts, client scripts or both in C# instead of JavaScript. A C# script
gets the same SDK a JavaScript script gets, as typed .NET classes: every type, member and event, with the same
rules for starting, stopping and errors. You build your scripts with the .NET 9 SDK into `.dll` files, and
KCDS runs them on .NET compiled to WebAssembly, on the server and in each player's game.

## A C# resource

`KCDS-Types-<version>.zip` on the download page has a resource to start from in `csharp/template`, with the
SDK next to it:

```text
csharp/
  Kcds.Sdk.dll          the SDK your scripts are built against
  Kcds.Sdk.xml          its documentation, for your editor
  template/
    Directory.Build.props
    package.json
    server/
      Server.csproj
      Main.cs
    client/
      Client.csproj
      Main.cs
```

Copy `template` into your server's `resources` folder under your resource's name, such as
`resources/hello-cs`, and keep `Kcds.Sdk.dll` in the folder above it, or give its path to the build with
`-p:KcdsSdk=<path to Kcds.Sdk.dll>`. Then build each side:

```text
cd resources/hello-cs/server
dotnet build -c Release
cd ../client
dotnet build -c Release
```

The scripts land in `bin/server/Server.dll` and `bin/client/Client.dll`, which `package.json` lists:

```json
{
  "name": "hello-cs",
  "version": "1.0.0",
  "kcds": {
    "server": "bin/server/Server.dll",
    "client": "bin/client/Client.dll"
  }
}
```

A side's language comes from its scripts' file extensions: `.dll` is C#, `.lua` Lua, `.js`, `.cjs` and
`.mjs` JavaScript. All scripts of one side are in one language, so a resource can have a C# server and a
JavaScript client. A C# side lists the assemblies it was built into, never its `.cs` sources; the server
refuses a resource that lists sources.

Players get the files of the client scripts' folder (see [Client resources](client-resources.md)), so the
template builds each side into a folder of its own: `bin/client/` holds only the client's assembly, and
your server code and sources never reach a player's PC. Keep it so if you change the layout.

`server/Main.cs`:

```cs
public class Main : Script
{
    public override void OnStart()
    {
        // Runs when a player has joined and loaded the world.
        Events.On("playerReady", (Player player) =>
        {
            Chat.SendToPlayer(player, "Welcome, " + player.Nickname + "!", new { author = "Server" });
        });

        // Runs for "/hello" or "/hello some words" in the chat.
        Events.On("playerCommand", (Player player, string command, string[] args) =>
        {
            if (command.ToLowerInvariant() != "hello")
            {
                return;
            }
            var reply = args.Length > 0 ? string.Join(" ", args) : "Hello!";
            Chat.SendToPlayer(player, reply, new { author = "Server", color = Color.FromRGB(255, 210, 77) });
        });

        // A client asks how many players are online.
        Events.FromClient("hello:count", (Player player) =>
        {
            player.Emit("hello:count", new { players = Player.All().Length });
        });
    }
}
```

`client/Main.cs`:

```cs
public class Main : Script
{
    public override void OnStart()
    {
        Events.On("playerReady", () => Hud.ShowNotification("The hello resource is running."));

        // F7 asks the server how many players are online.
        Key.Bind("f7", new Action(() => Events.ToServer("hello:count")));

        Events.On("hello:count", (JsValue data) =>
        {
            Hud.ShowNotification(data["players"] + " players online");
        });
    }
}
```

`ensure hello-cs` in the server console starts it, as for any resource. After a change, build again and
`restart hello-cs`.

## Scripts

Every public class of the resource's assemblies that derives from `Script` is a script: when the resource
starts, KCDS makes one of each and calls its `OnStart`. What a script registers there (handlers, timers,
exports) belongs to the resource and ends when it stops. A resource may have as many script classes as it
likes, in one assembly or several.

Each resource's assemblies load on their own, so its classes and their static fields are its own: two
resources may both have a `Main` class, and a restarted resource starts with fresh statics. The SDK
(`Kcds.Sdk.dll`) is the host's, so a resource never ships it.

## Editor support

`Kcds.Sdk.xml` is the SDK's documentation: with it next to `Kcds.Sdk.dll`, Visual Studio, Rider and VS Code
show every member's description and parameters. The template's `Directory.Build.props` references the SDK
and imports its namespaces for every file:

| Namespace         | What it has                                                        |
| ----------------- | ------------------------------------------------------------------ |
| `Kcds`            | `Script`, `Log`, `JsValue`, `JsError`, `Timers`, `Network`, `Sdk`  |
| `Kcds.Core`       | `Events`, `Chat`, `Exports`, `Imports`, `Messages`, `Runtime`, ... |
| `Kcds.Players`    | `Player`, `Entity`, `Inventory`, `Items`, `LocalPlayer`, ...       |
| `Kcds.Characters` | `Horse`, `Dog`, NPCs and their kin                                 |
| `Kcds.World`      | `World`, `Area`, `Dice`, `CookPot`, `Gathering`, `Crafting`, ...   |
| `Kcds.Ui`         | `Hud`, `Key`, `Controls`, `ActionHint`, `Book`, `NativeUI`, ...    |
| `Kcds.Math`       | `Vector3`, `Quaternion`, `Color`, ...                              |

The guide's C# examples are written with these namespaces imported, as the template has them. Use the
SDK version matching your server.

## What is different from JavaScript

The API is the same; only the language's own ways differ.

| JavaScript                                   | C#                                            |
| -------------------------------------------- | --------------------------------------------- |
| `kcds.core.Events.on('playerReady', fn)`     | `Events.On("playerReady", fn)`                |
| `player.nickname`, `player.emit(...)`        | `player.Nickname`, `player.Emit(...)`         |
| `new kcds.math.Vector3(1, 2, 3)`             | `new Vector3(1, 2, 3)`                        |
| `list[0]`, `list.length`                     | `list[0]`, `list.Length`                      |
| `{ author: 'Server' }`                       | `new { author = "Server" }`                   |
| `(player, command) => { ... }`               | `(Player player, string command) => { ... }`  |
| `try { ... } catch (error) { ... }`          | `try { ... } catch (JsError problem) { ... }` |
| `console.log(text)`                          | `Log.Info(text)`                              |
| `setTimeout(fn, 500)`                        | `Timers.SetTimeout(fn, 500)`                  |
| `await new Promise((r) => setTimeout(r, 5))` | `await Timers.Delay(5)`                       |

- **Names** are the JavaScript names with a capital first letter: `Player.GetById(7)`, `vector.Length`,
  `Hud.ShowNotification(...)`. Event names stay as they are (`"playerReady"`), and `EventNames` has them as
  constants.
- **Handlers** are lambdas with the event's arguments as typed parameters, such as
  `(Player player, string command, string[] args)`. A parameter you leave out is not passed; one typed
  `JsValue` takes any value. A handler that answers a value answers it to JavaScript: `false` refuses where
  an event can be refused, `null` leaves it. A handler that answers a `Task` (an `async` lambda) is waited
  for, as an async JavaScript handler is. Where a member takes a function as a plain value (such as
  `Key.Bind`'s second argument), give the lambda its type: `new Action(() => ...)`.
- **Values without a type** (an event's payload, the answer of `Inventory.Get`, options) are `JsValue`s:
  `payload["text"]`, `list[0]`, `list.Length`, `value.Keys`, `foreach (var row in list)`, and
  `value.AsString()`, `AsNumber()`, `AsInt()`, `AsBool()` or `As<T>()` to convert. `value.IsNull` says it
  is `null` or `undefined`, `value.Type` is JavaScript's `typeof`, `value.ToJson()` its JSON. A `JsValue`
  that is a function is called with `Invoke(...)`.
- **Objects** you pass (anonymous objects, records, dictionaries, lists and arrays) become JavaScript objects
  and arrays. Property names start lower-case in JavaScript: `new { Author = "Server" }` is
  `{ author: "Server" }`. A property left `null` is not given.
- **Nothing**: a member that answers `null` or `undefined` in JavaScript answers `null`. A `null` you pass
  for an optional parameter is an argument not given, so the member's default applies:
  `Horse.Spawn(position, null, null, "Pebbles")`. Where a member asks for `null` itself, such as
  `state.OnChange(Sdk.Null, handler)` for every key, pass `Sdk.Null`.
- **Text**: C# writes `true` and `false` as `True` and `False`; a number as the shortest text for it,
  with `.` as the decimal point.
- **Errors**: an SDK error is a `JsError` with the JavaScript error's `Message` and `Name` (`TypeError`,
  `RangeError`, ...), and the error itself as `Error`. An exception in a handler is that handler's error, as
  a JavaScript throw is, and an exception in `OnStart` stops the resource (or restarts it, as its
  `errorBehavior` says).

```cs
var a = new Vector3(3, 4, 0);
Log.Info(a.Length + " " + a.Distance(Vector3.Zero()));

try
{
    Chat.SendToAll(5);
}
catch (JsError problem)
{
    Log.Info(problem.Name + ": " + problem.Message); // TypeError: Chat text must be string
}

foreach (var player in Player.All())
{
    Log.Info(player.Id + " " + player.Nickname);
}
```

## Tasks

A member that answers a promise in JavaScript answers a `Task` in C#, which you `await`. A rejected promise
throws its `JsError` at the `await`:

```cs
public class Status : Script
{
    public override void OnStart()
    {
        Events.On("status:check", async () =>
        {
            try
            {
                var response = await Network.Fetch("https://example.com/status.json");
                var status = await Network.Json(response);
                Log.Info("players online " + status["players"]);
            }
            catch (JsError problem)
            {
                Log.Info("request failed " + problem.Message);
            }
        });
    }
}
```

What follows an `await` runs a moment later than in JavaScript: .NET continues a task from its own queue,
after JavaScript's promise callbacks. A handler that answers a `Task` is still waited for.

## Timers, logging and the web

`Timers.SetTimeout`, `SetInterval`, `ClearTimeout` and `ClearInterval` are the [timers](timers.md) of
JavaScript, and `Timers.Delay(ms)` a task that completes after a while. Stopping a resource cancels them.
`Log.Info`, `Log.Warn` and `Log.Error` write to the resource's log, and so does `Console.WriteLine`.

Server scripts reach the web with `Network.Fetch(url, options)`, `Network.Json(response)` and
`Network.Text(response)`, the same `fetch` JavaScript has
([Network access](resources.md#network-access-from-server-scripts)).

```cs
var heartbeat = Timers.SetInterval(() => Log.Info("still running"), 5000);
Timers.ClearInterval(heartbeat);
await Timers.Delay(100);
```

## Between languages

A C# resource shares with resources in JavaScript and Lua as they share with each other
([Imports and exports](imports-exports.md)): the same events, exports and messages. An export is a delegate:

```cs
public class Ledger : Script
{
    private readonly Dictionary<double, double> balances = new();

    public override void OnStart()
    {
        Exports.Register("balance", new Func<double, double>((playerId) => balances.GetValueOrDefault(playerId)));
        Exports.Register("pay", new Func<double, double, bool>((playerId, amount) =>
        {
            var after = balances.GetValueOrDefault(playerId) + amount;
            if (after < 0)
            {
                return false;
            }
            balances[playerId] = after;
            return true;
        }));
    }
}
```

and another resource's export is a `JsValue` you call with `Invoke`:

```cs
var paid = Exports.Get("ledger", "pay").Invoke(7, -500).AsBool();
```

## What C# scripts cannot do

- Files, processes, threads and the network outside `Network.Fetch` are out of reach: .NET runs inside the
  host's WebAssembly, with one thread and a file system that only lives in memory. For data that must last,
  call a JavaScript resource's [exports](imports-exports.md) that save it, or a web service.
- The C# resources of one side share one .NET runtime. A resource's types and statics are its own, but what
  is global to .NET itself (`AppContext`, environment variables, the default culture) is shared, and a stack
  overflow or `Environment.Exit` ends every C# resource of that side until the server or the game starts
  again.
- There is no time limit on a call into C#: an endless loop holds up every script of its side. Keep handlers
  short and use `await` or timers for waiting.
- A restarted resource's old assemblies stay in memory (a few tens of kilobytes each time) until the server or
  the game starts again.
