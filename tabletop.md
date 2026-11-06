# Tabletop games

A `kcds.world.Tabletop` is a table for a board or card game: checkers, nine men's morris, tic-tac-toe, a
hand of cards. You write only the game's rules. The table puts a board of rows and columns in the world,
stands the pieces on its cells and moves them smoothly, seats the players, keeps the turn order and its
timer, lets a seated player point at the board in their own game, and keeps everyone's view in step,
including players who join later.

The table runs on the server. Players need no client script of yours: their game shows a **Sit down**
hint (U) near a free seat and **Stand up** while they sit, and a seated player points at a cell with the
crosshair and clicks with mouse 1 or Enter. With a camera shot (the table's or the seat's), the view stays
on the board and the mouse pointer shows: the cell under it is the pointed one, as the game's dice
players pick their dice. The arrow keys move the pointed cell as the player sees the board, up being
away from them. While seated, the game's fighting is held off, so a click never swings a weapon.
Their game only sends the click. Your handlers decide what it does, so a player cannot cheat.

`controls` changes how a table is played: `pointer` is `'cursor'`, `'crosshair'` or `'keys'` (only the
arrows), and `block` is the list of the game's actions held off while seated (by default the combat
ones, and the mouse look with the cursor), for example `controls: { pointer: 'keys', block: [] }`.

## A table

```js
const { Tabletop } = kcds.world;

const board = Tabletop.create({
  position: { x: 1650.2, y: 2160.5, z: 36.1 }, // the middle of the board's top
  rotation: 90, // degrees about the vertical axis
  board: { model: 'dice_board', rows: 8, columns: 8, cellSize: 0.07, height: 0.06 },
  seats: [
    { position: { x: 1650.2, y: 2161.1, z: 35.6 }, rotation: 180, side: 'white' },
    { position: { x: 1650.2, y: 2159.9, z: 35.6 }, rotation: 0, side: 'black' },
  ],
  turnSeconds: 60, // a turn passes after a minute; 0 for no limit
});
```

```ts
const { Tabletop } = kcds.world;

const board = Tabletop.create({
  position: { x: 1650.2, y: 2160.5, z: 36.1 }, // the middle of the board's top
  rotation: 90, // degrees about the vertical axis
  board: { model: 'dice_board', rows: 8, columns: 8, cellSize: 0.07, height: 0.06 },
  seats: [
    { position: { x: 1650.2, y: 2161.1, z: 35.6 }, rotation: 180, side: 'white' },
    { position: { x: 1650.2, y: 2159.9, z: 35.6 }, rotation: 0, side: 'black' },
  ],
  turnSeconds: 60, // a turn passes after a minute; 0 for no limit
});
```

```lua
local Tabletop = kcds.world.Tabletop

local board = Tabletop.create({
  position = { x = 1650.2, y = 2160.5, z = 36.1 }, -- the middle of the board's top
  rotation = 90, -- degrees about the vertical axis
  board = { model = "dice_board", rows = 8, columns = 8, cellSize = 0.07, height = 0.06 },
  seats = {
    { position = { x = 1650.2, y = 2161.1, z = 35.6 }, rotation = 180, side = "white" },
    { position = { x = 1650.2, y = 2159.9, z = 35.6 }, rotation = 0, side = "black" },
  },
  turnSeconds = 60, -- a turn passes after a minute; 0 for no limit
})
```

```cs
var board = Tabletop.Create(new
{
    position = new { x = 1650.2, y = 2160.5, z = 36.1 }, // the middle of the board's top
    rotation = 90, // degrees about the vertical axis
    board = new { model = "dice_board", rows = 8, columns = 8, cellSize = 0.07, height = 0.06 },
    seats = new[]
    {
        new { position = new { x = 1650.2, y = 2161.1, z = 35.6 }, rotation = 180, side = "white" },
        new { position = new { x = 1650.2, y = 2159.9, z = 35.6 }, rotation = 0, side = "black" },
    },
    turnSeconds = 60, // a turn passes after a minute; 0 for no limit
});
```

Cells are `{ row, column }` (or `[row, column]`), both from 0: rows run along the table's y axis and columns
along its x axis. `cellPosition(cell)` gives a cell's middle in the world and `cellAt(position)` the cell
under a position. `height` is how far above `position` the pieces stand: the board model's top. The game's
`dice_board` has its top 6 cm above its middle, and 8 cells of 7 cm fill its playing area. A seat can also have an `animation`, `{ fragment, tags }`, which plays while the player
sits, and the table a `camera` (a shot as `Camera.startShot` takes it) that a seated player sees; with a
camera, they point with the arrow keys. A seat's own `camera` comes before the table's, so each player
looks at the board from their side. Give the shot a `look`, `{ yaw, up, down }` in degrees, and the
mouse turns the view that far, as the game's dice players look about their table: the crosshair then
points, and the game's own mouse look is held off. A table belongs to its resource and goes when it
stops.

A seat with `chair: true` is one of the game's own seats: a chair or bench you place with
`SmartObject.spawn` (or the level's own) at the seat's `position`. Players sit on it with the
game's E, with its walk up and sit-down, and the table seats a player when their game reports them
sitting within the seat's `reach` of `position` (0.9 m by default, up to 3 m: a tavern's long bench
seats a player anywhere along it). So any table of the level becomes a game table: put the Tabletop on
its top and its seats where its benches or chairs are. While they sit there, W, A, S and D move the pointed cell like
the arrows, instead of getting them up; **Stand up** (U) gives the keys back, and W then gets them
up from the chair, which stands them up from the table. Such a seat shows no Sit hint and takes no
`animation`:

```js
const { SmartObject, Tabletop } = kcds.world;

// The game's bench for one: its seat is at the prefab's origin, the sitter facing along its +y
// (here turned to face -y, the board). 'Bench/bench_1place_low_table' sits with both hands on the
// table, over a board.
SmartObject.spawn({
  blueprint: 'Bench/bench_1place_low',
  position: [1650.2, 2161.1, 35.0],
  rotation: [0, 0, 1, 0],
});
Tabletop.create({
  position: { x: 1650.2, y: 2160.5, z: 35.77 },
  board: { rows: 8, columns: 8, cellSize: 0.058 },
  seats: [
    {
      position: { x: 1650.2, y: 2161.1, z: 35.0 },
      chair: true,
      camera: {
        position: { x: 1650.2, y: 2160.95, z: 36.27 },
        lookAt: { x: 1650.2, y: 2160.53, z: 35.77 },
        fov: 55,
        look: { yaw: 35, up: 20, down: 25 },
      },
    },
  ],
});
```

```ts
const { SmartObject, Tabletop } = kcds.world;

// The game's bench for one: its seat is at the prefab's origin, the sitter facing along its +y
// (here turned to face -y, the board). 'Bench/bench_1place_low_table' sits with both hands on the
// table, over a board.
SmartObject.spawn({
  blueprint: 'Bench/bench_1place_low',
  position: [1650.2, 2161.1, 35.0],
  rotation: [0, 0, 1, 0],
});
Tabletop.create({
  position: { x: 1650.2, y: 2160.5, z: 35.77 },
  board: { rows: 8, columns: 8, cellSize: 0.058 },
  seats: [
    {
      position: { x: 1650.2, y: 2161.1, z: 35.0 },
      chair: true,
      camera: {
        position: { x: 1650.2, y: 2160.95, z: 36.27 },
        lookAt: { x: 1650.2, y: 2160.53, z: 35.77 },
        fov: 55,
        look: { yaw: 35, up: 20, down: 25 },
      },
    },
  ],
});
```

