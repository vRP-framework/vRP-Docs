# Admin Module

The admin module adds an administration (kick, ban, etc.) menu and functions.

## Extension

### Client

```lua
Admin:toggleNoclip()
Admin:teleportToMarker()

-- TUNNEL

Admin.tunnel.toggleNoclip = Admin.toggleNoclip
Admin.tunnel.teleportToMarker = Admin.teleportToMarker
```

## Menu

### admin

Administration menu.

### admin.users

Administration sub-menu: user list.

### admin.users.user

Administration sub-menu: user.

**Data:**

* `id` - user id (offline or online)

## Usage Examples

### Basic Admin Functions

```lua
-- Toggle noclip for admin
Admin:toggleNoclip()

-- Teleport admin to marker
Admin:teleportToMarker()

-- Access admin menu
vRP.openMenu("admin")
```

### Admin Menu Structure

The admin module provides a hierarchical menu system:

1. **Main Admin Menu** - Access to all administrative functions
2. **Users List** - View all online and offline users
3. **User Management** - Individual user administration options

### Client-Side Integration

The admin module exposes tunnel functions for client-side access:

```lua
-- From another client resource
local admin = Tunnel.getInterface("vRP.EXT.Admin")
admin.toggleNoclip()
admin.teleportToMarker()
```
