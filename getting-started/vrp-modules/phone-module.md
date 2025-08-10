# Phone Module

This module adds a phone system.

## Extension

### User

```lua
self.phone_sms
self.cdata.phone_directory
self.phone_call -- source of correspondent if in phone call

-- send an sms to a phone number
-- return true on success
User:sendSMS(phone, msg)

-- add sms to phone
User:addSMS(phone, msg)

-- get directory name by number for a specific user
User:getPhoneDirectoryName(phone)

User:phoneHangUp()

-- call a phone number
-- return true if the communication is established
User:phoneCall(phone)

-- send smspos to a phone number
-- return true on success
User:sendSMSPos(phone, x,y,z)
```

### Server

```lua
self.cfg

-- Send a service alert to all service listeners
--- sender: user or nil (optional, if not nil, it is a call request alert)
--- service_name: service name
--- x,y,z: coordinates
--- msg: alert message
Phone:sendServiceAlert(sender,service_name,x,y,z,msg)
```

### Client

```lua
-- Client-side phone functionality
```

## Menu

### phone

Phone menu.

### phone.directory

Phone directory sub-menu.

### phone.directory.entry

Phone directory entry sub-menu.

**Data:**

* `phone` - phone number

### phone.sms

Phone sms sub-menu.

### phone.service

Phone service sub-menu.

### phone.announce

Phone announce sub-menu.

## Usage Examples

### Basic Phone Operations

```lua
-- Send SMS to a phone number
if user:sendSMS("555-0123", "Hello!") then
  -- SMS sent successfully
end

-- Add SMS to phone (for incoming messages)
user:addSMS("555-0123", "Hello from someone!")

-- Get contact name by phone number
local name = user:getPhoneDirectoryName("555-0123")

-- Make a phone call
if user:phoneCall("555-0123") then
  -- Call initiated
end

-- Hang up phone
user:phoneHangUp()

-- Send position via SMS
user:sendSMSPos("555-0123", 100.0, 200.0, 30.0)
```

### Service Alerts

```lua
-- Send service alert to all listeners
Phone:sendServiceAlert(nil, "police", 100.0, 200.0, 30.0, "Robbery in progress!")

-- Send call request alert
Phone:sendServiceAlert(user, "ambulance", 150.0, 250.0, 30.0, "Medical emergency!")
```

### Phone Directory Management

```lua
-- Add contact to directory
user.cdata.phone_directory["555-0123"] = "John Doe"

-- Remove contact from directory
user.cdata.phone_directory["555-0123"] = nil

-- Get all contacts
local directory = user.cdata.phone_directory
```

### SMS History

```lua
-- Get SMS history
local sms_history = user.phone_sms

-- Add incoming SMS
user:addSMS("555-0123", "Incoming message")

-- Check if in call
if user.phone_call then
  -- User is currently in a call
  local caller_source = user.phone_call
end
```

### Phone Integration

```lua
-- Listen for phone events
function MyExt.event:playerSpawn(user, first_spawn)
  -- Initialize phone for new player
  if first_spawn then
    -- Set up default contacts or phone settings
  end
end

-- Handle incoming calls
function MyExt.proxy:incomingCall(caller, phone_number)
  -- Handle incoming call logic
  if user:phoneCall(phone_number) then
    -- Call accepted
  end
end
```

### Emergency Services

```lua
-- Emergency number handling
function MyExt.proxy:emergencyCall(user, service)
  local x, y, z = user:getPosition()
  
  if service == "police" then
    Phone:sendServiceAlert(user, "police", x, y, z, "Emergency call from " .. user.name)
  elseif service == "ambulance" then
    Phone:sendServiceAlert(user, "ambulance", x, y, z, "Medical emergency from " .. user.name)
  elseif service == "fire" then
    Phone:sendServiceAlert(user, "fire", x, y, z, "Fire emergency from " .. user.name)
  end
end
```
