# Shops and player trade

Both kinds of trade use the game's shop screen. In both, the screen only estimates the deal. The server
recalculates it from its own data when the player confirms.

## NPC shops

`kcds.characters.Vendor` is a shop stored on the server. It has a shelf of priced rows, a list of what
it pays for players' items, and a purse. Prices are counted in the game's `money` item.

```js
const { Vendor } = kcds.characters;
const { Events } = kcds.core;

const smith = Vendor.create({ name: 'Smith', purse: 5000 });

Vendor.setShelf(smith, [
  { item: 'arrow_crude', amount: 50, price: 12 },
  { item: 'bread', amount: 10, price: 30 },
]);
Vendor.setOffers(smith, [{ item: 'arrow_crude', price: 6 }]);

Events.on('npcInteract', (npc, player) => {
  if (npc.name === 'Smith') {
    Vendor.open(smith, player, npc);
  }
});

Events.on('vendorTrade', (vendor, player, bought, sold, balance, haggled) => {
  console.info(`${player.nickname} traded, balance ${balance}`);
});
```

```lua
local Vendor = kcds.characters.Vendor
local Events = kcds.core.Events

local smith = Vendor.create({ name = "Smith", purse = 5000 })

Vendor.setShelf(smith, {
  { item = "arrow_crude", amount = 50, price = 12 },
  { item = "bread", amount = 10, price = 30 },
})
Vendor.setOffers(smith, { { item = "arrow_crude", price = 6 } })

Events.on("npcInteract", function(npc, player)
  if npc.name == "Smith" then
    Vendor.open(smith, player, npc)
  end
end)

Events.on("vendorTrade", function(vendor, player, bought, sold, balance, haggled)
  print(player.nickname .. " traded, balance " .. balance)
end)
```

```cs
var smith = Vendor.Create(new { name = "Smith", purse = 5000 });

Vendor.SetShelf(smith, new[]
{
    new { item = "arrow_crude", amount = 50, price = 12 },
    new { item = "bread", amount = 10, price = 30 },
});
Vendor.SetOffers(smith, new[] { new { item = "arrow_crude", price = 6 } });

Events.On("npcInteract", (Npc npc, Player player) =>
{
    if (npc.Name == "Smith")
    {
        Vendor.Open(smith, player, npc);
    }
});

Events.On("vendorTrade", (JsValue vendor, Player player, JsValue bought, JsValue sold, double balance, bool haggled) =>
{
    Log.Info($"{player.Nickname} traded, balance {balance}");
});
```

### Setting up

