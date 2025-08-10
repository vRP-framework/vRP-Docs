# Inventory Module

This module adds an inventory/item system.

## Extension

### User

```lua
self.cdata.inventory

-- return map of fullid => amount
User:getInventory()

-- try to give an item
-- dry: if passed/true, will not affect
-- return true on success or false
User:tryGiveItem(fullid,amount,dry,no_notify)

-- try to take an item from inventory
-- dry: if passed/true, will not affect
-- return true on success or false
User:tryTakeItem(fullid,amount,dry,no_notify)

-- get item amount in the inventory
User:getItemAmount(fullid)

-- return inventory total weight
User:getInventoryWeight()

-- return maximum weight of inventory
User:getInventoryMaxWeight()

User:clearInventory()

-- open a chest by identifier (GData)
-- cb_close(id): called when the chest is closed (optional)
-- cb_in(chest_id, fullid, amount): called when an item is added (optional)
-- cb_out(chest_id, fullid, amount): called when an item is taken (optional)
-- return chest menu or nil
User:openChest(id, max_weight, cb_close, cb_in, cb_out)
```

### Server

```lua
self.cfg
self.items -- item definitions
self.computed_items -- computed item definitions
self.chests -- loaded chests

-- define an inventory item (parametric or plain text data)
-- id: unique item identifier (string, no "." or "|")
-- name: display name, value or genfunction(args)
-- description: value or genfunction(args) (html)
-- menu_builder: (optional) genfunction(args, menu)
-- weight: (optional) value or genfunction(args)
--
-- genfunction are functions returning a correct value as: function(args, ...)
-- where args is a list of {base_idname,args...}
Inventory:defineItem(id,name,description,menu_builder,weight)

-- compute item definition (cached)
-- return computed item or nil
-- computed item {}
--- name
--- description
--- weight
--- menu_builder: can be nil
--- args: parametric args
Inventory:computeItem(fullid)

-- compute weight of a list of items (in inventory/chest format)
Inventory:computeItemsWeight(items)

-- load global chest
-- id: identifier (string)
-- return chest (as inventory, map of fullid => amount)
Inventory:loadChest(id)

-- unload global chest
-- id: identifier (string)
Inventory:unloadChest(id)
```

## Menu

### inventory

Player inventory menu.

### inventory.item

Player inventory item sub-menu.

**Data:**

* `fullid` - item fullid

### chest

Chest menu.

**Data:**

* `id` - chest id
* `chest` - loaded chest
* `max_weight` - chest max weight
* `cb_close` - cb\_close
* `cb_in` - cb\_in
* `cb_out` - cb\_out

### chest.put

Chest put sub-menu. Same data as `chest`.

### chest.take

Chest take sub-menu. Same data as `chest`.

## Permission

### item

Item amount comparison.

`!item.<fullid>.<op>`

**Parameters:**

* `fullid` - item fullid
* `op` - >x, \<x, x (equal) amount

**Examples:**

* `!item.edible|tacos.>0` - one or more tacos
* `!item.dirty_money.0` - no dirty money

## Transformer processor

Consume and produce items in transformers.

`items` - map of item fullid => amount

**Example:**

```lua
...
items = {
  ["edible|peach"] = 1
}
...
```

## Usage Examples

### Basic Inventory Operations

```lua
-- Get player inventory
local inventory = user:getInventory()

-- Give item to player
if user:tryGiveItem("edible|tacos", 5) then
  -- Item given successfully
end

-- Take item from player
if user:tryTakeItem("money", 100) then
  -- Item taken successfully
end

-- Check item amount
local amount = user:getItemAmount("edible|tacos")

-- Get inventory weight
local weight = user:getInventoryWeight()
local max_weight = user:getInventoryMaxWeight()
```

### Item Definition

```lua
-- Define a simple item
Inventory:defineItem("apple", "Apple", "A fresh red apple", nil, 0.1)

-- Define a parametric item
Inventory:defineItem("weapon", function(args)
  return "Weapon " .. args[2]
end, function(args)
  return "A " .. args[2] .. " weapon"
end, nil, 1.0)

-- Usage: weapon|pistol, weapon|rifle
```

### Chest System

```lua
-- Open a chest
local menu = user:openChest("police_evidence", 1000, 
  function(id) 
    print("Chest closed: " .. id)
  end,
  function(chest_id, fullid, amount)
    print("Item added to chest: " .. fullid .. " x" .. amount)
  end,
  function(chest_id, fullid, amount)
    print("Item taken from chest: " .. fullid .. " x" .. amount)
  end
)

-- Load/unload chests
local chest = Inventory:loadChest("police_evidence")
Inventory:unloadChest("police_evidence")
```

### Weight Management

```lua
-- Check if player can carry item
local item_weight = Inventory:computeItem("heavy_item").weight
local current_weight = user:getInventoryWeight()
local max_weight = user:getInventoryMaxWeight()

if current_weight + item_weight <= max_weight then
  -- Can carry item
  user:tryGiveItem("heavy_item", 1)
end
```

### Permission Examples

```lua
-- Check if player has specific items
if user:hasPermission("!item.edible|tacos.>0") then
  -- Player has tacos
end

if user:hasPermission("!item.money.>1000") then
  -- Player has more than 1000 money
end
```
