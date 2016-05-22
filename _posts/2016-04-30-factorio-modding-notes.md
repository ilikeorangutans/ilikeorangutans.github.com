---
layout: post
title: "Factorio Modding Notes"
description: "Creating a simple Factorio mod"
category:
tags: [Factorio, Modding]
---
{% include JB/setup %}

* printing information to player via `player.print()`, you can get `for _,player in ipairs(game.players) do ...`
* Lua gotcha: functions have to be defined before calling them. So setting scripi
* to get hot reloading of `control.lua` you need to install the mod as a dir, not a zip

# Useful Links

* [Lua Events](https://wiki.factorio.com/index.php?title=Lua/Events) events you can subscribe to