```lua
local SmartObject = kcds.world.SmartObject
local Tabletop = kcds.world.Tabletop

SmartObject.spawn({
  blueprint = "Bench/bench_1place_low",
  position = { 1650.2, 2161.1, 35.0 },
  rotation = { 0, 0, 1, 0 },
})
Tabletop.create({
  position = { x = 1650.2, y = 2160.5, z = 35.77 },
  board = { rows = 8, columns = 8, cellSize = 0.058 },
  seats = {
    {
      position = { x = 1650.2, y = 2161.1, z = 35.0 },
      chair = true,
      camera = {
        position = { x = 1650.2, y = 2160.95, z = 36.27 },
        lookAt = { x = 1650.2, y = 2160.53, z = 35.77 },
        fov = 55,
        look = { yaw = 35, up = 20, down = 25 },
      },
    },
  },
})
```

```cs
SmartObject.Spawn(new
{
    blueprint = "Bench/bench_1place_low",
    position = new[] { 1650.2, 2161.1, 35.0 },
    rotation = new[] { 0.0, 0.0, 1.0, 0.0 },
});
Tabletop.Create(new
{
    position = new { x = 1650.2, y = 2160.5, z = 35.77 },
    board = new { rows = 8, columns = 8, cellSize = 0.058 },
    seats = new[]
    {
        new
        {
            position = new { x = 1650.2, y = 2161.1, z = 35.0 },
            chair = true,
            camera = new
            {
                position = new { x = 1650.2, y = 2160.95, z = 36.27 },
                lookAt = new { x = 1650.2, y = 2160.53, z = 35.77 },
                fov = 55,
                look = new { yaw = 35, up = 20, down = 25 },
            },
        },
    },
});
```

## Pieces

```js
const stone = board.addPiece({ model: 'dicebadge_balatro_badge_silver', cell: [0, 1], side: 'white' });

await stone.moveTo([1, 2]); // a hop, in 0.4 s
await stone.moveTo([3, 4], { path: 'slide', ms: 600 });
console.info(board.pieceAt([3, 4]) === stone, stone.cell);
stone.remove();
```

```ts
const stone = board.addPiece({ model: 'dicebadge_balatro_badge_silver', cell: [0, 1], side: 'white' });

await stone.moveTo([1, 2]); // a hop, in 0.4 s
await stone.moveTo([3, 4], { path: 'slide', ms: 600 });
console.info(board.pieceAt([3, 4]) === stone, stone.cell);
stone.remove();
```

```lua
local stone = board:addPiece({ model = "dicebadge_balatro_badge_silver", cell = { 0, 1 }, side = "white" })

-- moveTo answers a promise: :next runs once the piece has arrived.
stone:moveTo({ 1, 2 }):next(function()
  print(board:pieceAt({ 1, 2 }) == stone, stone.cell.row, stone.cell.column)
  stone:remove()
end)
```

```cs
var stone = board.AddPiece(new { model = "dicebadge_balatro_badge_silver", cell = new[] { 0, 1 }, side = "white" });

await stone.MoveTo(new[] { 1, 2 }); // a hop, in 0.4 s
await stone.MoveTo(new[] { 3, 4 }, new { path = "slide", ms = 600 });
Log.Info($"{board.PieceAt(new[] { 3, 4 }) == stone} {stone.Cell.ToJson()}");
stone.Remove();
```

A piece is a prop, so every player of the table's dimension sees it, and its `side` and `data` are yours:
whose piece it is, a king's crown, a card's value. Pieces on one cell stack, bottom first in `piecesAt(cell)`.
`moveTo` moves along a hop (the default), a slide or at once (`'jump'`). The board's `model` and the pieces'
are props: the game's own models, such as the dice boards and badges, or meshes your resource ships under
`objects/kcds/...` ([Props](props.md), [Audio and assets](audio-and-assets.md)).

## Seats, turns and moves

```js
const { Events, Chat } = kcds.core;

// Start once both seats are taken.
Events.on('tabletopSeated', (table) => {
  if (table.state === 'idle' && table.seats.every((seat) => seat.player)) {
    table.start();
  }
});

Events.on('tabletopTurn', (table, seat, player) => {
  Chat.sendToPlayer(player, 'Your move.');
});

// A player may pick only their own pieces, and move one only to an empty cell.
Events.on('tabletopPieceSelect', (table, player, piece) => piece.side === table.seats[table.seatOf(player)].side);
Events.on('tabletopMoveRequest', (table, player, piece, cell) => {
  if (table.pieceAt(cell) !== null) {
    return false;
  }
  table.nextTurn();
  return true;
});

Events.on('tabletopEnd', (table, winner, reason) => {
  Chat.sendToAll(winner ? `${winner.nickname} wins (${reason}).` : 'No winner.');
});
```

```ts
const { Events, Chat } = kcds.core;

// Start once both seats are taken.
Events.on('tabletopSeated', (table: kcds.world.Tabletop) => {
  if (table.state === 'idle' && table.seats.every((seat) => seat.player)) {
    table.start();
  }
});

Events.on('tabletopTurn', (table, seat, player) => {
  Chat.sendToPlayer(player, 'Your move.');
});

// A player may pick only their own pieces, and move one only to an empty cell.
Events.on('tabletopPieceSelect', (table, player, piece) => piece.side === table.seats[table.seatOf(player)].side);
Events.on('tabletopMoveRequest', (table, player, piece, cell) => {
  if (table.pieceAt(cell) !== null) {
    return false;
  }
  table.nextTurn();
  return true;
});

Events.on('tabletopEnd', (table, winner, reason) => {
  Chat.sendToAll(winner ? `${winner.nickname} wins (${reason}).` : 'No winner.');
});
```

```lua
local Events, Chat = kcds.core.Events, kcds.core.Chat

-- Start once both seats are taken.
Events.on("tabletopSeated", function(t)
  local full = true
  for _, seat in ipairs(t.seats) do
    full = full and seat.player ~= nil
  end
  if t.state == "idle" and full then
    t:start()
  end
end)

Events.on("tabletopTurn", function(t, seat, player)
  Chat.sendToPlayer(player, "Your move.")
end)

-- A player may pick only their own pieces, and move one only to an empty cell.
Events.on("tabletopPieceSelect", function(t, player, piece)
  return piece.side == t.seats[t:seatOf(player) + 1].side
end)
Events.on("tabletopMoveRequest", function(t, player, piece, cell)
  if t:pieceAt(cell) ~= nil then
    return false
  end
  t:nextTurn()
  return true
end)

Events.on("tabletopEnd", function(t, winner, reason)
  Chat.sendToAll(winner and (winner.nickname .. " wins (" .. reason .. ").") or "No winner.")
end)
```

```cs
// Start once both seats are taken.
Events.On("tabletopSeated", (Tabletop table) =>
{
    if (table.State == "idle" && table.Seats.All((seat) => !seat["player"].IsNull))
    {
        table.Start();
    }
});

Events.On("tabletopTurn", (Tabletop table, double seat, Player player) =>
{
    Chat.SendToPlayer(player, "Your move.");
});

// A player may pick only their own pieces, and move one only to an empty cell.
Events.On("tabletopPieceSelect", (Tabletop table, Player player, TabletopPiece piece) =>
{
    return piece.Side.ToString() == table.Seats[(int)table.SeatOf(player)]["side"].ToString();
});
Events.On("tabletopMoveRequest", (Tabletop table, Player player, TabletopPiece piece, JsValue cell) =>
{
    if (table.PieceAt(cell) != null)
    {
        return false;
    }
    table.NextTurn();
    return true;
});

Events.On("tabletopEnd", (Tabletop table, Player? winner, string reason) =>
{
    Chat.SendToAll(winner != null ? $"{winner.Nickname} wins ({reason})." : "No winner.");
});
```

