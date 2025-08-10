# Garage Module

This module manages garages and character vehicles.

Each owned vehicle has a saved state which can be extended with new properties.

**Note:** Vehicles in FiveM/GTA V are entities which can unload/load/despawn themselves on multiple clients, this module tries to handle it this way:

* It tracks the owned vehicles with a decorator and will try to re-own the vehicles when near them
* If multiple versions of the same owned vehicle exist at the same place, it will try to delete the others
* Vehicles states are saved and it will try to re-spawn the vehicles if near their last saved position
* If a vehicle is stuck or completely lost somewhere, players can force its re-spawn at the garage by paying a fee

## Extension

### User

```lua
self.cdata.vehicles
self.cdata.rent_vehicles
self.vehicle_states

-- get owned vehicles
-- return map of model
User:getVehicles()

-- get vehicle model state table (may be async)
User:getVehicleState(model)
```

### Server

```lua
self.cfg
self.models -- map of all garage defined models

-- get vehicle trunk chest id by character id and model
Garage.getVehicleChestId(cid, model)
```

### Client

```lua
self.vehicles -- map of vehicle model => veh id (owned vehicles)
self.hash_models -- map of hash => model

-- veh: vehicle game id
-- return owner character id and model or nil if not managed by vRP
Garage:getVehicleInfo(veh)

-- spawn vehicle (will despawn first)
-- will be placed on ground properly
-- one vehicle per model allowed at the same time
--
-- state: (optional) vehicle state (client)
-- position: (optional) {x,y,z}, if not passed the vehicle will be spawned on the player (and will be put inside the vehicle)
-- rotation: (optional) quaternion {x,y,z,w}, if passed with the position, will be applied to the vehicle entity
Garage:spawnVehicle(model, state, position, rotation) 

-- return true if despawned
Garage:despawnVehicle(model)

Garage:despawnVehicles()

-- get all game vehicles
-- return list of veh
Garage:getAllVehicles()

-- return map of veh => distance
Garage:getNearestVehicles(radius)

-- return veh
Garage:getNearestVehicle(radius)

Garage:trySpawnOutVehicles()

-- try re-own vehicles
Garage:tryOwnVehicles()

-- cleanup invalid owned vehicles
Garage:cleanupVehicles()

Garage:fixNearestVehicle(radius)

Garage:replaceNearestVehicle(radius)

-- return model or nil
Garage:getNearestOwnedVehicle(radius)

-- return ok,x,y,z
Garage:getAnyOwnedVehiclePosition()

-- return x,y,z or nil
Garage:getOwnedVehiclePosition(model)

Garage:putInOwnedVehicle(model)

-- eject the ped from the vehicle
Garage:ejectVehicle()

Garage:isInVehicle()

-- return model or nil if not in owned vehicle
Garage:getInOwnedVehicleModel()

-- VEHICLE STATE

Garage:getVehicleCustomization(veh)

-- partial update per property
Garage:setVehicleCustomization(veh, custom)

Garage:getVehicleState(veh)

-- partial update per property
Garage:setVehicleState(veh, state)

-- VEHICLE COMMANDS

Garage:vc_openDoor(model, door_index)

Garage:vc_closeDoor(model, door_index)

Garage:vc_detachTrailer(model)

Garage:vc_detachTowTruck(model)

Garage:vc_detachCargobob(model)

Garage:vc_toggleEngine(model)

-- return true if locked, false if unlocked
Garage:vc_toggleLock(model)

-- TUNNEL

Garage.tunnel.spawnVehicle = Garage.spawnVehicle
Garage.tunnel.despawnVehicle = Garage.despawnVehicle
Garage.tunnel.despawnVehicles = Garage.despawnVehicles
Garage.tunnel.fixNearestVehicle = Garage.fixNearestVehicle
Garage.tunnel.replaceNearestVehicle = Garage.replaceNearestVehicle
Garage.tunnel.getNearestOwnedVehicle = Garage.getNearestOwnedVehicle
Garage.tunnel.getAnyOwnedVehiclePosition = Garage.getAnyOwnedVehiclePosition
Garage.tunnel.getOwnedVehiclePosition = Garage.getOwnedVehiclePosition
Garage.tunnel.putInOwnedVehicle = Garage.putInOwnedVehicle
Garage.tunnel.getInOwnedVehicleModel = Garage.getInOwnedVehicleModel
Garage.tunnel.trySpawnOutVehicles = Garage.trySpawnOutVehicles
Garage.tunnel.cleanupVehicles = Garage.cleanupVehicles
Garage.tunnel.tryOwnVehicles = Garage.tryOwnVehicles
Garage.tunnel.ejectVehicle = Garage.ejectVehicle
Garage.tunnel.isInVehicle = Garage.isInVehicle
Garage.tunnel.vc_openDoor = Garage.vc_openDoor
Garage.tunnel.vc_closeDoor = Garage.vc_closeDoor
Garage.tunnel.vc_detachTrailer = Garage.vc_detachTrailer
Garage.tunnel.vc_detachTowTruck = Garage.vc_detachTowTruck
Garage.tunnel.vc_detachCargobob = Garage.vc_detachCargobob
Garage.tunnel.vc_toggleEngine = Garage.vc_toggleEngine
Garage.tunnel.vc_toggleLock = Garage.vc_toggleLock
```

