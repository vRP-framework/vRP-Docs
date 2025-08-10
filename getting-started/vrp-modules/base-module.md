# Base Module

vRP base module providing core functionality.

## Extension

### Server

```lua
-- Server-side base functionality
```

### Client

```lua
self.id -- user id
self.cid -- character id
self.players -- map of player id, keeps track of connected players (server id)
self.ragdoll -- flag

-- trigger vRP respawn
Base:triggerRespawn()

-- heading: (optional) entity heading
Base:teleport(x,y,z,heading)

-- teleport vehicle when inside one (placed on ground)
-- heading: (optional) entity heading
Base:vehicleTeleport(x,y,z,heading)

-- return x,y,z
Base:getPosition()

-- return false if in exterior, true if inside a building
Base:isInside()

-- return ped speed (based on velocity)
Base:getSpeed()

-- return dx,dy,dz
Base:getCamDirection()

-- return map of player id => distance
Base:getNearestPlayers(radius)

-- return player id or nil
Base:getNearestPlayer(radius)

-- GTA 5 text notification
Base:notify(msg)

-- GTA 5 picture notification
Base:notifyPicture(icon, type, sender, title, text)

-- SCREEN

-- play a screen effect
-- name, see https://wiki.fivem.net/wiki/Screen_Effects
-- duration: in seconds, if -1, will play until stopScreenEffect is called
Base:playScreenEffect(name, duration)

-- stop a screen effect
-- name, see https://wiki.fivem.net/wiki/Screen_Effects
Base:stopScreenEffect(name)

-- ANIM

-- animations dict and names: http://docs.ragepluginhook.net/html/62951c37-a440-478c-b389-c471230ddfc5.htm

-- play animation (new version)
-- upper: true, only upper body, false, full animation
-- seq: list of animations as {dict,anim_name,loops} (loops is the number of loops, default 1) or a task def (properties: task, play_exit)
-- looping: if true, will infinitely loop the first element of the sequence until stopAnim is called
Base:playAnim(upper, seq, looping)

-- stop animation (new version)
-- upper: true, stop the upper animation, false, stop full animations
Base:stopAnim(upper)

-- RAGDOLL

-- set player ragdoll flag (true or false)
Base:setRagdoll(flag)

-- SOUND
-- some lists: 
-- pastebin.com/A8Ny8AHZ
-- https://wiki.gtanet.work/index.php?title=FrontEndSoundlist

-- play sound at a specific position
Base:playSpatializedSound(dict,name,x,y,z,range)

-- play sound
Base:playSound(dict,name)

-- TUNNEL

Base.tunnel.triggerRespawn = Base.triggerRespawn
Base.tunnel.teleport = Base.teleport
Base.tunnel.vehicleTeleport = Base.vehicleTeleport
Base.tunnel.getPosition = Base.getPosition
Base.tunnel.isInside = Base.isInside
Base.tunnel.getSpeed = Base.getSpeed
Base.tunnel.getNearestPlayers = Base.getNearestPlayers
Base.tunnel.getNearestPlayer = Base.getNearestPlayer
Base.tunnel.notify = Base.notify
Base.tunnel.notifyPicture = Base.notifyPicture
Base.tunnel.playScreenEffect = Base.playScreenEffect
Base.tunnel.stopScreenEffect = Base.stopScreenEffect
Base.tunnel.playAnim = Base.playAnim
Base.tunnel.stopAnim = Base.stopAnim
Base.tunnel.setRagdoll = Base.setRagdoll
Base.tunnel.playSpatializedSound = Base.playSpatializedSound
Base.tunnel.playSound = Base.playSound
```

## Events

| Event              | Description                          |
| ------------------ | ------------------------------------ |
| `playerTeleport()` | called when the player is teleported |

## Menu

### characters

Characters menu.

## Permission

### inside

`!inside`

Check if inside a building (interior). Will do a tunnel call.

## Usage Examples

### Basic Player Operations

```lua
-- Get player position
local x, y, z = Base:getPosition()

-- Teleport player
Base:teleport(100.0, 200.0, 30.0, 90.0)

-- Check if player is inside
if Base:isInside() then
  -- Player is in a building
end

-- Get nearest players
local players = Base:getNearestPlayers(10.0)

-- Send notification
Base:notify("Hello World!")
```

### Animation System

```lua
-- Play full body animation
Base:playAnim(false, {{"mp_common", "givetake1_a", 1}}, false)

-- Play upper body animation with looping
Base:playAnim(true, {{"mp_common", "givetake1_a", 1}}, true)

-- Stop animation
Base:stopAnim(false)
```

### Screen Effects

```lua
-- Play screen effect for 5 seconds
Base:playScreenEffect("DrugsMichaelAliensFight", 5.0)

-- Play screen effect until stopped
Base:playScreenEffect("DrugsMichaelAliensFight", -1)

-- Stop screen effect
Base:stopScreenEffect("DrugsMichaelAliensFight")
```

### Sound System

```lua
-- Play sound at position
Base:playSpatializedSound("DLC_HEIST_FLEECA_SOUNDSET", "BANK_ALARM_1", 100.0, 200.0, 30.0, 50.0)

-- Play sound without position
Base:playSound("DLC_HEIST_FLEECA_SOUNDSET", "BANK_ALARM_1")
```
