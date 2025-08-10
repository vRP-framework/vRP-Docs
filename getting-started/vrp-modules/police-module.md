# Police Module

This module adds some police features.

## Extension

### User

```lua
self.police_records -- list of strings

-- insert a police record (will do a save)
--- record: text for one line (html)
User:insertPoliceRecord(record)

User:savePoliceRecords()
```

### Server

```lua
self.cfg
self.wantedlvl_users -- map of user => wanted level

Police:getUserWantedLevel(user)
```

### Client

```lua
self.handcuffed -- flag
self.cop -- flag
self.wanted_level
self.current_jail
self.follow_player

-- set player as cop (true or false)
Police:setCop(flag)

-- HANDCUFF

Police:toggleHandcuff()

Police:setHandcuffed(flag)

Police:isHandcuffed()

Police:putInNearestVehicleAsPassenger(radius)

-- FOLLOW

-- follow another player
-- player: nil to disable
Police:followPlayer(player)

-- return player or nil if not following anyone
Police:getFollowedPlayer()

-- JAIL

-- jail the player in a no-top no-bottom cylinder 
Police:jail(x,y,z,radius)

-- unjail the player
Police:unjail()

Police:isJailed()

-- WANTED

Police:applyWantedLevel(new_wanted)

-- TUNNEL

Police.tunnel.setCop = Police.setCop
Police.tunnel.toggleHandcuff = Police.toggleHandcuff
Police.tunnel.setHandcuffed = Police.setHandcuffed
Police.tunnel.isHandcuffed = Police.isHandcuffed
Police.tunnel.putInNearestVehicleAsPassenger = Police.putInNearestVehicleAsPassenger
Police.tunnel.followPlayer = Police.followPlayer
Police.tunnel.getFollowedPlayer = Police.getFollowedPlayer
Police.tunnel.jail = Police.jail
Police.tunnel.unjail = Police.unjail
Police.tunnel.isJailed = Police.isJailed
Police.tunnel.applyWantedLevel = Police.applyWantedLevel
```

## Menu

### police

Police menu.

### police.fine

Police fine sub-menu.

**Data:**

* `tuser` - target user
* `money` - target maximum available money

### police.check

Police check report menu.

**Data:**

* `tuser` - target user

### police\_pc

Police PC menu.

### police\_pc.records

Police PC records sub-menu. Manage police records of a character.

**Data:**

* `tuser` - target user

## Item

### bulletproof\_vest

Bulletproof vest, set maximum armour when used.

## Usage Examples

### Basic Police Operations

```lua
-- Set player as cop
Police:setCop(true)

-- Check if player is cop
if self.cop then
  -- Player is a police officer
end

-- Toggle handcuffs
Police:toggleHandcuff()

-- Check if handcuffed
if Police:isHandcuffed() then
  -- Player is handcuffed
end

-- Set handcuffed state
Police:setHandcuffed(true)
```

### Arrest and Jail System

```lua
-- Jail player at specific location
Police:jail(100.0, 200.0, 30.0, 5.0)

-- Check if player is jailed
if Police:isJailed() then
  -- Player is in jail
end

-- Release player from jail
Police:unjail()

-- Put handcuffed player in nearest vehicle
Police:putInNearestVehicleAsPassenger(10.0)
```

### Wanted Level System

```lua
-- Apply wanted level to player
Police:applyWantedLevel(3)

-- Get player's wanted level
local wanted_level = Police:getUserWantedLevel(user)

-- Check wanted level
if self.wanted_level > 0 then
  -- Player is wanted
end
```

### Following System

```lua
-- Follow another player
Police:followPlayer(target_player)

-- Get currently followed player
local followed = Police:getFollowedPlayer()
if followed then
  -- Currently following someone
end

-- Stop following
Police:followPlayer(nil)
```

### Police Records

```lua
-- Add police record
user:insertPoliceRecord("Arrested for speeding - Officer Johnson")

-- Save police records
user:savePoliceRecords()

-- Get police records
local records = user.police_records
for _, record in ipairs(records) do
  print(record)
end
```

### Police Menu Integration

```lua
-- Open police menu
vRP.openMenu("police")

-- Open police fine menu
vRP.openMenu("police.fine", {tuser = target_user, money = target_money})

-- Open police check menu
vRP.openMenu("police.check", {tuser = target_user})

-- Open police PC records
vRP.openMenu("police_pc.records", {tuser = target_user})
```

### Bulletproof Vest

```lua
-- Give bulletproof vest
user:tryGiveItem("bulletproof_vest", 1)

-- Use bulletproof vest (in item usage)
function MyExt.proxy:useBulletproofVest(user)
  -- Set maximum armour
  PlayerState:setArmour(100)
  user:notify("Bulletproof vest equipped")
end
```

### Police Events

```lua
-- Listen for police events
function MyExt.event:playerSpawn(user, first_spawn)
  if user:hasGroup("police") then
    -- Set up police equipment and permissions
    Police:setCop(true)
  end
end

-- Handle arrest events
function MyExt.proxy:arrestPlayer(arresting_officer, suspect)
  -- Log arrest
  suspect:insertPoliceRecord("Arrested by " .. arresting_officer.name)
  
  -- Apply wanted level
  Police:applyWantedLevel(0)
  
  -- Handcuff suspect
  Police:setHandcuffed(true)
end
```
