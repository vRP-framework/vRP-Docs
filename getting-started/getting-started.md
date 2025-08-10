---
description: This guide will help you get started with vRP development.
---

# Getting Started

## Installation

### Prerequisites

* FiveM server
* Basic knowledge of Lua programming
* MySQL database

### Steps

* Clone the repository or download the master [archive](https://github.com/vRP-framework/vRP/archive/master.zip)&#x20;
* copy the `vrp` directory to the resources folder.&#x20;
* Add `vrp` to the loading resource list.

## Configuration <a href="#configuration" id="configuration"></a>

{% hint style="warning" %}
Only the files in the `cfg/` directory should be modified.
{% endhint %}

There is one required file to configure before launching the server, `cfg/base.lua`, to setup the MySQL database credentials, but it also depends on the DB driver used.

There is a lot to configure in vRP, everyone can make his unique server.\
See the vRP documentation and configuration files, default stuff is here as an example.

## **Database**

To run vRP, we also need to install a DB driver, it is a resource/extension used to communicate with a MySQL database.

{% hint style="info" %}
Some maintained DB drivers: [https://github.com/vRP-framework/vRP-db-drivers](https://github.com/vRP-framework/vRP-db-drivers)\
`ghmattimysql` bridge is recommended.
{% endhint %}

DB drivers will register themselves with a specific name to use in `cfg/base.lua`. Since there is no guarantee about when the driver will be registered, all queries will be cached until that moment.

{% hint style="warning" %}
`[vRP] DB driver "driver_name" not initialized yet (X prepares cached, Y queries cached).` is not an error, but a warning that the driver is not registered yet and will stop being outputted if the driver is loaded (a message will also say that the driver is loaded).
{% endhint %}

## Update <a href="#update" id="update"></a>

#### A good way to update:

1. use git to clone vRP to create your own version of it, checkout the wanted branch, create a branch from it
2. create a symbolic link (or an update script) to `vrp/` in the fxserver resources directory
3. (repeat) configure, commit your changes, stay updated with the vRP repository, solve conflicts

This way, you will know when config files should be updated and what exactly has been updated.

#### A more primitive way to update:

1. save your `cfg/` folder somewhere
2. copy all new files in `vrp/`
3. compare your old `cfg/` folder with the new one, fill the gaps
4. replace the new `cfg/` folder with the old modified `cfg/` folder