While a game is on (`start` to `finish`), only the player whose turn it is can pick and move. Picking a piece
raises `tabletopPieceSelect` and marks it for that player; clicking a cell then raises `tabletopMoveRequest`,
and unless a handler answers `false` the piece hops there. The turn passes when you call `nextTurn()`, so a
move that captures can keep it, and when `turnSeconds` run out (`tabletopTurnTimeout`, which can keep it).
`tabletopCellClick` comes before all this for every click: answer `false` to take the click yourself, as a
game that places pieces does. A player who stands up or leaves during a game forfeits it
(`tabletopForfeit`), and with one player left the game ends with them as the winner.

| Event                 | Arguments                         | Refusable                            |
| --------------------- | --------------------------------- | ------------------------------------ |
| `tabletopSit`         | table, player, seat               | yes                                  |
| `tabletopSeated`      | table, player, seat               | no                                   |
| `tabletopLeave`       | table, player, seat, reason       | no                                   |
| `tabletopCellClick`   | table, player, cell               | yes: `false` takes the click         |
| `tabletopPieceSelect` | table, player, piece              | yes                                  |
| `tabletopMoveRequest` | table, player, piece, cell        | yes; otherwise the piece moves there |
| `tabletopStart`       | table                             | no                                   |
| `tabletopTurn`        | table, seat, player               | no                                   |
| `tabletopTurnTimeout` | table, seat, player               | yes: `false` keeps the turn          |
| `tabletopForfeit`     | table, player, seat, reason       | no                                   |
| `tabletopEnd`         | table, winner, reason             | no                                   |

`highlight(cells, { players, color, style })` marks cells for some players or everyone, for example the
moves a picked piece can make, and `clearHighlight(players)` takes the marks away. The players' games
draw the marks flat on the board in the `color` (0xAARRGGBB, a soft gold by default) and the `style`:

| `style`   | Looks like                                                    |
| --------- | ------------------------------------------------------------- |
| `'frame'` | a border round the cell with a faint inside (the default)     |
| `'fill'`  | the whole cell tinted, as for the last move's two cells       |
| `'dot'`   | a round dot in the cell's middle, as for where a piece may go |

The pointed cell is a light blue frame of its own, and the table marks a player's picked piece with a
`'frame'` by itself.

## Hands and bets

```js
// A card only its holder sees, and a stake from everyone seated.
const card = board.addPiece({ model: 'dicebadge_extravalue', cell: [7, 0], side: 'black', visibleTo: player });
if (await board.bet(20)) {
  console.info(`${board.pot} groschen in the pot`);
}
card.showOnlyTo(null); // turned face up for all
```

```ts
// A card only its holder sees, and a stake from everyone seated.
const card = board.addPiece({ model: 'dicebadge_extravalue', cell: [7, 0], side: 'black', visibleTo: player });
if (await board.bet(20)) {
  console.info(`${board.pot} groschen in the pot`);
}
card.showOnlyTo(null); // turned face up for all
```

```lua
-- A card only its holder sees, and a stake from everyone seated.
local card = board:addPiece({ model = "dicebadge_extravalue", cell = { 7, 0 }, side = "black", visibleTo = player })
board:bet(20):next(function(paid)
  if paid then
    print(board.pot .. " groschen in the pot")
  end
  card:showOnlyTo(null) -- turned face up for all
end)
```

```cs
// A card only its holder sees, and a stake from everyone seated.
var card = board.AddPiece(new { model = "dicebadge_extravalue", cell = new[] { 7, 0 }, side = "black", visibleTo = player });
if (await board.Bet(20))
{
    Log.Info($"{board.Pot} groschen in the pot");
}
card.ShowOnlyTo(Sdk.Null); // turned face up for all
```

`bet(amount, { item })` takes the stake (`'money'` by default) from every seated player, all or nothing. When
the game ends, `finish(winner)` gives the pot to the winner, and a game without one gives everyone back what
they put in.

## Decks held in the hands

A table's deck is a set of flat pieces the players really hold: playing cards by default, or your own
(tarot, dominoes, tiles). `table.createDeck()` lays the stack face down on the table. Dealt pieces fly into
the players' left hands. They sit as the game's dice players sit, the forearms on the table, and raise the
left hand before the chest: the pieces stand as a fan on the inside of the fingers, the thumb across their
faces. Each player's game shows them the faces of their own pieces and everyone else the backs, and no other
game ever receives what a hidden piece is. To play, the right hand reaches the fan, takes the piece between
thumb and index, swings over the table and lets go: the piece flies on and lands on its pile.

Give the seats `camera: 'eyes'` (or no camera) and each player keeps the game's own first-person view,
from their own eyes wherever their seat puts them: their hand of pieces before them, the table beyond,
the mouse looking about.

```js
// Five cards each, and the flop face up in a row in the middle.
const [alice, bob] = kcds.players.Player.all();
const deck = board.createDeck();
deck.addPile('board', { position: { x: 0, y: 0.08 }, layout: 'row' });
deck.shuffle();
deck.deal([alice, bob], 5);
deck.dealTo('board', 3);

kcds.core.Events.on('deckPlayRequest', (played, player, pieces) => {
  // Only on their turn, one piece at a time; the deck throws it onto its 'table' pile.
  return board.playerAt(board.turn) === player && pieces.length === 1;
});
```

```ts
// Five cards each, and the flop face up in a row in the middle.
const [alice, bob] = kcds.players.Player.all();
const deck = board.createDeck();
deck.addPile('board', { position: { x: 0, y: 0.08 }, layout: 'row' });
deck.shuffle();
deck.deal([alice, bob], 5);
deck.dealTo('board', 3);

kcds.core.Events.on('deckPlayRequest', (played, player, pieces) => {
  // Only on their turn, one piece at a time; the deck throws it onto its 'table' pile.
  return board.playerAt(board.turn) === player && pieces.length === 1;
});
```

```lua
-- Five cards each, and the flop face up in a row in the middle.
local everyone = kcds.players.Player.all()
local alice, bob = everyone[1], everyone[2]
local deck = board:createDeck()
deck:addPile("board", { position = { x = 0, y = 0.08 }, layout = "row" })
deck:shuffle()
deck:deal({ alice, bob }, 5)
deck:dealTo("board", 3)

kcds.core.Events.on("deckPlayRequest", function(played, player, pieces)
  -- Only on their turn, one piece at a time; the deck throws it onto its 'table' pile.
  return board:playerAt(board.turn) == player and #pieces == 1
end)
```

```cs
// Five cards each, and the flop face up in a row in the middle.
var everyone = Player.All();
var alice = everyone[0];
var bob = everyone[1];
var deck = board.CreateDeck();
deck.AddPile("board", new { position = new { x = 0, y = 0.08 }, layout = "row" });
deck.Shuffle();
deck.Deal(new[] { alice, bob }, 5);
deck.DealTo("board", 3);

Events.On("deckPlayRequest", object? (JsValue played, Player player, JsValue pieces) =>
{
    // Only on their turn, one piece at a time; the deck throws it onto its 'table' pile.
    return board.PlayerAt(board.Turn)?.Id == player.Id && pieces["length"].AsNumber() == 1;
});
```

A seated player works their hand with the keys:

| Key                | What it does                                                                                   |
| ------------------ | ---------------------------------------------------------------------------------------------- |
| Left, Right (A, D) | point at a piece of the hand: it rises a little out of the fan                                 |
| Up (W), mouse 1    | select or unselect the pointed piece: it stands higher (`deckSelect`)                          |
| Enter              | play the selected pieces, or the pointed one (`deckPlayRequest`)                               |
| Down (S)           | lay the hand face down on the table, the hand resting on it, or take it up again (`deckCover`) |
| C, or holding E    | look at the hand close up; again, at the table as the game's dice players do; again, the seat's view |
| Space              | pass: a check, a stand, a pass, as your game names it (`deckPass`)                             |
| E                  | draw a piece from the stack, when the deck has a draw key (`deckDrawRequest`)                  |

