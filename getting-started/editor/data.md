# Data

## **Multi-server and character**

vRP 2 has multi-server and multi-character support. Each server has a string identifier.

{% hint style="info" %}
Players can use another character after spawned, thus extensions should properly handle character load/unload events and check if the character is ready.
{% endhint %}

## **Key-value stores**

vRP provides a key-value API to store custom binary data in the database. A string key is associated to a binary value (ex: a Lua string). The key is a `VARCHAR(100)` and the value a `BLOB` (probably 65535 bytes).

Types

| global        | GData |
| ------------- | ----- |
| per server    | SData |
| per user      | UData |
| per character | CData |

{% hint style="info" %}
vRP core tables, as core files, should not be modified for extensions compatibility. New tables and queries should be created if the key-value system is not powerful enough.
{% endhint %}

## **Core**

vRP core modules data may be stored in specific MySQL tables or using the key-value stores. For the latter, the keys will probably be prefixed by `vRP` and the binary data will use the [MessagePack](https://msgpack.org/) format.

### Script loading <a href="#script_loading" id="script_loading"></a>

To use vRP 2, a script must be loaded in the vRP resource context of a specific side.

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