- `Vendor.create({ name?, purse?, buys?, haggle? })`: with no `purse`, the money never runs out.
  With `buys: false`, the shop only sells. With `haggle`, the shop haggles (see
  [Haggling](#haggling)).
- `setShelf(vendor, rows)`: up to 128 rows of `{ item, amount, price, metadata? }`.
- `setOffers(vendor, rows)`: what the shop pays for items.
- `fillPurse` and `purseOf`: set and read its money.

### Running a session

`Vendor.open(vendor, player, npc?)` returns a session id. If you pass an `npc`, the shop closes when
that NPC dies or disappears. Also available: `Vendor.close(session)`, `activeSession(player)` and
`shopOf(session)`.

### What a trade reports

`vendorTrade(vendor, player, bought, sold, balance, haggled)` fires once the server has settled a
basket:

- `bought` and `sold` are lists of `{ item, name, amount, price, metadata }`: the item's class id
  and name, the units, the vendor's price for one unit, and the item's metadata (quality and
  health).
- `balance` is the money the player got, in money units, and a negative number is what they paid.
  Without a haggle it is what the `sold` lines pay minus what the `bought` lines cost.
- `haggled` is `true` when the basket settled at a price agreed in the haggle. Then `balance` is
  minus the agreed price, and the lines still show the vendor's list prices.

A player who buys 3 bread at 30 and sells 2 arrows at 6 gets `balance` -78.

### Refused trades

The server refuses a basket if it was priced against an older shelf, if either side cannot pay, or if
the goods do not fit in the player's inventory. Items players sell do not appear on the shelf.

### Haggling

A shop created with `haggle` haggles in the game's own haggle. The player chooses a price on the
haggle panel, and the trader answers in a dialogue: it takes the offer, asks another price, mocks an
offer made twice, or refuses. The panel shows the trader's price and mood. The server decides every
answer, and a basket agreed in the haggle settles at the agreed price. Haggling needs an `npc` in
`Vendor.open`, because the game haggles with a trader.

```js
const { Vendor } = kcds.characters;
const { Events } = kcds.core;

// At most 15% off, and the trader refuses after 4 offers.
const armourer = Vendor.create({ name: 'Armourer', haggle: { tolerance: 0.15, rounds: 4 } });

// Members of the guild pay no more than 80% if they ask for it.
Events.on('vendorHaggle', (vendor, player, basket, offered) => {
  if (vendor !== armourer || player.state.get('guild') !== 'smiths') {
    return; // the shop's own rule answers
  }
  if (offered >= basket.price * 0.8) {
    return { accept: true };
  }
  return { counter: Math.ceil(basket.price * 0.8), mood: 0.2 };
});
```

```lua
local Vendor = kcds.characters.Vendor
local Events = kcds.core.Events

-- At most 15% off, and the trader refuses after 4 offers.
local armourer = Vendor.create({ name = "Armourer", haggle = { tolerance = 0.15, rounds = 4 } })

-- Members of the guild pay no more than 80% if they ask for it.
Events.on("vendorHaggle", function(vendor, player, basket, offered)
  if vendor ~= armourer or player.state:get("guild") ~= "smiths" then
    return nil -- the shop's own rule answers
  end
  if offered >= basket.price * 0.8 then
    return { accept = true }
  end
  return { counter = math.ceil(basket.price * 0.8), mood = 0.2 }
end)
```

```cs
// At most 15% off, and the trader refuses after 4 offers.
var armourer = Vendor.Create(new { name = "Armourer", haggle = new { tolerance = 0.15, rounds = 4 } });

// Members of the guild pay no more than 80% if they ask for it.
Events.On("vendorHaggle", object? (JsValue vendor, Player player, JsValue basket, double offered) =>
{
    if (vendor.AsNumber() != armourer || player.State.Get("guild").AsString() != "smiths")
    {
        return null; // the shop's own rule answers
    }
    var price = basket["price"].AsNumber();
    if (offered >= price * 0.8)
    {
        return new { accept = true };
    }
    return new { counter = Math.Ceiling(price * 0.8), mood = 0.2 };
});
```

Prices in the haggle are what the player pays, in money units; when the player only sells, they are
negative (what the shop pays).

- `haggle: true` takes the defaults; `{ tolerance, rounds }` sets them. `tolerance` is the most the
  shop gives off the list price, as a share of the basket's value (what the player buys plus what
  they sell), from 0 to 0.35, 0.1 by default. `rounds` is how many offers it hears before it
  refuses, from 1 to 20, 5 by default.
- `vendorHaggle(vendor, player, basket, offered)` answers an offer first. `basket` is `{ bought,
sold, balance, price, asked, round, mood }`: the lines as in `vendorTrade`, the list balance, the
  list price, the shop's last price, the offer's round (1 for the first) and the trader's mood (0
  calm, 1 about to leave). Return `true` or `{ accept: true }` to take the offer, `{ counter:
price, mood? }` to ask another price, `false` or `{ refuse: true }` to stop haggling, or nothing
  to leave the offer to the next handler. A counter at or below the offer takes the offer.
- Without an answer, the shop follows the game's own rule. An offer at or above the shop's last
  price is taken. Otherwise the shop comes part of the way towards the offer, and takes it once the
  two are close. Each offer far from the shop's price makes the trader more impatient, and an
  impatient trader refuses. The player's Speech moves the shop further and makes it take more: the
  game's barter dominance, from -1 at Speech 0 to 1 at Speech 30. The shop never goes below the
  list price minus `tolerance`, nor pays more than its purse holds.
- A refusal ends the haggling for that session: later offers are refused, and the basket still
  trades at the list price. Another basket, or a shelf that changed, starts a new haggle.
- A basket the player confirms after the trader agreed settles at the agreed price. Any other
  basket settles at the list prices.

### When a shop closes

The event `vendorClosed(vendor, player, reason)` gives one of these reasons:

| Reason | Meaning                   |
| ------ | ------------------------- |
| 0      | the player closed it      |
| 1      | the server closed it      |
| 2      | another shop replaced it  |
| 3      | the screen could not open |
| 4      | the player left           |

Reference: [Vendor](/docs/api/vendor).

## Trade between players

`kcds.players.Trade` shows one player's inventory as a shop on another player's screen. The buyer sees
the seller's items and the seller's money as the purse, but not the seller's money items, quest items
or worn gear. Each basket is checked against both inventories, then both change together or neither
does.

