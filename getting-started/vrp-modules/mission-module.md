# Mission Module

This module adds a simple mission system.

## Extension

### User

```lua
self.mission
self.mission_step

-- start a mission for a player
--- mission: 
---- name: mission name
---- steps: ordered list of
----- text: (html)
----- position: {x,y,z}
----- radius: (optional) area radius (affect default PoI)
----- height: (optional) area height (affect default PoI)
----- onenter: see Map.User:setArea
----- onleave: (optional) see Map.User:setArea
----- map_entity: (optional) a simple PoI by default
User:startMission(mission)

-- end the current player mission step
User:nextMissionStep()

-- stop the player mission
User:stopMission()

-- check if the player has a mission
User:hasMission()
```

### Server

```lua
self.cfg
```

## Events

| Event                      | Description                                                                                                             |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `playerMissionStart(user)` | called when the player start a mission (the mission data is attached to the player at the beginning of the call)        |
| `playerMissionStep(user)`  | called when the player start a new mission step                                                                         |
| `playerMissionStop(user)`  | called when the player stop the mission (the mission data is still attached to the player at the beginning of the call) |

## Usage Examples

### Basic Mission Operations

```lua
-- Start a mission
local mission = {
  name = "Delivery Mission",
  steps = {
    {
      text = "Go to the pickup location",
      position = {100.0, 200.0, 30.0},
      radius = 5.0,
      onenter = function(user)
        user:notify("You have reached the pickup location!")
      end
    },
    {
      text = "Deliver the package to the destination",
      position = {150.0, 250.0, 30.0},
      radius = 3.0,
      onenter = function(user)
        user:notify("Package delivered! Mission complete!")
        user:giveWallet(500)
        user:stopMission()
      end
    }
  }
}

user:startMission(mission)

-- Check if player has mission
if user:hasMission() then
  -- Player is on a mission
end

-- Advance to next step
user:nextMissionStep()

-- Stop mission
user:stopMission()
```

### Complex Mission Structure

```lua
-- Multi-step mission with rewards
local delivery_mission = {
  name = "Express Delivery",
  steps = {
    {
      text = "Pick up the package from the warehouse",
      position = {100.0, 200.0, 30.0},
      radius = 5.0,
      onenter = function(user)
        user:notify("Package picked up!")
        user:tryGiveItem("package", 1)
      end
    },
    {
      text = "Drive to the first checkpoint",
      position = {120.0, 220.0, 30.0},
      radius = 8.0,
      onenter = function(user)
        user:notify("Checkpoint reached!")
      end
    },
    {
      text = "Deliver to the customer",
      position = {180.0, 280.0, 30.0},
      radius = 3.0,
      onenter = function(user)
        if user:tryTakeItem("package", 1) then
          user:notify("Delivery successful!")
          user:giveWallet(1000)
          user:giveBank(500)
          user:stopMission()
        else
          user:notify("You lost the package!")
          user:stopMission()
        end
      end
    }
  }
}
```

### Mission with Custom Map Entities

```lua
-- Mission with custom map markers
local custom_mission = {
  name = "Custom Mission",
  steps = {
    {
      text = "Find the hidden location",
      position = {100.0, 200.0, 30.0},
      radius = 10.0,
      map_entity = {
        type = "PoI",
        blip_id = 1,
        blip_color = 2,
        marker_id = 1,
        scale = {1.0, 1.0, 1.0},
        color = {255, 0, 0, 128}
      },
      onenter = function(user)
        user:notify("Location found!")
      end
    }
  }
}
```

### Mission Events

```lua
-- Listen for mission events
function MyExt.event:playerMissionStart(user)
  local mission = user.mission
  print(user.name .. " started mission: " .. mission.name)
  
  -- Send notification to other players
  -- Update mission statistics
  -- etc.
end

function MyExt.event:playerMissionStep(user)
  local step = user.mission.steps[user.mission_step]
  print(user.name .. " reached step: " .. step.text)
end

function MyExt.event:playerMissionStop(user)
  local mission = user.mission
  print(user.name .. " completed mission: " .. mission.name)
  
  -- Award experience points
  -- Update mission completion statistics
  -- etc.
end
```

### Mission Management

```lua
-- Create mission templates
local mission_templates = {
  delivery = {
    name = "Standard Delivery",
    steps = {
      {
        text = "Pick up package",
        position = {100.0, 200.0, 30.0},
        radius = 5.0
      },
      {
        text = "Deliver package",
        position = {150.0, 250.0, 30.0},
        radius = 3.0
      }
    }
  },
  
  escort = {
    name = "Escort Mission",
    steps = {
      {
        text = "Meet the VIP",
        position = {100.0, 200.0, 30.0},
        radius = 5.0
      },
      {
        text = "Escort to destination",
        position = {200.0, 300.0, 30.0},
        radius = 10.0
      }
    }
  }
}

-- Start mission from template
function MyExt.proxy:startMissionFromTemplate(user, template_name)
  local template = mission_templates[template_name]
  if template then
    user:startMission(template)
  end
end
```

### Mission Conditions

```lua
-- Mission with conditions
local conditional_mission = {
  name = "Conditional Mission",
  steps = {
    {
      text = "Collect 5 items",
      position = {100.0, 200.0, 30.0},
      radius = 5.0,
      onenter = function(user)
        local item_count = user:getItemAmount("collectible_item")
        if item_count >= 5 then
          user:notify("Items collected! Moving to next step.")
          user:nextMissionStep()
        else
          user:notify("You need " .. (5 - item_count) .. " more items!")
        end
      end
    },
    {
      text = "Return to base",
      position = {50.0, 150.0, 30.0},
      radius = 3.0,
      onenter = function(user)
        user:notify("Mission complete!")
        user:giveWallet(750)
        user:stopMission()
      end
    }
  }
}
```
