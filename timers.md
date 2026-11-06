# Timers, logging and the runtime

Scripts on both sides have the usual JavaScript scheduling functions: `setTimeout`, `setInterval`,
`clearTimeout`, `clearInterval` and `queueMicrotask`. Extra arguments after the delay are passed to
your callback.

```js
const greeting = setTimeout((text) => console.info(text), 250, 'Hello after 250 ms');
clearTimeout(greeting);

const heartbeat = setInterval(() => console.debug('still running'), 5000);
clearInterval(heartbeat);

queueMicrotask(() => console.log('runs when the current task ends'));
```

```ts
const greeting = setTimeout((text: string) => console.info(text), 250, 'Hello after 250 ms');
clearTimeout(greeting);

const heartbeat = setInterval(() => console.debug('still running'), 5000);
clearInterval(heartbeat);

queueMicrotask(() => console.log('runs when the current task ends'));
```

```lua
local greeting = setTimeout(function(text) print(text) end, 250, "Hello after 250 ms")
clearTimeout(greeting)

local heartbeat = setInterval(function() print("still running") end, 5000)
clearInterval(heartbeat)

queueMicrotask(function() print("runs when the current task ends") end)
```

```cs
const string text = "Hello after 250 ms";
var greeting = Timers.SetTimeout(() => Log.Info(text), 250);
Timers.ClearTimeout(greeting);

var heartbeat = Timers.SetInterval(() => Log.Info("still running"), 5000);
Timers.ClearInterval(heartbeat);

// C# continues after an await instead of queueing a microtask.
await Task.Yield();
Log.Info("runs when the current task ends");
```

Stopping or restarting a resource cancels the timers it set, so a pending callback never reaches the
new copy.

## What a timer handle is

Keep the value a timer call returns and pass it to the matching clear call. Its type depends on the
side:

| Side   | Handle                                                                     |
| ------ | -------------------------------------------------------------------------- |
| Client | a number                                                                   |
| Server | a Node `Timeout` object, with `ref()`, `unref()`, `hasRef()`, `refresh()` |

On the server, `ref` and `unref` decide whether the timer keeps the process alive. Calling
`refresh()` on a handle of a stopped resource does not bring its callback back.

## Logging

`console.log`, `console.info`, `console.warn`, `console.error` and `console.debug` write to the
resource's log.

## Which side and version am I on?

`kcds.core.Runtime` is read-only:

| Property        | Meaning                                  |
| --------------- | ---------------------------------------- |
| `isClient`      | true in a client script                  |
| `isServer`      | true in a server script                  |
| `apiVersion`    | the version of the script API            |
| `serverVersion` | the version configured for this server   |

## Console commands of your own

A server script hears what is typed into the server console, except the commands the host handles
itself, such as those that start and stop resources. The first word arrives as `command`, the other
words as `args`:

```js
kcds.core.Events.on('consoleCommand', (command, args) => {
  if (command === 'greet') {
    console.info(`Hello, ${args.join(' ')}`);
  }
});
```

```ts
kcds.core.Events.on('consoleCommand', (command, args) => {
  if (command === 'greet') {
    console.info(`Hello, ${args.join(' ')}`);
  }
});
```

```lua
kcds.core.Events.on("consoleCommand", function(command, args)
  if command == "greet" then
    print("Hello, " .. table.concat(args, " "))
  end
end)
```

```cs
Events.On("consoleCommand", (string command, string[] args) =>
{
    if (command == "greet")
    {
        Log.Info($"Hello, {string.Join(" ", args)}");
    }
});
```
