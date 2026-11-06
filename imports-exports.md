# Sharing code between resources

A resource can offer values and functions to other resources on the **same side**: server to server,
or client to client. Nothing goes over the network, and live functions and objects cannot cross it.
To reach the other side, use `Events`.

## Offering something

First list the names in your manifest under `kcds.exports`:

```json
{
  "name": "library",
  "kcds": {
    "serverScripts": ["server.js"],
    "exports": ["config", "pay"]
  }
}
```

Then register them:

```js
const { Exports } = kcds.core;

Exports.register('config', { currency: 'groschen' });
Exports.register('pay', (amount) => amount - 1);
```

```ts
const { Exports } = kcds.core;

Exports.register('config', { currency: 'groschen' });
Exports.register('pay', (amount: number) => amount - 1);
```

```lua
local Exports = kcds.core.Exports

Exports.register("config", { currency = "groschen" })
Exports.register("pay", function(amount) return amount - 1 end)
```

```cs
Exports.Register("config", new { currency = "groschen" });
Exports.Register("pay", new Func<double, double>((amount) => amount - 1));
```

`Exports.register` always returns `true`. A name missing from the manifest is not stored, and a
warning goes to the log. Registering a listed name again overwrites its value.

## Using it

The other resource lists yours under `kcds.requires`, which also makes yours start first:

```json
{
  "name": "shop",
  "kcds": {
    "serverScripts": ["server.js"],
    "requires": ["library"]
  }
}
```

If the server may not have the other resource, mark it optional with `{ "name": "library",
"optional": true }`. It starts first when present, and yours starts without it otherwise.

```js
const { Exports, Imports } = kcds.core;

const pay = Exports.get('library', 'pay');
const afterFee = pay(100);

const everything = Imports.get('library'); // { config, pay }
```

```ts
const { Exports, Imports } = kcds.core;

const pay = Exports.get('library', 'pay');
const afterFee = pay(100);

const everything = Imports.get('library'); // { config, pay }
```

```lua
local Exports, Imports = kcds.core.Exports, kcds.core.Imports

local pay = Exports.get("library", "pay")
local afterFee = pay(100)

local everything = Imports.get("library") -- { config, pay }
```

```cs
var pay = Exports.Get("library", "pay");
var afterFee = pay.Invoke(100).AsNumber();

var everything = Imports.Get("library"); // { config, pay }
```

- `Exports.get(resource, name)` returns one value.
- `Imports.get(resource)` returns a new object with all of that resource's current exports, or `{}`
  if it has none.

## Shared, not copied

You receive the same object the other resource holds, so a change made by anyone is seen by everyone.
For that reason, a library should offer functions instead of data others can change.

## When something is wrong

| Situation                                | Result                                                  |
| ---------------------------------------- | ------------------------------------------------------- |
| The name does not exist                  | throws `Export not found`                               |
| The resource is not running              | throws `Resource is not available`                      |
| The resource is not in your `requires`   | a warning in the log, once for each resource you read   |

Listing a link as optional also removes that warning.

## Restarts

Exports are a local list, so reading one sends no request to the server or game. When a resource
stops, its exports disappear. `ensure <name>` restarts the resource and the running resources that
depend on it, and the old values are gone, so read your imports again after a restart.

If you want looser coupling, with a request and a reply, see [Messages](sharing.md).

<!-- only lua cs -->
## Between languages

A resource in JavaScript and one in [Lua](lua.md) share exports, events and messages the same way: the other
resource's language makes no difference. Values cross like this:

- A function is a function you call: a Lua resource calls a JavaScript export with `pay(100)`, and a
  JavaScript resource calls a Lua export as an ordinary function. The same export is the same function each
  time you read it, so you can compare it with `==` or `===`.
- A Lua table arrives in JavaScript as an object, or as an array when its keys are 1 to n. A JavaScript
  object or array arrives in Lua as a value you read with `.` and index from 1.
- `null` and `undefined` arrive in Lua as `nil`. A Lua table has no order of its keys, so do not count on
  the order of an object that came from Lua.
- SDK values, such as a player or a `Vector3`, stay themselves on both sides.
- An error thrown in one language is an error in the other: a Lua `pcall` catches a JavaScript throw, and a
  JavaScript `try` catches a Lua `error`.

A [C#](csharp.md) resource shares the same way. Its export is a delegate, which other resources call as a
function, and another resource's export is a `JsValue` it calls with `Invoke(...)`. Objects and arrays from
JavaScript or Lua are `JsValue`s you read with `["name"]` and index from 0; what C# passes becomes plain
objects and arrays. `null` and `undefined` arrive in C# as `null`. An error in one language is a `JsError` a C#
`catch` takes, and a C# exception is an error JavaScript and Lua catch.
<!-- end -->