A, D, W and S are these keys at a seat with the game's own bench or chair, where the walk keys are held off.
Space, E and holding E show as the game's own key hints, with your labels: `createDeck({ actions: { draw:
'Draw a card', pass: 'Stand', look: 'Look at the cards' } })`, or later `deck.setActions({ pass: 'Check' })`,
for one player with `deck.setActions({ pass: 'Call' }, player)`; `false` takes a key away. At first there
are Pass and Look closer keys and no draw key.

The events are refusable, so the rules decide: `deckSelect(deck, player, piece, selected)`,
`deckPlayRequest(deck, player, pieces)` (otherwise the pieces go to the deck's `playPile`, `'table'` by
default), `deckDrawRequest(deck, player)` (otherwise one piece flies into their hand) and
`deckCover(deck, player, covered)`; `deckPass(deck, player)` only tells you. A script does the same with
`deck.select(piece)`, `deck.play(player, pieces, pile, { faceUp })`, `deck.deal(player)`, `deck.cover(player)`
and `deck.uncover(player)`. `table.lookCloser(player)`, or the player's own C, steps through the close looks:
the cards in their hand, then the table from above, then their seat's view again.

Prší's rules, for example, are a handful of handlers: a card follows the top one by suit or rank, a player
who cannot play draws, and when the stack is empty the played cards but the top one become the stack again.

```js
// Prší with the 32 cards, four each, the first card face up.
const [alice, bob] = kcds.players.Player.all();
const deck = board.createDeck({
  ranks: ['7', '8', '9', 'T', 'J', 'Q', 'K', 'A'],
  actions: { draw: 'Draw a card', pass: false },
});
deck.shuffle();
deck.deal([alice, bob], 4);
deck.dealTo('table', 1, { faceUp: true });
board.start();

const myTurn = (player) => board.playerAt(board.turn) === player;

kcds.core.Events.on('deckPlayRequest', (played, player, pieces) => {
  const top = deck.pile('table').at(-1);
  const [card] = pieces;
  return (
    myTurn(player) && pieces.length === 1 && (card.suit === top?.suit || card.rank === top?.rank)
  );
});

kcds.core.Events.on('deckDrawRequest', (played, player) => myTurn(player));

kcds.core.Events.on('tabletopTurn', async () => {
  if (deck.remaining === 0) {
    await deck.restock('table');
  }
});
```

```ts
// Prší with the 32 cards, four each, the first card face up.
const [alice, bob] = kcds.players.Player.all();
const deck = board.createDeck({
  ranks: ['7', '8', '9', 'T', 'J', 'Q', 'K', 'A'],
  actions: { draw: 'Draw a card', pass: false },
});
deck.shuffle();
deck.deal([alice, bob], 4);
deck.dealTo('table', 1, { faceUp: true });
board.start();

const myTurn = (player: kcds.players.Player) => board.playerAt(board.turn) === player;

kcds.core.Events.on('deckPlayRequest', (played, player, pieces) => {
  const top = deck.pile('table').at(-1);
  const [card] = pieces;
  return (
    myTurn(player) && pieces.length === 1 && (card.suit === top?.suit || card.rank === top?.rank)
  );
});

kcds.core.Events.on('deckDrawRequest', (played, player) => myTurn(player));

kcds.core.Events.on('tabletopTurn', async () => {
  if (deck.remaining === 0) {
    await deck.restock('table');
  }
});
```

```lua
-- Prší with the 32 cards, four each, the first card face up.
local everyone = kcds.players.Player.all()
local alice, bob = everyone[1], everyone[2]
local deck = board:createDeck({
  ranks = { "7", "8", "9", "T", "J", "Q", "K", "A" },
  actions = { draw = "Draw a card", pass = false },
})
deck:shuffle()
deck:deal({ alice, bob }, 4)
deck:dealTo("table", 1, { faceUp = true })
board:start()

local function myTurn(player)
  return board:playerAt(board.turn) == player
end

