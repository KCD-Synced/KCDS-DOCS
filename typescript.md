# TypeScript

KCDS runs CommonJS JavaScript, so you compile TypeScript to `.js` before the server starts your resource. Each
release includes declarations for the whole SDK, which give your editor every type, member and side.

## Getting the declarations

Install the npm package `@kcds/types` in the version your server runs:

```text
npm install --save-dev @kcds/types@0.4.0
```

A KCDS hotfix x.y.z.w is the package version x.y.z-hotfix.w, for example `@kcds/types@0.4.0-hotfix.1` for
0.4.0.1.

| File              | Declares                                                                  |
| ----------------- | ------------------------------------------------------------------------- |
| `server.d.ts`     | a server project's entry: `shared.d.ts` and `server-api.d.ts`             |
| `client.d.ts`     | a client project's entry: `shared.d.ts` and `client-api.d.ts`             |
| `shared.d.ts`     | what both sides have: `console`, `require` and the shared named types     |
| `server-api.d.ts` | server scripts: the `kcds` object with the server's types and events      |
| `client-api.d.ts` | client scripts: the `kcds` object with the client's types and events      |

The same three declaration files are also in `KCDS-Types-<version>.zip` on the download page. Without npm, copy
them into a `types/` folder, set `"types": []` and list the side's two files in `files`, for example
`["types/shared.d.ts", "types/server-api.d.ts"]`.

Server and client scripts are checked against different files, so using a client type in a server file is a
compile error.

Each SDK type is a value and a type at the same path, such as `kcds.world.Door`. A name on its own, such as `Door`,
is an error.

## Setting up a project

```text
resources/
  hello-ts/
    package.json
    tsconfig.server.json
    tsconfig.client.json
    src/
      server/main.ts
      client/main.ts
    dist/             compiler output, which the manifest points at
```

`package.json`:

```json
{
  "name": "hello-ts",
  "version": "1.0.0",
  "scripts": {
    "build": "tsc -p tsconfig.server.json && tsc -p tsconfig.client.json"
  },
  "devDependencies": {
    "@kcds/types": "0.4.0",
    "typescript": "^5.9.3"
  },
  "kcds": {
    "serverScripts": ["dist/server/main.js"],
    "clientScripts": ["dist/client/main.js"]
  }
}
```

`tsconfig.server.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "types": ["@kcds/types/server"],
    "strict": true,
    "rootDir": "src/server",
    "outDir": "dist/server"
  },
  "include": ["src/server/**/*.ts"]
}
```

`tsconfig.client.json` is the same, with `client` in place of `server`.

**Why these settings?**

- `"lib": ["ES2022"]` and `"types"` naming only `@kcds/types/server` (or `/client`) leave out browser and Node
  declarations, so a script sees only what the SDK declares.
- Server scripts also have `fetch`, `WebSocket` and the web platform's other classes (`URL`, `Headers`,
  `AbortController` and the rest). For their types, add `"DOM"` to the server project's lib:
  `"lib": ["ES2022", "DOM"]` (with `@kcds/types` 1.0.2 or newer). `require` of Node's own modules (such as
  `node:sqlite`) gives `any`. Do not add `@types/node`: its `require`, `module` and `console` clash with the SDK's.
- Two separate projects keep each side's names out of the other.
- Only `dist/client/` goes to players, because the manifest sends the folder of the client entry script (see
  [Project structure](project-structure.md)).

## Writing the code

`src/server/main.ts`:

```ts
const { Events, Chat } = kcds.core;
const { Player } = kcds.players;

function welcomeLine(name: string): string {
  return `Welcome, ${name}!`;
}

Events.on('playerReady', (player: kcds.players.Player) => {
  Chat.sendToPlayer(player, welcomeLine(player.nickname), { author: 'Server' });
});

Events.fromClient('hello:count', (player: kcds.players.Player) => {
  player.emit('hello:count', { players: Player.all().length });
});
```

`src/client/main.ts`:

```ts
const { Events } = kcds.core;
const { Hud, Key } = kcds.ui;

interface CountAnswer {
  players: number;
}

Key.bind('f7', () => {
  Events.toServer('hello:count');
});

Events.on('hello:count', (data: CountAnswer) => {
  Hud.showNotification(`${data.players} players online`);
});
```

## Building

```text
npm install
npm run build
```

Then enter `ensure hello-ts` in the server console. Build and `ensure` again after each change. Running `tsc
--watch` on both projects rebuilds as you save.

## What gets checked

The compiler catches wrong names, members the SDK does not have, and types from the wrong side. Your editor also
shows each member's summary and return type.

Most parameters are typed `any`. The SDK checks values when the script runs, and throws a `TypeError` or
`RangeError` naming the argument.

The declarations compile with `strict` and `skipLibCheck: false`. Handler parameters are typed, so none is an
implicit `any`:

- game and SDK events, through `KcdsTypes.ServerEvents` and `KcdsTypes.ClientEvents`
- your own event names, with `KcdsTypes.EventHandler`, whose arguments are `any`
- `Messages.handle`, with `KcdsTypes.MessageHandler`, which receives the payload and `reply`
- `PropPlacer.on`, with `KcdsTypes.PlacementHandler`, which receives a pose or `null`, and whether it is valid
- `Key.bind`, `Web.on` and `ActionHint.create`

Event payloads contain whatever the sender put in, so describe your own with an interface, like `CountAnswer`
above.

## Named types

Your code can also use these types from the `KcdsTypes` namespace:

| Type                                  | Describes                                                     |
| ------------------------------------- | ------------------------------------------------------------- |
| `InventorySnapshot`, `InventoryRow`   | `Inventory.get` and `player.getInventory()`, and each row     |
| `InventoryUnit`, `InventoryResult`    | `{ id, amount }` for `Inventory.remove` and `transfer`, and the answer to any change |
| `InventoryChange`                     | the `change` of `playerInventoryChanged`                      |
| `PickpocketedItem`                    | a row of `playerPickpocketed`'s `items`                       |
| `LevelEditDefinition`                 | the argument of `LevelEdit.apply`                             |
| `DialoguePage`, `DialogueOption`      | what `Dialogue.open` and `update` take                        |
| `CookPotSnapshot`, `CookPotFood`      | a pot from `CookPot.all` or `get`; `food` is `'goulash'`, `'lentil'`, `'soup'` or `null` |
| `ScreenEffectInfo`                    | an entry of `Hud.effectList()`                                |
| `Appearance`                          | a body's look                                                 |
| `ProgressionSnapshot`                 | a player's skills and perks                                   |
| `JoinRequest`                         | a join request                                                |

Deprecated members still work but are left out of the declarations. For example, a client blip's `state` has been
`markState` since 0.3.4, as on the server.
