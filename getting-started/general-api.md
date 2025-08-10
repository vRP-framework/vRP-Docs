# General API

This section covers the general API functions available in vRP.

## Utils

`lib/utils.lua` defines some useful globals.

```lua
-- side detection
SERVER -- boolean
CLIENT -- boolean

-- load a lua resource file as module (for a specific side)
-- rsc: resource name
-- path: lua file path without extension
module(rsc, path)

class -- Luaoop class

-- create an async returner or a thread (Citizen.CreateThreadNow)
-- func: if passed, will create a thread, otherwise will return an async returner
async(func)

-- convert Lua string to hexadecimal
tohex(str)

-- basic deep clone function (doesn't handle circular references)
clone(t)

parseInt(v)

-- will remove chars not allowed/disabled by strchars
-- allow_policy: if true, will allow all strchars, if false, will allow everything except the strchars
sanitizeString(str, strchars, allow_policy)

splitString(str, sep)
```

## Proxy and Tunnel

### Proxy

```lua
-- add event handler to call interface functions 
-- name: interface name
-- itable: table containing functions
Proxy.addInterface(name, itable)

-- get a proxy interface 
-- name: interface name
-- identifier: (optional) unique string to identify this proxy interface access; if nil, will be the name of the resource
Proxy.getInterface(name, identifier)
```

### Tunnel

```lua
-- set the base delay between Triggers for a destination
-- dest: player source
-- delay: milliseconds (0 for instant trigger)
Tunnel.setDestDelay(dest, delay)

-- bind an interface (listen to net requests)
-- name: interface name
-- interface: table containing functions
Tunnel.bindInterface(name,interface)

-- get a tunnel interface to send requests 
-- name: interface name
-- identifier: (optional) unique string to identify this tunnel interface access; if nil, will be the name of the resource
Tunnel.getInterface(name,identifier)
```

### Interface Usage

* Interface defined function names should not start with an underscore (`_`)
* The tunnel server-side call requires the player source as first parameter
* The tunnel server-side called function can use the global `source` (correct until a TriggerEvent/yield/etc) as the remote player source
* Using an underscore to call a remote function interface ignores (no wait) the returned values

```lua
-- PROXY any side, TUNNEL client-side

-- call and wait for returned values
-- ...: arguments
-- return values
interface.func(...)

-- call without waiting
-- ...: arguments
interface._func(...)

-- TUNNEL server-side

-- call and wait for returned values
-- ...: arguments
-- return values
interface.func(player, ...) -- or _func to ignore returned values
```

## DBDriver

```lua
-- called when the driver is initialized (connection), should return true on success
-- db_cfg: cfg/base.lua .db config
DBDriver:onInit(db_cfg)

-- should prepare the query (@param notation)
DBDriver:onPrepare(name, query)

-- should execute the prepared query
-- params: map of parameters
-- mode: 
--  "query": should return rows, affected
--  "execute": should return affected
--  "scalar": should return a scalar
DBDriver:onQuery(name, params, mode)
```

## Extension

```lua
self.remote -- tunnel interface to other network side

-- level: (optional) level, 0 by default
Extension:log(msg, level)

Extension:error(msg)
```

## User

User inherits from all extensions sub-class User (if registered before the first user instantiation).

```lua
self.source
self.name -- FiveM name (may be steam name)
self.id
self.cid -- character id
self.endpoint -- FiveM endpoint
self.data -- user data
self.cdata -- character data
self.loading_character -- flag
self.use_character_action -- action delay
self.spawns -- spawn count

-- return true if the user character is ready (loaded, not loading)
User:isReady()

User:save()

-- return characters id list
User:getCharacters()

-- return created character id or nil if failed
User:createCharacter()

-- use character
-- return true or false, err_code
-- err_code: 
--  1: delay error, too soon
--  2: already loading
--  3: invalid character
User:useCharacter(id)

-- delete character
-- return true or false on failure
User:deleteCharacter(id)
```

## vRP

### Shared

```lua
self.EXT -- map of name => ext
self.modules -- cfg/modules

vRP.Extension

-- register an extension
-- extension: Extension class
vRP:registerExtension(extension)

-- trigger event (with async call for each listener)
vRP:triggerEvent(name, ...)

-- trigger event and wait for all listeners to complete
vRP:triggerEventSync(name, ...)

-- msg: log message
-- suffix: (optional) category, string
-- level: (optional) level, 0 by default
vRP:log(msg, suffix, level)

-- msg: error message
-- suffix: optional category, string
vRP:error(msg, suffix)
```

#### Events

| Event                      | Description                                                                                                                                         |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `extensionLoad(extension)` | called when an extension is loaded, passing the extension instance (can be used to initialize with another extension when loaded before the latter) |

### Server

```lua
self.cfg -- cfg/base config
self.lang -- loaded lang (https://github.com/ImagicTheCat/Luang)
self.users -- map of id => User
self.pending_users -- pending user source update (first spawn), map of ids key => user
self.users_by_source -- map of source => user
self.users_by_cid -- map of character id => user

-- db/SQL API
self.db_drivers
self.db_driver
self.db_initialized

vRP.DBDriver

-- return identification string for a specific source
vRP.getSourceIdKey(source)

vRP.getPlayerEndpoint(player)

vRP.getPlayerName(player)

-- register a DB driver
-- db_driver: DBDriver class
vRP:registerDBDriver(db_driver)

-- prepare a query
-- name: unique name for the query
-- query: SQL string with @params notation
vRP:prepare(name, query)

-- execute a query
-- name: unique name of the query
-- params: map of parameters
-- mode: default is "query"
--  "query": should return rows (list of map of parameter => value), affected
--  "execute": should return affected
--  "scalar": should return a scalar
vRP:query(name, params, mode)

-- shortcut for vRP.query with "execute"
vRP:execute(name, params)

-- shortcut for vRP.query with "scalar"
vRP:scalar(name, params)

-- user data
-- value: binary string
vRP:setUData(user_id,key,value)

vRP:getUData(user_id,key)

-- character data
-- value: binary string
vRP:setCData(character_id,key,value)

vRP:getCData(character_id,key)

-- server data
-- value: binary string
vRP:setSData(key,value,id)

vRP:getSData(key,id)

-- global data
-- value: binary string
vRP:setGData(key,value)

vRP:getGData(key)

-- reason: (optional)
vRP:kick(user, reason)

vRP:save()
```

#### Events

| Event                            | Description                                                                               |
| -------------------------------- | ----------------------------------------------------------------------------------------- |
| (sync) `characterLoad(user)`     | called right after the character loading                                                  |
| (sync) `characterUnload(user)`   | called before character unloading                                                         |
| `playerJoin(user)`               | called when a player joins (valid user)                                                   |
| `playerRejoin(user)`             | called when a player re-joins (ex: after a crash)                                         |
| `playerDelay(user, state)`       | called when the player tunnel delay changes, `state` is true if delay is enabled          |
| `playerSpawn(user, first_spawn)` | called when the player spawns                                                             |
| `playerDeath(user)`              | called when the player dies                                                               |
| (sync) `playerLeave(user)`       | called before user removal                                                                |
| `save`                           | called when vRP performs a save (can be used to sync the save of external extension data) |

### Client

```lua
self.cfg -- cfg/client config
```

#### Events

| Event           | Description                   |
| --------------- | ----------------------------- |
| `playerSpawn()` | called when the player spawns |
| `playerDeath()` | called when the player dies   |
