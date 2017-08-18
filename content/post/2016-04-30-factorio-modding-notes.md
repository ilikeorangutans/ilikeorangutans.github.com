---
date: 2016-04-30T00:00:00Z
description: Creating a simple Factorio mod
tags:
- Factorio
- Modding
title: Factorio Modding Notes
url: /2016/04/30/factorio-modding-notes/
---



* printing information to player via `player.print()`, you can get `for _,player in ipairs(game.players) do ...`
* Lua gotcha: functions have to be defined before calling them. So setting scripi
* to get hot reloading of `control.lua` you need to install the mod as a dir, not a zip

# Useful Links

* [Lua Events](https://wiki.factorio.com/index.php?title=Lua/Events) events you can subscribe to