kcds.core.Events.on("deckPlayRequest", function(played, player, pieces)
  local pile = deck:pile("table")
  local top = pile[#pile]
  local card = pieces[1]
  return myTurn(player) and #pieces == 1 and (card.suit == top.suit or card.rank == top.rank)
end)

kcds.core.Events.on("deckDrawRequest", function(played, player)
  return myTurn(player)
end)

kcds.core.Events.on("tabletopTurn", function()
  if deck.remaining == 0 then
    deck:restock("table")
  end
end)
```

```cs
// Prší with the 32 cards, four each, the first card face up.
var everyone = Player.All();
var alice = everyone[0];
var bob = everyone[1];
var deck = board.CreateDeck(new
{
    ranks = new[] { "7", "8", "9", "T", "J", "Q", "K", "A" },
    actions = new { draw = "Draw a card", pass = false },
});
deck.Shuffle();
deck.Deal(new[] { alice, bob }, 4);
deck.DealTo("table", 1, new { faceUp = true });
board.Start();

bool MyTurn(Player player) => board.PlayerAt(board.Turn)?.Id == player.Id;

Events.On("deckPlayRequest", object? (JsValue played, Player player, JsValue pieces) =>
{
    var top = deck.Pile("table").Last();
    var card = pieces[0];
    var follows = card["suit"].AsString() == top.Suit || card["rank"].AsString() == top.Rank;
    return MyTurn(player) && pieces["length"].AsNumber() == 1 && follows;
});

Events.On("deckDrawRequest", object? (JsValue played, Player player) => MyTurn(player));

Events.On("tabletopTurn", async () =>
{
    if (deck.Remaining == 0)
    {
        await deck.Restock("table");
    }
});
```

Passing the turn is yours: `board.nextTurn()` once a play or a draw is allowed. A seven (the next player
draws two, `deck.deal(next, 2)`) or an ace (they stand, with a Pass key labelled 'Stand' for them) is a few
lines more.

| Deck                                                             | What it does                                                               |
| ---------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `shuffle()`                                                      | shuffles the stack                                                         |
| `deal(players, count)`                                           | round by round, one piece to each player; returns the pieces dealt         |
| `dealTo(pile, count, { faceUp })`                                | from the stack onto a pile                                                 |
| `hand(player)`, `selected(player)`                               | the pieces a player holds, the ones they selected                          |
| `addPile(name, { position, faceUp, layout, spacing, rotation })` | a place on the table: `'stack'` (thrown on a heap) or `'row'`              |
| `pile(name)`                                                     | the pieces on a pile, the last thrown last                                 |
| `move(piece, pile, { faceUp })`                                  | any piece to a pile, without a throw                                       |
| `flip(piece, faceUp)`                                            | turns a piece on a pile over                                               |
| `reveal(player)`, `conceal(player)`                              | the hand shows its faces to everyone in the player's hand, or only to them |
| `collect()`                                                      | every piece flies back into the stack, face down                           |
| `restock(pile, { keep, shuffle })`                               | a pile back into the stack but its top `keep` (1), shuffled                |
| `setActions({ draw, pass }, player)`, `actions(player)`          | the labels of the Space and E keys                                         |
| `save()`, `load(saved)`                                          | the stack's order, the hands and the piles                                 |

A piece (`DeckPiece`) has its `id`, `place` (`'stack'`, `'hand'`, `'pile'` or `'moving'`), `holder`,
`pile`, `faceUp`, `selected`, your own `data`, and for playing cards `rank` (`'A'`, `'2'` ... `'9'`, `'T'`,
`'J'`, `'Q'`, `'K'`) and `suit` (`'S'`, `'H'`, `'D'`, `'C'`). The ids of the standard deck are rank and
suit, `'AS'` to `'KC'`; `createDeck({ jokers: true })` adds `'RJ'` and `'BJ'`, and `createDeck({ ranks:
['7', '8', '9', 'T', 'J', 'Q', 'K', 'A'] })` is the 32-card deck of Prší and Mariáš.

Your own pieces are `createDeck({ pieces: ['d0_0', 'd0_1', ...], models: 'objects/kcds/<your resource>' })`:
the folder holds `<id>.cgf` for each piece (its face on top, its back below, lying on the table) and
`back.cgf` (the back on both sides), in your resource's `stream` folder. A player who leaves the table gives
their hand back under the stack.

## Saving the board

```js
const saved = JSON.stringify(board.save()); // pieces, their data and the turn
board.load(JSON.parse(saved));
```

```ts
const saved = JSON.stringify(board.save()); // pieces, their data and the turn
board.load(JSON.parse(saved));
```

```lua
local saved = json.encode(board:save()) -- pieces, their data and the turn
board:load(json.decode(saved))
```

```cs
var saved = board.Save().ToJson(); // pieces, their data and the turn
board.Load(JsValue.FromJson(saved));
```

Store what `save()` gives where your resource keeps its data, and `load` it after a restart.

## Tic-tac-toe

A whole game: two players sit, each click on an empty cell places their mark, three in a row wins.

```js
// tic-tac-toe
const { Tabletop } = kcds.world;
const { Events, Chat } = kcds.core;

const MARKS = { x: 'dicebadge_balatro_badge_gold', o: 'dicebadge_balatro_badge_silver' };
const LINES = [
  [[0, 0], [0, 1], [0, 2]], [[1, 0], [1, 1], [1, 2]], [[2, 0], [2, 1], [2, 2]],
  [[0, 0], [1, 0], [2, 0]], [[0, 1], [1, 1], [2, 1]], [[0, 2], [1, 2], [2, 2]],
  [[0, 0], [1, 1], [2, 2]], [[0, 2], [1, 1], [2, 0]],
];

const board = Tabletop.create({
  position: { x: 1650.2, y: 2160.5, z: 36.1 },
  board: { model: 'dice_board', rows: 3, columns: 3, cellSize: 0.12, height: 0.06 },
  seats: [
    { position: { x: 1650.2, y: 2161.1, z: 35.6 }, rotation: 180, side: 'x' },
    { position: { x: 1650.2, y: 2159.9, z: 35.6 }, rotation: 0, side: 'o' },
  ],
});

function wins(side) {
  return LINES.some((line) => line.every((cell) => board.pieceAt(cell)?.side === side));
}

Events.on('tabletopSeated', (table) => {
  if (table === board && board.state === 'idle' && board.seats.every((seat) => seat.player)) {
    board.clear();
    board.start();
  }
});

Events.on('tabletopCellClick', (table, player, cell) => {
  if (table !== board) {
    return;
  }
  const side = board.seats[board.seatOf(player)].side;
  if (board.pieceAt(cell) === null) {
    board.addPiece({ model: MARKS[side], cell, side });
    if (wins(side)) {
      board.finish(player, 'three in a row');
    } else if (board.pieces.length === 9) {
      board.finish(null, 'draw');
    } else {
      board.nextTurn();
    }
  }
  return false;
});

Events.on('tabletopEnd', (table, winner, reason) => {
  if (table === board) {
    Chat.sendToAll(winner ? `${winner.nickname} wins at tic-tac-toe: ${reason}` : 'Tic-tac-toe: a draw');
  }
});
```

```ts
// tic-tac-toe
const { Tabletop } = kcds.world;
const { Events, Chat } = kcds.core;

const MARKS = { x: 'dicebadge_balatro_badge_gold', o: 'dicebadge_balatro_badge_silver' };
const LINES = [
  [[0, 0], [0, 1], [0, 2]], [[1, 0], [1, 1], [1, 2]], [[2, 0], [2, 1], [2, 2]],
  [[0, 0], [1, 0], [2, 0]], [[0, 1], [1, 1], [2, 1]], [[0, 2], [1, 2], [2, 2]],
  [[0, 0], [1, 1], [2, 2]], [[0, 2], [1, 1], [2, 0]],
];

const board = Tabletop.create({
  position: { x: 1650.2, y: 2160.5, z: 36.1 },
  board: { model: 'dice_board', rows: 3, columns: 3, cellSize: 0.12, height: 0.06 },
  seats: [
    { position: { x: 1650.2, y: 2161.1, z: 35.6 }, rotation: 180, side: 'x' },
    { position: { x: 1650.2, y: 2159.9, z: 35.6 }, rotation: 0, side: 'o' },
  ],
});

function wins(side: keyof typeof MARKS): boolean {
  return LINES.some((line) => line.every((cell) => board.pieceAt(cell)?.side === side));
}

Events.on('tabletopSeated', (table) => {
  if (table === board && board.state === 'idle' && board.seats.every((seat) => seat.player)) {
    board.clear();
    board.start();
  }
});

Events.on('tabletopCellClick', (table, player, cell) => {
  if (table !== board) {
    return;
  }
  const side: keyof typeof MARKS = board.seats[board.seatOf(player)].side;
  if (board.pieceAt(cell) === null) {
    board.addPiece({ model: MARKS[side], cell, side });
    if (wins(side)) {
      board.finish(player, 'three in a row');
    } else if (board.pieces.length === 9) {
      board.finish(null, 'draw');
    } else {
      board.nextTurn();
    }
  }
  return false;
});

Events.on('tabletopEnd', (table, winner, reason) => {
  if (table === board) {
    Chat.sendToAll(winner ? `${winner.nickname} wins at tic-tac-toe: ${reason}` : 'Tic-tac-toe: a draw');
  }
});
```

```lua
-- tic-tac-toe
local Tabletop = kcds.world.Tabletop
local Events, Chat = kcds.core.Events, kcds.core.Chat

local MARKS = { x = "dicebadge_balatro_badge_gold", o = "dicebadge_balatro_badge_silver" }
local LINES = {
  { { 0, 0 }, { 0, 1 }, { 0, 2 } }, { { 1, 0 }, { 1, 1 }, { 1, 2 } }, { { 2, 0 }, { 2, 1 }, { 2, 2 } },
  { { 0, 0 }, { 1, 0 }, { 2, 0 } }, { { 0, 1 }, { 1, 1 }, { 2, 1 } }, { { 0, 2 }, { 1, 2 }, { 2, 2 } },
  { { 0, 0 }, { 1, 1 }, { 2, 2 } }, { { 0, 2 }, { 1, 1 }, { 2, 0 } },
}

local board = Tabletop.create({
  position = { x = 1650.2, y = 2160.5, z = 36.1 },
  board = { model = "dice_board", rows = 3, columns = 3, cellSize = 0.12, height = 0.06 },
  seats = {
    { position = { x = 1650.2, y = 2161.1, z = 35.6 }, rotation = 180, side = "x" },
    { position = { x = 1650.2, y = 2159.9, z = 35.6 }, rotation = 0, side = "o" },
  },
})

local function wins(side)
  for _, line in ipairs(LINES) do
    local all = true
    for _, cell in ipairs(line) do
      local piece = board:pieceAt(cell)
      all = all and piece ~= nil and piece.side == side
    end
    if all then
      return true
    end
  end
  return false
end

Events.on("tabletopSeated", function(t)
  if t == board and board.state == "idle" and board:playerAt(0) and board:playerAt(1) then
    board:clear()
    board:start()
  end
end)

Events.on("tabletopCellClick", function(t, player, cell)
  if t ~= board then
    return
  end
  local side = board.seats[board:seatOf(player) + 1].side
  if board:pieceAt(cell) == nil then
    board:addPiece({ model = MARKS[side], cell = cell, side = side })
    if wins(side) then
      board:finish(player, "three in a row")
    elseif #board.pieces == 9 then
      board:finish(null, "draw")
    else
      board:nextTurn()
    end
  end
  return false
end)

Events.on("tabletopEnd", function(t, winner, reason)
  if t == board then
    Chat.sendToAll(winner and (winner.nickname .. " wins at tic-tac-toe: " .. reason) or "Tic-tac-toe: a draw")
  end
end)
```

```cs
// tic-tac-toe
public class TicTacToe : Script
{
    private static readonly Dictionary<string, string> Marks = new()
    {
        ["x"] = "dicebadge_balatro_badge_gold",
        ["o"] = "dicebadge_balatro_badge_silver",
    };

    private static readonly int[][][] Lines =
    {
        new[] { new[] { 0, 0 }, new[] { 0, 1 }, new[] { 0, 2 } },
        new[] { new[] { 1, 0 }, new[] { 1, 1 }, new[] { 1, 2 } },
        new[] { new[] { 2, 0 }, new[] { 2, 1 }, new[] { 2, 2 } },
        new[] { new[] { 0, 0 }, new[] { 1, 0 }, new[] { 2, 0 } },
        new[] { new[] { 0, 1 }, new[] { 1, 1 }, new[] { 2, 1 } },
        new[] { new[] { 0, 2 }, new[] { 1, 2 }, new[] { 2, 2 } },
        new[] { new[] { 0, 0 }, new[] { 1, 1 }, new[] { 2, 2 } },
        new[] { new[] { 0, 2 }, new[] { 1, 1 }, new[] { 2, 0 } },
    };

    private Tabletop board = null!;

    private bool Wins(string side)
    {
        return Lines.Any((line) => line.All((cell) => board.PieceAt(cell)?.Side.ToString() == side));
    }

    public override void OnStart()
    {
        board = Tabletop.Create(new
        {
            position = new { x = 1650.2, y = 2160.5, z = 36.1 },
            board = new { model = "dice_board", rows = 3, columns = 3, cellSize = 0.12, height = 0.06 },
            seats = new[]
            {
                new { position = new { x = 1650.2, y = 2161.1, z = 35.6 }, rotation = 180, side = "x" },
                new { position = new { x = 1650.2, y = 2159.9, z = 35.6 }, rotation = 0, side = "o" },
            },
        });

        Events.On("tabletopSeated", (Tabletop table) =>
        {
            if (table == board && board.State == "idle" && board.PlayerAt(0) != null && board.PlayerAt(1) != null)
            {
                board.Clear();
                board.Start();
            }
        });

        Events.On("tabletopCellClick", (Tabletop table, Player player, JsValue cell) =>
        {
            if (table != board)
            {
                return (bool?)null;
            }
            var side = board.Seats[(int)board.SeatOf(player)]["side"].ToString();
            if (board.PieceAt(cell) == null)
            {
                board.AddPiece(new { model = Marks[side], cell, side });
                if (Wins(side))
                {
                    board.Finish(player, "three in a row");
                }
                else if (board.Pieces.Length == 9)
                {
                    board.Finish(Sdk.Null, "draw");
                }
                else
                {
                    board.NextTurn();
                }
            }
            return false;
        });

        Events.On("tabletopEnd", (Tabletop table, Player? winner, string reason) =>
        {
            if (table == board)
            {
                Chat.SendToAll(winner != null ? $"{winner.Nickname} wins at tic-tac-toe: {reason}" : "Tic-tac-toe: a draw");
            }
        });
    }
}
```

## Checkers

The full game on an 8 by 8 board: men move one step diagonally forward, capture by jumping over a rival
piece, keep the turn to jump again, and become kings (which move both ways) on the far row. A side with no
pieces left loses, and a player who walks away forfeits.

```js
// checkers
const { Tabletop } = kcds.world;
const { Events, Chat } = kcds.core;

const MEN = { white: 'dicebadge_balatro_badge_silver', black: 'dicebadge_balatro_badge_plumb' };
const board = Tabletop.create({
  position: { x: 1652.4, y: 2158.3, z: 36.1 },
  board: { model: 'dice_board', rows: 8, columns: 8, cellSize: 0.07, height: 0.06 },
  seats: [
    { position: { x: 1652.4, y: 2158.9, z: 35.6 }, rotation: 180, side: 'white' },
    { position: { x: 1652.4, y: 2157.7, z: 35.6 }, rotation: 0, side: 'black' },
  ],
  turnSeconds: 90,
});
// The piece that jumps again in the middle of a capture run.
let jumping = null;

function sideOf(player) {
  return board.seats[board.seatOf(player)].side;
}

// What a step of `piece` from `from` to `to` is: 'move', 'capture' or null for neither.
function stepKind(piece, from, to) {
  const rows = to.row - from.row;
  const columns = to.column - from.column;
  const forward = piece.side === 'white' ? 1 : -1;
  if (to.row < 0 || to.row > 7 || to.column < 0 || to.column > 7 || Math.abs(rows) !== Math.abs(columns)) {
    return null;
  }
  if (board.pieceAt(to) !== null || (!piece.data.king && Math.sign(rows) !== forward)) {
    return null;
  }
  if (Math.abs(rows) === 1) {
    return 'move';
  }
  const jumped = Math.abs(rows) === 2 ? board.pieceAt({ row: from.row + rows / 2, column: from.column + columns / 2 }) : null;
  return jumped !== null && jumped.side !== piece.side ? 'capture' : null;
}

function canCapture(piece, from) {
  return [-2, 2].some((rows) =>
    [-2, 2].some((columns) => stepKind(piece, from, { row: from.row + rows, column: from.column + columns }) === 'capture'),
  );
}

function newGame() {
  board.clear();
  jumping = null;
  for (let row = 0; row < 8; row++) {
    for (let column = 0; column < 8; column++) {
      if ((row + column) % 2 === 1 && (row < 3 || row > 4)) {
        const side = row < 3 ? 'white' : 'black';
        board.addPiece({ model: MEN[side], cell: { row, column }, side, data: { king: false } });
      }
    }
  }
  board.start();
}

Events.on('tabletopSeated', (table) => {
  if (table === board && board.state === 'idle' && board.seats.every((seat) => seat.player)) {
    newGame();
  }
});

Events.on('tabletopTurn', (table) => {
  if (table === board) {
    jumping = null;
  }
});

Events.on('tabletopPieceSelect', (table, player, piece) => {
  return table !== board || (piece.side === sideOf(player) && (jumping === null || piece === jumping));
});

Events.on('tabletopMoveRequest', (table, player, piece, cell) => {
  if (table !== board) {
    return;
  }
  const from = piece.cell;
  const kind = stepKind(piece, from, cell);
  if (kind === null || (jumping !== null && kind !== 'capture')) {
    return false;
  }
  if (kind === 'capture') {
    board.pieceAt({ row: (from.row + cell.row) / 2, column: (from.column + cell.column) / 2 }).remove();
  }
  if (cell.row === (piece.side === 'white' ? 7 : 0)) {
    piece.data.king = true;
  }
  if (!board.pieces.some((other) => other.side !== piece.side)) {
    board.finish(player, 'all pieces taken');
    return true;
  }
  if (kind === 'capture' && canCapture(piece, cell)) {
    jumping = piece;
  } else {
    board.nextTurn();
  }
  return true;
});

Events.on('tabletopEnd', (table, winner, reason) => {
  if (table === board) {
    Chat.sendToAll(winner ? `${winner.nickname} wins at checkers: ${reason}` : 'Checkers: no winner');
  }
});
```

```ts
// checkers
const { Tabletop } = kcds.world;
const { Events, Chat } = kcds.core;

type Piece = kcds.world.TabletopPiece;
type Cell = { row: number; column: number };

const MEN = { white: 'dicebadge_balatro_badge_silver', black: 'dicebadge_balatro_badge_plumb' };
const board = Tabletop.create({
  position: { x: 1652.4, y: 2158.3, z: 36.1 },
  board: { model: 'dice_board', rows: 8, columns: 8, cellSize: 0.07, height: 0.06 },
  seats: [
    { position: { x: 1652.4, y: 2158.9, z: 35.6 }, rotation: 180, side: 'white' },
    { position: { x: 1652.4, y: 2157.7, z: 35.6 }, rotation: 0, side: 'black' },
  ],
  turnSeconds: 90,
});
// The piece that jumps again in the middle of a capture run.
let jumping: Piece | null = null;

function sideOf(player: kcds.players.Player): string {
  return board.seats[board.seatOf(player)].side;
}

// What a step of `piece` from `from` to `to` is: 'move', 'capture' or null for neither.
function stepKind(piece: Piece, from: Cell, to: Cell): 'move' | 'capture' | null {
  const rows = to.row - from.row;
  const columns = to.column - from.column;
  const forward = piece.side === 'white' ? 1 : -1;
  if (to.row < 0 || to.row > 7 || to.column < 0 || to.column > 7 || Math.abs(rows) !== Math.abs(columns)) {
    return null;
  }
  if (board.pieceAt(to) !== null || (!piece.data.king && Math.sign(rows) !== forward)) {
    return null;
  }
  if (Math.abs(rows) === 1) {
    return 'move';
  }
  const jumped = Math.abs(rows) === 2 ? board.pieceAt({ row: from.row + rows / 2, column: from.column + columns / 2 }) : null;
  return jumped !== null && jumped.side !== piece.side ? 'capture' : null;
}

function canCapture(piece: Piece, from: Cell): boolean {
  return [-2, 2].some((rows) =>
    [-2, 2].some((columns) => stepKind(piece, from, { row: from.row + rows, column: from.column + columns }) === 'capture'),
  );
}

function newGame() {
  board.clear();
  jumping = null;
  for (let row = 0; row < 8; row++) {
    for (let column = 0; column < 8; column++) {
      if ((row + column) % 2 === 1 && (row < 3 || row > 4)) {
        const side = row < 3 ? 'white' : 'black';
        board.addPiece({ model: MEN[side], cell: { row, column }, side, data: { king: false } });
      }
    }
  }
  board.start();
}

Events.on('tabletopSeated', (table) => {
  if (table === board && board.state === 'idle' && board.seats.every((seat) => seat.player)) {
    newGame();
  }
});

Events.on('tabletopTurn', (table) => {
  if (table === board) {
    jumping = null;
  }
});

Events.on('tabletopPieceSelect', (table, player, piece) => {
  return table !== board || (piece.side === sideOf(player) && (jumping === null || piece === jumping));
});

Events.on('tabletopMoveRequest', (table, player, piece, cell) => {
  if (table !== board) {
    return;
  }
  const from = piece.cell;
  const kind = stepKind(piece, from, cell);
  if (kind === null || (jumping !== null && kind !== 'capture')) {
    return false;
  }
  if (kind === 'capture') {
    board.pieceAt({ row: (from.row + cell.row) / 2, column: (from.column + cell.column) / 2 })?.remove();
  }
  if (cell.row === (piece.side === 'white' ? 7 : 0)) {
    piece.data.king = true;
  }
  if (!board.pieces.some((other) => other.side !== piece.side)) {
    board.finish(player, 'all pieces taken');
    return true;
  }
  if (kind === 'capture' && canCapture(piece, cell)) {
    jumping = piece;
  } else {
    board.nextTurn();
  }
  return true;
});

Events.on('tabletopEnd', (table, winner, reason) => {
  if (table === board) {
    Chat.sendToAll(winner ? `${winner.nickname} wins at checkers: ${reason}` : 'Checkers: no winner');
  }
});
```

```lua
-- checkers
local Tabletop = kcds.world.Tabletop
local Events, Chat = kcds.core.Events, kcds.core.Chat

local MEN = { white = "dicebadge_balatro_badge_silver", black = "dicebadge_balatro_badge_plumb" }
local board = Tabletop.create({
  position = { x = 1652.4, y = 2158.3, z = 36.1 },
  board = { model = "dice_board", rows = 8, columns = 8, cellSize = 0.07, height = 0.06 },
  seats = {
    { position = { x = 1652.4, y = 2158.9, z = 35.6 }, rotation = 180, side = "white" },
    { position = { x = 1652.4, y = 2157.7, z = 35.6 }, rotation = 0, side = "black" },
  },
  turnSeconds = 90,
})
-- The piece that jumps again in the middle of a capture run.
local jumping = nil

local function sideOf(player)
  return board.seats[board:seatOf(player) + 1].side
end

-- What a step of `piece` from `from` to `to` is: "move", "capture" or nil for neither.
local function stepKind(piece, from, to)
  local rows, columns = to.row - from.row, to.column - from.column
  local forward = piece.side == "white" and 1 or -1
  if to.row < 0 or to.row > 7 or to.column < 0 or to.column > 7 or math.abs(rows) ~= math.abs(columns) then
    return nil
  end
  local ahead = (rows > 0 and 1 or -1) == forward
  if board:pieceAt(to) ~= nil or (not piece.data.king and not ahead) then
    return nil
  end
  if math.abs(rows) == 1 then
    return "move"
  end
  if math.abs(rows) ~= 2 then
    return nil
  end
  local jumped = board:pieceAt({ row = from.row + rows // 2, column = from.column + columns // 2 })
  if jumped ~= nil and jumped.side ~= piece.side then
    return "capture"
  end
  return nil
end

local function canCapture(piece, from)
  for _, rows in ipairs({ -2, 2 }) do
    for _, columns in ipairs({ -2, 2 }) do
      if stepKind(piece, from, { row = from.row + rows, column = from.column + columns }) == "capture" then
        return true
      end
    end
  end
  return false
end

local function newGame()
  board:clear()
  jumping = nil
  for row = 0, 7 do
    for column = 0, 7 do
      if (row + column) % 2 == 1 and (row < 3 or row > 4) then
        local side = row < 3 and "white" or "black"
        board:addPiece({ model = MEN[side], cell = { row = row, column = column }, side = side, data = { king = false } })
      end
    end
  end
  board:start()
end

Events.on("tabletopSeated", function(t)
  if t == board and board.state == "idle" and board:playerAt(0) and board:playerAt(1) then
    newGame()
  end
end)

Events.on("tabletopTurn", function(t)
  if t == board then
    jumping = nil
  end
end)

Events.on("tabletopPieceSelect", function(t, player, piece)
  return t ~= board or (piece.side == sideOf(player) and (jumping == nil or piece == jumping))
end)

Events.on("tabletopMoveRequest", function(t, player, piece, cell)
  if t ~= board then
    return
  end
  local from = piece.cell
  local kind = stepKind(piece, from, cell)
  if kind == nil or (jumping ~= nil and kind ~= "capture") then
    return false
  end
  if kind == "capture" then
    board:pieceAt({ row = (from.row + cell.row) // 2, column = (from.column + cell.column) // 2 }):remove()
  end
  if cell.row == (piece.side == "white" and 7 or 0) then
    piece.data.king = true
  end
  local rivals = 0
  for _, other in ipairs(board.pieces) do
    if other.side ~= piece.side then
      rivals = rivals + 1
    end
  end
  if rivals == 0 then
    board:finish(player, "all pieces taken")
    return true
  end
  if kind == "capture" and canCapture(piece, cell) then
    jumping = piece
  else
    board:nextTurn()
  end
  return true
end)

Events.on("tabletopEnd", function(t, winner, reason)
  if t == board then
    Chat.sendToAll(winner and (winner.nickname .. " wins at checkers: " .. reason) or "Checkers: no winner")
  end
end)
```

```cs
// checkers
public class Checkers : Script
{
    private static readonly Dictionary<string, string> Men = new()
    {
        ["white"] = "dicebadge_balatro_badge_silver",
        ["black"] = "dicebadge_balatro_badge_plumb",
    };

    private Tabletop board = null!;
    // The piece that jumps again in the middle of a capture run.
    private TabletopPiece? jumping;

    private string SideOf(Player player) => board.Seats[(int)board.SeatOf(player)]["side"].ToString();

    // What a step of `piece` from (fromRow, fromColumn) to (toRow, toColumn) is: "move", "capture" or null.
    private string? StepKind(TabletopPiece piece, int fromRow, int fromColumn, int toRow, int toColumn)
    {
        var rows = toRow - fromRow;
        var columns = toColumn - fromColumn;
        var forward = piece.Side.ToString() == "white" ? 1 : -1;
        if (toRow < 0 || toRow > 7 || toColumn < 0 || toColumn > 7 || Math.Abs(rows) != Math.Abs(columns))
        {
            return null;
        }
        var king = piece.Data["king"].AsBool();
        if (board.PieceAt(new[] { toRow, toColumn }) != null || (!king && Math.Sign(rows) != forward))
        {
            return null;
        }
        if (Math.Abs(rows) == 1)
        {
            return "move";
        }
        var jumped = Math.Abs(rows) == 2 ? board.PieceAt(new[] { fromRow + rows / 2, fromColumn + columns / 2 }) : null;
        return jumped != null && jumped.Side.ToString() != piece.Side.ToString() ? "capture" : null;
    }

    private bool CanCapture(TabletopPiece piece, int row, int column)
    {
        var steps = new[] { -2, 2 };
        return steps.Any((rows) => steps.Any((columns) => StepKind(piece, row, column, row + rows, column + columns) == "capture"));
    }

    private void NewGame()
    {
        board.Clear();
        jumping = null;
        for (var row = 0; row < 8; row++)
        {
            for (var column = 0; column < 8; column++)
            {
                if ((row + column) % 2 == 1 && (row < 3 || row > 4))
                {
                    var side = row < 3 ? "white" : "black";
                    board.AddPiece(new { model = Men[side], cell = new[] { row, column }, side, data = new { king = false } });
                }
            }
        }
        board.Start();
    }

    public override void OnStart()
    {
        board = Tabletop.Create(new
        {
            position = new { x = 1652.4, y = 2158.3, z = 36.1 },
            board = new { model = "dice_board", rows = 8, columns = 8, cellSize = 0.07, height = 0.06 },
            seats = new[]
            {
                new { position = new { x = 1652.4, y = 2158.9, z = 35.6 }, rotation = 180, side = "white" },
                new { position = new { x = 1652.4, y = 2157.7, z = 35.6 }, rotation = 0, side = "black" },
            },
            turnSeconds = 90,
        });

        Events.On("tabletopSeated", (Tabletop table) =>
        {
            if (table == board && board.State == "idle" && board.PlayerAt(0) != null && board.PlayerAt(1) != null)
            {
                NewGame();
            }
        });

        Events.On("tabletopTurn", (Tabletop table) =>
        {
            if (table == board)
            {
                jumping = null;
            }
        });

        Events.On("tabletopPieceSelect", (Tabletop table, Player player, TabletopPiece piece) =>
        {
            return table != board || (piece.Side.ToString() == SideOf(player) && (jumping == null || piece == jumping));
        });

        Events.On("tabletopMoveRequest", (Tabletop table, Player player, TabletopPiece piece, JsValue cell) =>
        {
            if (table != board)
            {
                return (bool?)null;
            }
            var (fromRow, fromColumn) = (piece.Cell["row"].AsInt(), piece.Cell["column"].AsInt());
            var (toRow, toColumn) = (cell["row"].AsInt(), cell["column"].AsInt());
            var kind = StepKind(piece, fromRow, fromColumn, toRow, toColumn);
            if (kind == null || (jumping != null && kind != "capture"))
            {
                return false;
            }
            if (kind == "capture")
            {
                board.PieceAt(new[] { (fromRow + toRow) / 2, (fromColumn + toColumn) / 2 })!.Remove();
            }
            if (toRow == (piece.Side.ToString() == "white" ? 7 : 0))
            {
                piece.Data.Set("king", true);
            }
            if (!board.Pieces.Any((other) => other.Side.ToString() != piece.Side.ToString()))
            {
                board.Finish(player, "all pieces taken");
                return true;
            }
            if (kind == "capture" && CanCapture(piece, toRow, toColumn))
            {
                jumping = piece;
            }
            else
            {
                board.NextTurn();
            }
            return true;
        });

        Events.On("tabletopEnd", (Tabletop table, Player? winner, string reason) =>
        {
            if (table == board)
            {
                Chat.SendToAll(winner != null ? $"{winner.Nickname} wins at checkers: {reason}" : "Checkers: no winner");
            }
        });
    }
}
```

The piece and pointing layer works without a whole game too: any script can put pieces on a table and move
them, and use `cellAt` and `highlight` for its own kind of play.

<!-- only examples -->
## Chess

A whole chess game is an example resource of this SDK, `Dev/examples/resources/chess` in the KCDS
repository. Neither the server nor the game ships it: copy the folder into your server's
`resources` to play. It has a board and pieces of its own (Poly Haven's CC0 chess set, streamed from
the resource's `stream/` folder) and the full rules in `rules.js`: check, castling, en passant,
promotion, checkmate, stalemate, the fifty-move rule, threefold repetition and insufficient
material.

- `/chess` puts the board on the table in front of you, the level's own or any other: your game
  traces straight down ahead of you (`World.raycast`), and a surface 0.55 to 1.2 m above your feet
  is a table's top. The players sit on the benches or chairs already at its two sides, a tavern's
  long benches too (chair seats with a `reach` of 1.3 m), with the game's E. A table without seats
  gets two of the game's benches with `/chess benches`. With no table there, it stands a table of
  its own 2 m ahead, with benches, as `/chess table` always does. `/chess remove` takes away the
  free one nearest to you. White moves first. W, A, S, D and the arrows move the mark; U, then W,
  gets a player up.
- A seated player looks at the board in the game's own first person, and the mouse looks about. The
  arrows pick a piece, then where it
  goes. The legal moves show as green dots for the player who picked the piece, the last move's two
  cells tinted yellow and a king in check framed red for both.
- The pieces slide, a knight jumps, the rook follows the king when castling, and a taken piece leaps
  off the board and stands beside it. Each move reaches both players in the chat, in algebraic
  notation, and the whole game at its end.
- `/resign` gives up, `/draw` offers a draw (the other player's `/draw` accepts it) and
  `/promote q|r|b|n` chooses what a pawn becomes on the last rank (a queen by default).
- The server console has `chess <x> <y> <z> [degrees]` (a table at a place), `chess near <player id>`
  (as that player's `/chess`) and `chess move e2e4` (the side to move plays it).

It is a good start for a game of your own: `server.js` shows the whole flow of a Tabletop game, a
resource's own models and its own animated moves (a move request answered `false` and the pieces
moved by the script).
<!-- end -->

`table.playerAt(seat)` is the player on a seat, or null.

Reference: [Tabletop](/docs/api/tabletop), [TabletopPiece](/docs/api/tabletoppiece), [SmartObject](/docs/api/smartobject).