## Events

| Event                         | Description                        |
| ----------------------------- | ---------------------------------- |
| `garageVehicleSpawn(model)`   | called when a vehicle is spawned   |
| `garageVehicleDespawn(model)` | called when a vehicle is despawned |

## Item

### repairkit

Used to repair vehicles, but can be used for other stuff.

## Menu

### vehicle

Owned vehicle menu.

**Data:**

* `model` - vehicle model

### garage

Garage menu.

**Data:**

* `type` - garage type
* `vehicles` - garage type table

### garage.owned

Garage sub-menu. Same data as `garage`.

### garage.buy

Garage sub-menu. Same data as `garage`.

### garage.sell

Garage sub-menu. Same data as `garage`.

### garage.rent

Garage sub-menu. Same data as `garage`.

## Permission

### in\_vehicle

`!in_vehicle`

Will do a tunnel call.

### in\_owned\_vehicle

`!in_owned_vehicle[.<model>]`

Will do a tunnel call.

**Parameters:**

* `model` - (optional) vehicle model

**Examples:**

* `!in_owned_vehicle` - check if inside an owned vehicle
* `!in_owned_vehicle.taxi` - check if inside owned taxi model

## Vehicle state

| Property           | Description                       |
| ------------------ | --------------------------------- |
| customization      | customization properties          |
| condition          | damages properties                |
| position, rotation | position/rotation for persistence |
| locked             | doors state                       |

## Usage Examples

### Basic Vehicle Operations

```lua
-- Get owned vehicles
local vehicles = user:getVehicles()

-- Spawn a vehicle
Garage:spawnVehicle("adder")

-- Spawn vehicle at specific position
Garage:spawnVehicle("adder", nil, {100.0, 200.0, 30.0})

-- Despawn vehicle
Garage:despawnVehicle("adder")

-- Check if in vehicle
if Garage:isInVehicle() then
  local model = Garage:getInOwnedVehicleModel()
  if model then
    -- In owned vehicle
  end
end
```

### Vehicle State Management

```lua
-- Get vehicle customization
local custom = Garage:getVehicleCustomization(vehicle_id)

-- Set vehicle customization
Garage:setVehicleCustomization(vehicle_id, {
  color1 = 0,
  color2 = 1,
  -- other customization properties
})

-- Get vehicle state
local state = Garage:getVehicleState(vehicle_id)

-- Set vehicle state
Garage:setVehicleState(vehicle_id, {
  engine_health = 1000,
  body_health = 1000,
  -- other state properties
})
```

### Vehicle Commands

```lua
-- Open/close doors
Garage:vc_openDoor("adder", 0)  -- Front left door
Garage:vc_closeDoor("adder", 0)

-- Toggle engine
Garage:vc_toggleEngine("adder")

-- Toggle lock
local is_locked = Garage:vc_toggleLock("adder")

-- Detach trailers/tow trucks
Garage:vc_detachTrailer("adder")
Garage:vc_detachTowTruck("adder")
Garage:vc_detachCargobob("adder")
```

### Vehicle Management

```lua
-- Get nearest vehicles
local vehicles = Garage:getNearestVehicles(10.0)

-- Get nearest owned vehicle
local model = Garage:getNearestOwnedVehicle(5.0)

-- Fix nearest vehicle
Garage:fixNearestVehicle(5.0)

-- Replace nearest vehicle
Garage:replaceNearestVehicle(5.0)

-- Try to spawn out vehicles
Garage:trySpawnOutVehicles()

-- Try to re-own vehicles
Garage:tryOwnVehicles()

-- Cleanup invalid vehicles
Garage:cleanupVehicles()
```

### Permission Examples

```lua
-- Check if player is in any vehicle
if user:hasPermission("!in_vehicle") then
  -- Player is in a vehicle
end

-- Check if player is in specific owned vehicle
if user:hasPermission("!in_owned_vehicle.taxi") then
  -- Player is in owned taxi
end
```
