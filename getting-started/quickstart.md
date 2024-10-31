---
icon: bullseye-arrow
description: This page aims to help you understand how vRP works, and how it is used.
---

# Understanding vRP



## Concepts

### OOP

vRP 3 Object-Oriented Programming uses the [Luaoop](https://github.com/ImagicTheCat/Luaoop) library.

{% hint style="info" %}
By using OOP, instead of loading resources communicating with vRP through Proxy, extensions can directly be loaded into the vRP resource context. Each extension keeps its data and is able to access other extensions data and methods, making extending vRP easier, more powerful (we can access stuff even if not exposed) and with less overhead.
{% endhint %}

### Asynchronous <a href="#asynchronous" id="asynchronous"></a>

vRP 3 extensively uses asynchronous tasks in a transparent way using coroutines, some functions may not return immediately (or never).

### DB driver <a href="#db_driver" id="db_driver"></a>

A DB driver is a class handling MySQL queries. When the vRP MySQL API is used, the DB driver methods are called to process the queries. If the DB driver is not loaded yet, queries will be cached until loaded.

### Proxy and Tunnel <a href="#proxy_and_tunnel" id="proxy_and_tunnel"></a>

The proxy library is used to call other resources functions through a proxy event.

The idea behind tunnel is to easily access any exposed server function from any client resource, and to access any exposed client function from any server resource.

{% hint style="info" %}
Good practice is to get the interface once and set it as a global. If we want to get multiple times the same interface from the same resource, we need to specify a unique identifier (the name of the resource + a unique id for each one).
{% endhint %}

{% hint style="info" %}
Tunnel and Proxy are blocking calls in the current coroutine until the values are returned, to bypass this behaviour, especially for the Tunnel to optimize speed (ping latency of each call), use `_` as a prefix for the function name (Proxy/Tunnel interfaces should not have functions starting with `_`). This will discard the returned values, but if we still need them, we can make normal calls in a new Citizen thread with `Citizen.CreateThreadNow` or `async` to have non-blocking code.
{% endhint %}

{% hint style="info" %}
Also remember that Citizen event handlers (used by Proxy and Tunnel) may not work while loading the resource; to use the Proxy at loading time, we will need to delay it with `Citizen.CreateThread` or a `SetTimeout`.
{% endhint %}

### Data

#### Multi-server and character

vRP 3 has multi-server and multi-character support. Each server has a string identifier.

{% hint style="info" %}
Players can use another character after spawned, thus extensions should properly handle character load/unload events and check if the character is ready.
{% endhint %}

#### Key-value stores

vRP provides a key-value API to store custom binary data in the database. A string key is associated to a binary value (ex: a Lua string). The key is a VARCHAR(100) and the value a BLOB (probably 65535 bytes).

```
Types

global 	GData
per server 	SData
per user 	UData
per character 	CData
```

{% hint style="info" %}
vRP core tables, as core files, should not be modified for extensions compatibility. New tables and queries should be created if the key-value system is not powerful enough.
{% endhint %}

#### Core

vRP core modules data may be stored in specific MySQL tables or using the key-value stores. For the latter, the keys will probably be prefixed by `vRP` and the binary data will use the [MessagePack](https://msgpack.org) format.

### Script loading

To use vRP, a script must be loaded in the vRP resource context of a specific side.

```lua
-- include `@vrp/lib/utils.lua` in `__resource.lua` (for the targeted side)

local Proxy = module("vrp", "lib/Proxy")
local vRP = Proxy.getInterface("vRP")

vRP.loadScript("my_resource", "vrp") -- load "my_resource/vrp.lua"
```

The content of `vrp.lua` is now executed in the vRP context and can now use the API.

{% hint style="info" %}
`vRP.loadScript()` is a proxy to call `module()` in the vRP context.
{% endhint %}

### Extension

An extension is a class extending vRP.

Two versions of the same extension (same name) can be loaded: for the server-side and the client-side. They will be able to interact with each other through the `tunnel`/`remote` interfaces.

```lua
local MyExt = class("MyExt", vRP.Extension)
```

Loaded extensions are accessibles through the vRP instance:

```lua
vRP.EXT.MyExt:test()
```

{% hint style="info" %}
You can see how an extension is made by looking at the code of vRP [modules](https://vrp-framework.github.io/vrp/modules) or [https://github.com/vRP-framework/vRP-basic-mission](https://github.com/vRP-framework/vRP-basic-mission).
{% endhint %}

#### User

Extensions can extend User properties/methods with a User class (constructor is called).

{% hint style="info" %}
To not conflict with other extensions, make sure the added properties and methods have a very specific name or prefix.
{% endhint %}

```lua
MyExt.User = class("User")
```

#### Event

Extensions can listen to global events by defining methods in the `event` table.

```lua
MyExt.event = {}

function MyExt.event:playerSpawn(user, first_spawn)
end
```

{% hint style="info" %}
Events marked with `(sync)` in the documentation may be called using `vRP:triggerEventSync` which will wait for the listeners to complete, meaning that listeners must return (mostly in a short time frame) in order to let the execution continue normally.
{% endhint %}

#### Proxy and Tunnel

Extensions can listen to proxy/tunnel calls by defining methods in the `proxy` or `tunnel` table.

```lua
MyExt.proxy = {}
function MyExt.proxy:getInfo()
end

-- client-side
MyExt.tunnel = {}
function MyExt.tunnel:test()
end
```

The proxy interface generated will be accessible from other resources like this:

```lua
local my_ext = Proxy.getInterface("vRP.EXT.MyExt")
local info = my_ext.getInfo()
```

{% hint style="info" %}
Extensions don’t need and should not use proxy between them.
{% endhint %}

The tunnel is accessible (from the client-side or server-side extension) through the `remote` table.

```lua
-- server-side
function MyExt.event:playerSpawn(user, first_spawn)
  self.remote._test(user.source)
end

-- client-side
function MyExt.event:playerDeath()
  self.remote._test()
end
```

### Profiler

vRP embeds [ELProfiler](https://github.com/ImagicTheCat/ELProfiler) to profile Lua code, vRP and resources based on it. When a resource loads `@vrp/lib/utils.lua` (which is the case for resources based on vRP), it will setup itself to be recorded by the profiler.

To use the profiler, the module must be enabled in `cfg/modules.lua`. This will keep track of created coroutines to profile them. The overhead is probably small, thus it can be enabled on a live server.

Two options are available (with permissions) in the main and admin menus to respectively profile the client-side or server-side.

{% hint style="info" %}
Profiling has an overhead, but mostly because of the Lua debug hook. Being a statistical/sampling profiler, profiling a long period of time is fine (low memory usage).
{% endhint %}

