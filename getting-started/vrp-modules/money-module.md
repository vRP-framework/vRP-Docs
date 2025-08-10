# Money Module

This module adds money (wallet and bank).

## Extension

### User

```lua
self.cdata.wallet
self.cdata.bank

-- get wallet amount
User:getWallet()

-- get bank amount
User:getBank()

-- set wallet amount
User:setWallet(amount)

-- set bank amount
User:setBank(amount)

-- give money to bank
User:giveBank(amount)

-- give money to wallet
User:giveWallet(amount)

-- try a payment (with wallet)
-- dry: if passed/true, will not affect
-- return true if debited or false
User:tryPayment(amount, dry)

-- try a withdraw (from bank)
-- dry: if passed/true, will not affect
-- return true if withdrawn or false
User:tryWithdraw(amount, dry)

-- try a deposit
-- dry: if passed/true, will not affect
-- return true if deposited or false
User:tryDeposit(amount, dry)

-- try full payment (wallet + bank to complete payment)
-- dry: if passed/true, will not affect
-- return true if debited or false
User:tryFullPayment(amount, dry)
```

### Server

```lua
self.cfg
```

## Events

| Event                     | Description                                               |
| ------------------------- | --------------------------------------------------------- |
| `playerMoneyUpdate(user)` | called when the player money is updated (load, change...) |

## Item

### money

Money packed as item.

### money\_binder

Used to pack an amount of wallet money into money item.

## Transformer processor

Consume and produce money in transformers.

`money` - amount

**Example:**

```lua
...
money = 200
...
```

## Usage Examples

### Basic Money Operations

```lua
-- Get wallet balance
local wallet = user:getWallet()

-- Get bank balance
local bank = user:getBank()

-- Give money to wallet
user:giveWallet(1000)

-- Give money to bank
user:giveBank(5000)

-- Set specific amounts
user:setWallet(2000)
user:setBank(10000)
```

### Payment Operations

```lua
-- Try to pay with wallet only
if user:tryPayment(500) then
  -- Payment successful
else
  -- Not enough money in wallet
end

-- Try to pay with wallet + bank
if user:tryFullPayment(1500) then
  -- Payment successful (takes from wallet first, then bank)
else
  -- Not enough money total
end

-- Dry run (check without actually charging)
if user:tryPayment(1000, true) then
  -- User has enough money
end
```

### Banking Operations

```lua
-- Deposit money from wallet to bank
if user:tryDeposit(500) then
  -- Deposit successful
end

-- Withdraw money from bank to wallet
if user:tryWithdraw(1000) then
  -- Withdrawal successful
end
```

### Money Items

```lua
-- Give money as item
user:tryGiveItem("money", 1000)

-- Take money item
user:tryTakeItem("money", 500)

-- Use money binder to convert wallet money to item
user:tryGiveItem("money_binder", 1)
```

### Event Handling

```lua
-- Listen to money updates
function MyExt.event:playerMoneyUpdate(user)
  local wallet = user:getWallet()
  local bank = user:getBank()
  
  -- Update UI or perform other actions
  print("Money updated - Wallet: " .. wallet .. ", Bank: " .. bank)
end
```
