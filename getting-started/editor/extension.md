---
description: An extension is a class extending vRP.
---

# Extension

Two versions of the same extension (same name) can be loaded: for the server-side and the client-side. They will be able to interact with each other through the `tunnel`/`remote` interfaces.

```lua
-- Example
local MyExt = class("MyExt", vRP.Extension)
```

Loaded extensions are accessibles through the vRP instance:

<pre class="language-lua"><code class="lang-lua"><strong>-- Example
</strong><strong>vRP.EXT.MyExt:test()
</strong></code></pre>

{% hint style="info" %}
You can see how an extension is made by looking at the code of vRP [modules](https://vrp-framework.github.io/vrp/modules) or [https://github.com/vRP-framework/vRP-basic-mission](https://github.com/vRP-framework/vRP-basic-mission).
{% endhint %}

{% embed url="https://github.com/vRP-framework/vRP-basic-mission" %}

{% embed url="https://vrp-framework.github.io/vrp/modules" %}

## **User**

Extensions can extend User properties/methods with a User class (constructor is called).

{% hint style="warning" %}
To not conflict with other extensions, make sure the added properties and methods have a very specific name or prefix.
{% endhint %}

```lua
--Example
MyExt.User = class("User")
```

## **Event**

Extensions can listen to global events by defining methods in the `event` table.

```lua
--Example
MyExt.event = {}

function MyExt.event:playerSpawn(user, first_spawn)
end
```

{% hint style="info" %}
Events marked with `(sync)` in the documentation may be called using `vRP:triggerEventSync` which will wait for the listeners to complete, meaning that listeners must return (mostly in a short time frame) in order to let the execution continue normally.
{% endhint %}

## **Proxy and Tunnel**

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

{% hint style="warning" %}
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

#### Profiler <a href="#profiler" id="profiler"></a>

vRP embeds [ELProfiler](https://github.com/ImagicTheCat/ELProfiler) to profile Lua code, vRP and resources based on it. When a resource loads `@vrp/lib/utils.lua` (which is the case for resources based on vRP), it will setup itself to be recorded by the profiler.

To use the profiler, the module must be enabled in `cfg/modules.lua`. This will keep track of created coroutines to profile them. The overhead is probably small, thus it can be enabled on a live server.

Two options are available (with permissions) in the main and admin menus to respectively profile the client-side or server-side.

{% hint style="info" %}
Profiling has an overhead, but mostly because of the Lua debug hook. Being a statistical/sampling profiler, profiling a long period of time is fine (low memory usage).
{% endhint %}
