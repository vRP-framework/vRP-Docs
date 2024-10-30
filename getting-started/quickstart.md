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
Players can use another character after spawned, thus entensions should properly handle character load/unload events and check if the character is ready
{% endhint %}