```js
const { Events, Chat } = kcds.core;
const { Player, Trade } = kcds.players;

// "/sell Hans" shows your inventory as a shop on Hans's screen.
Events.on('playerCommand', (seller, command, args) => {
  if (command !== 'sell') {
    return;
  }
  const buyer = Player.all().find((player) => player.nickname === args[0]);
  if (!buyer) {
    return;
  }
  const session = Trade.open(seller, buyer, {
    prices: { apple: 7 },
    buys: true, // Hans can also sell to the seller
    buyPrices: { apple: 3 },
  });
  if (session === 0) {
    Chat.sendToPlayer(seller, 'The trade could not open.');
  }
});

// Return false to refuse a basket; here, no more than 10 of anything at once.
Events.on('tradeItem', (seller, buyer, bought, sold, balance) => {
  return bought.every((line) => line.amount <= 10);
});

Events.on('tradeDone', (seller, buyer, bought, sold, balance) => {
  console.info(`${buyer.nickname} paid ${-balance} to ${seller.nickname}`);
});
```

```lua
local Events, Chat = kcds.core.Events, kcds.core.Chat
local Player, Trade = kcds.players.Player, kcds.players.Trade

-- "/sell Hans" shows your inventory as a shop on Hans's screen.
Events.on("playerCommand", function(seller, command, args)
  if command ~= "sell" then
    return
  end
  local buyer
  for _, player in ipairs(Player.all()) do
    if player.nickname == args[1] then
      buyer = player
    end
  end
  if not buyer then
    return
  end
  local session = Trade.open(seller, buyer, {
    prices = { apple = 7 },
    buys = true, -- Hans can also sell to the seller
    buyPrices = { apple = 3 },
  })
  if session == 0 then
    Chat.sendToPlayer(seller, "The trade could not open.")
  end
end)

-- Return false to refuse a basket; here, no more than 10 of anything at once.
Events.on("tradeItem", function(seller, buyer, bought, sold, balance)
  for _, line in ipairs(bought) do
    if line.amount > 10 then
      return false
    end
  end
  return true
end)

Events.on("tradeDone", function(seller, buyer, bought, sold, balance)
  print(buyer.nickname .. " paid " .. -balance .. " to " .. seller.nickname)
end)
```

```cs
// "/sell Hans" shows your inventory as a shop on Hans's screen.
Events.On("playerCommand", (Player seller, string command, string[] args) =>
{
    if (command != "sell" || args.Length == 0)
    {
        return;
    }
    var buyer = Player.All().FirstOrDefault((player) => player.Nickname == args[0]);
    if (buyer == null)
    {
        return;
    }
    var session = Trade.Open(seller, buyer, new
    {
        prices = new { apple = 7 },
        buys = true, // Hans can also sell to the seller
        buyPrices = new { apple = 3 },
    });
    if (session == 0)
    {
        Chat.SendToPlayer(seller, "The trade could not open.");
    }
});

// Return false to refuse a basket; here, no more than 10 of anything at once.
Events.On("tradeItem", (Player seller, Player buyer, JsValue bought, JsValue sold, double balance) =>
{
    return bought.All((line) => line["amount"].AsNumber() <= 10);
});

Events.On("tradeDone", (Player seller, Player buyer, JsValue bought, JsValue sold, double balance) =>
{
    Log.Info($"{buyer.Nickname} paid {-balance} to {seller.Nickname}");
});
```

In `tradeItem` and `tradeDone`, `bought` and `sold` are lists of `{ item, name, amount, price,
metadata }`. `balance` is the money the buyer receives, and a negative number is what they pay.

**Opening:** `Trade.open(seller, buyer, options?)` returns the session id. It returns 0 if a player is
gone or not yet in the world, if both are the same player, or if a `tradeOpen` handler returned
`false`.

**Prices:** `prices` and `buyPrices` name items the same way as `Items` and take whole money units.
Items without a price use the server's price for their quality, from the item config or the game.

**Changes while open:** if either inventory changes, the screen updates, and a basket made on the old
screen is refused.

**Ending:** a trade ends when any of these happens:

- the buyer closes the screen
- either player leaves or is knocked out
- a script calls `Trade.close(session)`
- the resource that opened it stops

`tradeClosed(seller, buyer, reason)` then fires with the same reasons as `vendorClosed`.

`Trade.activeSession(player)` gives the trade a player is in as buyer or seller, or 0.

Reference: [Trade](/docs/api/trade).
