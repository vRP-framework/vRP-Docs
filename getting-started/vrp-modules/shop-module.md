# Shop Module

This module adds item shops.

## Extension

### Server

```lua
self.cfg
```

## Menu

### shop

Shop menu.

**Data:**

* `type` - shop type
* `items` - shop type table

## Usage Examples

### Basic Shop Operations

```lua
-- Open shop menu
vRP.openMenu("shop", {type = "convenience", items = shop_items})

-- Define shop items
local shop_items = {
  ["edible|tacos"] = {price = 10, stock = 50},
  ["edible|burger"] = {price = 15, stock = 30},
  ["drink|cola"] = {price = 5, stock = 100}
}

-- Open shop with items
vRP.openMenu("shop", {type = "convenience", items = shop_items})
```

### Shop Configuration

```lua
-- Shop configuration in cfg/shop.lua
cfg.shops = {
  convenience = {
    title = "Convenience Store",
    items = {
      ["edible|tacos"] = {price = 10, stock = 50},
      ["edible|burger"] = {price = 15, stock = 30},
      ["drink|cola"] = {price = 5, stock = 100}
    }
  },
  
  weapon_shop = {
    title = "Weapon Shop",
    items = {
      ["wbody|WEAPON_PISTOL"] = {price = 1000, stock = 10},
      ["wammo|WEAPON_PISTOL"] = {price = 50, stock = 100}
    }
  }
}
```

### Shop Integration

```lua
-- Create shop NPC
function MyExt.event:playerSpawn(user, first_spawn)
  -- Spawn shop NPCs
  if first_spawn then
    -- Create shop NPCs at specific locations
  end
end

-- Handle shop purchases
function MyExt.proxy:buyItem(user, item_id, amount)
  local shop_items = cfg.shops[shop_type].items
  local item = shop_items[item_id]
  
  if item and item.stock >= amount then
    local total_price = item.price * amount
    
    if user:tryPayment(total_price) then
      if user:tryGiveItem(item_id, amount) then
        -- Update stock
        item.stock = item.stock - amount
        user:notify("Purchase successful!")
        return true
      end
    else
      user:notify("Not enough money!")
    end
  end
  
  return false
end

-- Handle shop sales
function MyExt.proxy:sellItem(user, item_id, amount)
  local shop_items = cfg.shops[shop_type].items
  local item = shop_items[item_id]
  
  if item then
    local total_price = item.price * amount * 0.5 -- 50% of buy price
    
    if user:tryTakeItem(item_id, amount) then
      user:giveWallet(total_price)
      -- Update stock
      item.stock = item.stock + amount
      user:notify("Sale successful!")
      return true
    end
  end
  
  return false
end
```

### Dynamic Shop System

```lua
-- Create dynamic shop based on player location
function MyExt.proxy:openNearbyShop(user)
  local x, y, z = user:getPosition()
  
  -- Check if near convenience store
  if distance(x, y, z, 100, 200, 30) < 5 then
    vRP.openMenu("shop", {type = "convenience", items = cfg.shops.convenience.items})
  end
  
  -- Check if near weapon shop
  if distance(x, y, z, 150, 250, 30) < 5 then
    vRP.openMenu("shop", {type = "weapon_shop", items = cfg.shops.weapon_shop.items})
  end
end

-- Update shop stock dynamically
function MyExt.proxy:updateShopStock(shop_type, item_id, new_stock)
  if cfg.shops[shop_type] and cfg.shops[shop_type].items[item_id] then
    cfg.shops[shop_type].items[item_id].stock = new_stock
  end
end
```

### Shop Permissions

```lua
-- Check if player can access specific shop
function MyExt.proxy:canAccessShop(user, shop_type)
  if shop_type == "weapon_shop" then
    return user:hasPermission("!group.police") or user:hasPermission("!group.gun_dealer")
  end
  
  return true -- Most shops are public
end

-- Apply shop discounts
function MyExt.proxy:getShopDiscount(user, shop_type)
  if user:hasGroup("vip") then
    return 0.1 -- 10% discount for VIP
  end
  
  return 0 -- No discount
end
```

### Shop Events

```lua
-- Listen for shop events
function MyExt.event:shopPurchase(user, shop_type, item_id, amount, price)
  -- Log purchase
  print(user.name .. " purchased " .. amount .. "x " .. item_id .. " for $" .. price)
  
  -- Update statistics
  -- Send notifications to admins
  -- etc.
end

function MyExt.event:shopSale(user, shop_type, item_id, amount, price)
  -- Log sale
  print(user.name .. " sold " .. amount .. "x " .. item_id .. " for $" .. price)
end
```
