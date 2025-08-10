# Group Module

This module adds groups and permissions.

## Extension

### User

```lua
self.cdata.groups

-- return map of groups
User:getGroups()

User:hasGroup(name)

User:addGroup(name)

User:removeGroup(name)

-- get user group by type
-- return group name or nil
User:getGroupByType(gtype)

-- check if the user has a specific permission
User:hasPermission(perm)

-- check if the user has a specific list of permissions (all of them)
User:hasPermissions(perms)
```

### Server

```lua
self.cfg

-- return users list
Group:getUsersByGroup(name)

-- return users list
Group:getUsersByPermission(perm)

-- return title or nil
Group:getGroupTitle(group_name)

-- register a special permission function
-- name: name of the permission -> "!name.[...]"
-- callback(user, params) 
--- params: params (strings) of the permissions, ex "!name.param1.param2" -> ["name", "param1", "param2"]
--- should return true or false/nil
Group:registerPermissionFunction(name, callback)
```

## Events

| Event                                 | Description                         |
| ------------------------------------- | ----------------------------------- |
| `playerJoinGroup(user, name, gtype)`  | called when a player joins a group  |
| `playerLeaveGroup(user, name, gtype)` | called when a player leaves a group |

## Menu

### group\_selector

Group selector menu.

**Data:**

* `name` - selector name
* `groups` - selector table

## Permission

### not

Negate another permission function.

`!not.<...>`

**Example:**

* `!not.group.admin` - check if not an admin

### group

Group check.

`!group.<name>`

**Parameters:**

* `name` - group name

**Example:**

* `!group.admin` - check if an admin

## Usage Examples

### Basic Group Operations

```lua
-- Add user to admin group
user:addGroup("admin")

-- Remove user from group
user:removeGroup("police")

-- Check if user has group
if user:hasGroup("admin") then
  -- User is admin
end

-- Get all user groups
local groups = user:getGroups()
```

### Permission Checking

```lua
-- Check specific permission
if user:hasPermission("admin.kick") then
  -- User can kick players
end

-- Check multiple permissions
if user:hasPermissions({"admin.kick", "admin.ban"}) then
  -- User has both kick and ban permissions
end
```

### Custom Permission Functions

```lua
-- Register custom permission function
Group:registerPermissionFunction("custom", function(user, params)
  -- params[1] = "custom"
  -- params[2] = first parameter
  -- params[3] = second parameter
  
  if params[2] == "money" then
    return user:getWallet() > 1000
  end
  
  return false
end)

-- Usage: !custom.money
```

### Group Management

```lua
-- Get all users in a group
local admins = Group:getUsersByGroup("admin")

-- Get all users with a permission
local kickers = Group:getUsersByPermission("admin.kick")

-- Get group title
local title = Group:getGroupTitle("admin")
```
