# Join queue and whitelist

When a server has reached `maxPlayers` ([server.json](server.md#settings-serverjson)), new players wait
in a queue instead of being refused. Their server browser shows a window with their place, how long
they have waited, any status text you set, and a **LEAVE QUEUE** button. When a place opens, the first
player in line continues joining as normal.

The queue needs no setup and works first come, first served. Your scripts can change the order, show
their own text or turn players away.

## Working with the queue

```js
const { Events, Queue } = kcds.core;
const supporters = new Set(['76561198000000000']);

Events.on('playerQueued', (entry) => {
  if (supporters.has(entry.steamId)) {
    Queue.setPriority(entry.id, 10);
    Queue.setStatus(entry.id, {
      line: 'Welcome back! Supporters go first.',
      fields: [{ label: 'Priority', value: 'Supporter' }],
    });
  }
});

Events.on('playerUnqueued', (entry, reason) => {
  console.info(`${entry.nickname} left the queue: ${reason}`);
});
```

```lua
local Events, Queue = kcds.core.Events, kcds.core.Queue
local supporters = { ["76561198000000000"] = true }

Events.on("playerQueued", function(entry)
  if supporters[entry.steamId] then
    Queue.setPriority(entry.id, 10)
    Queue.setStatus(entry.id, {
      line = "Welcome back! Supporters go first.",
      fields = { { label = "Priority", value = "Supporter" } },
    })
  end
end)

Events.on("playerUnqueued", function(entry, reason)
  print(entry.nickname .. " left the queue: " .. reason)
end)
```

```cs
var supporters = new HashSet<string> { "76561198000000000" };

Events.On("playerQueued", (JsValue entry) =>
{
    if (supporters.Contains(entry["steamId"].AsString()!))
    {
        Queue.SetPriority(entry["id"], 10);
        Queue.SetStatus(entry["id"], new
        {
            line = "Welcome back! Supporters go first.",
            fields = new[] { new { label = "Priority", value = "Supporter" } },
        });
    }
});

Events.On("playerUnqueued", (JsValue entry, string reason) =>
{
    Log.Info($"{entry["nickname"]} left the queue: {reason}");
});
```

An entry looks like `{ id, nickname, steamId, ip, waitedSeconds, priority }`. The `reason` in
`playerUnqueued` is `'admitted'`, `'left'` or `'rejected'`.

| Call                              | Effect                                                                     |
| --------------------------------- | -------------------------------------------------------------------------- |
| `Queue.list()`                    | all waiting entries, next player first                                     |
| `Queue.setPriority(id, priority)` | a whole number where higher goes first; ties go by arrival; everyone starts at 0 |
| `Queue.setStatus(id, { line, fields })` | sets the window's text; without it the window shows place and time   |
| `Queue.reject(id, reason)`        | turns the player away and shows them the reason (up to 256 bytes)          |

For `setStatus`, `line` can be up to 256 bytes. `fields` holds up to 3 `{ label, value }` pairs, of up
to 32 and 96 bytes.

Keep in mind:

- `id` belongs to the queue entry and is not a player id, because a queued player has no `Player` object
  yet.
- `nickname` and `steamId` in the queue are what the player's game reports, and are verified only at
  the next step.
- A player whose game stops answering for 15 seconds is removed from the line with the reason `'left'`.

## Checking players as they join

Everyone who gets in, from the queue or directly, passes through `playerConnecting`. Here
`request.steamId` is verified: it is the Steam account on the player's licence (see [A legal copy of the
game](licence.md)).

```js
kcds.core.Events.on('playerConnecting', async (request) => {
  request.update('One moment, looking you up...');
  const account = await lookUpAccount(request.steamId);
  if (!account) {
    request.reject('Register on our website first: example.com/join');
  }
});
```

```lua
-- The handler returns the promise, so the join waits for it as for an async JavaScript handler.
kcds.core.Events.on("playerConnecting", function(request)
  request:update("One moment, looking you up...")
  return lookUpAccount(request.steamId):next(function(account)
    if not account then
      request:reject("Register on our website first: example.com/join")
    end
  end)
end)
```

```cs
// The handler answers a Task, so the join waits for it.
Events.On("playerConnecting", async (JsValue request) =>
{
    request.Call("update", "One moment, looking you up...");
    var account = await lookUpAccount(request["steamId"]);
    if (account.IsNull)
    {
        request.Call("reject", "Register on our website first: example.com/join");
    }
});
```

- `request.update(message)` shows a line of up to 256 bytes on the waiting screen and restarts the wait's
  timeout.
- `request.reject(reason)` turns the player away.
- A handler that throws or rejects also turns the player away.

The [Events](/docs/api/events) reference lists everything `request` has.

## Whitelist

To admit only listed players, put a `whitelist.json` in the server's resources folder:

```json
{
  "enabled": true,
  "nicknames": ["Henry", "Hans"],
  "steamIds": ["76561198000000000"],
  "reason": "Members only: apply on our Discord."
}
```

| Key         | Meaning                                                                                   |
| ----------- | ----------------------------------------------------------------------------------------- |
| `enabled`   | `true` turns it on; with `false` or with no file, anyone can join                         |
| `steamIds`  | allowed 64-bit Steam ids, written as text                                                 |
| `nicknames` | allowed nicknames, ignoring case; players choose their own nickname, so prefer Steam ids  |
| `reason`    | what others are told; the default is "This server only admits listed players."            |

The file is read when the server starts. Invalid JSON is reported in the log, and the server then runs
with no whitelist.

The whitelist is checked in the queue, and again before your `playerConnecting` handlers, against the
verified Steam id. The server's [password](server.md#password) is checked before the whitelist.
