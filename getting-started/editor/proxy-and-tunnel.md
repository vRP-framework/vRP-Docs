---
description: >-
  The proxy library is used to call other resources functions through a proxy
  event.
---

# Proxy and Tunnel

The idea behind tunnel is to easily access any exposed server function from any client resource, and to access any exposed client function from any server resource.

***

Good practice is to get the interface once and set it as a global. If we want to get multiple times the same interface from the same resource, we need to specify a unique identifier (the name of the resource + a unique id for each one).

{% hint style="info" %}
Tunnel and Proxy are blocking calls in the current coroutine until the values are returned, to bypass this behaviour, especially for the Tunnel to optimize speed (ping latency of each call), use `_` as a prefix for the function name (Proxy/Tunnel interfaces should not have functions starting with `_`). This will discard the returned values, but if we still need them, we can make normal calls in a new Citizen thread with `Citizen.CreateThreadNow` or `async` to have non-blocking code.
{% endhint %}

Citizen event handlers (used by Proxy and Tunnel) may not work while loading the resource; to use the Proxy at loading time, we will need to delay it with `Citizen.CreateThread` or a `SetTimeout`.
