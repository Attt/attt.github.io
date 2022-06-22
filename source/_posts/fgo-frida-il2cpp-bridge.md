---
title: Frida-il2cpp-bridge script for fate/go
date: 2022-03-17 15:10:42
categories: 
	- Amateur Exp.
tags:
  - wallhack
  - injection
  - reverse engineering
  - fgo
---

hack script for fgo based on [frida](https://github.com/frida/frida) and [frida-il2cpp-bridge](https://github.com/vfsfitvnm/frida-il2cpp-bridge) 

### Supports

- bilibili fate go v2.36.0

- aniplex fate go v2.49.0
  
  *fate go in other channels/platforms/versions may be functional but it's not tested*

### Steps

#### For Android

**Android 10 or 11 is recommended, among testing in android 9 sometimes game may crash*

- Install nodeJs for NPM

- Install frida and frida-tools with ` npm install frida && npm install frida-tools`

- Download latest frida-server from https://github.com/frida/frida/releases

- Start rooted device, rettach with ` adb root`

- Install game apk

- Copy ***frida-server*** file to device directory such as ***/data/local/tmp***

- Run `adb shell` to connect to device

- Change ***frida-server*** file permission with `chmod 777 {directory}/frida-server`

- Navigate to directory and run server with `./frida-server`

#### For IOS

- TODO

#### Run script

- See comments in ***fgo_hack_script.ts***, and change codes according to the *fgo* version
  
  - there is some tiny diffrences between *bili fatego* and *aniplex fatego*

- Build with `./build_aniplex.sh` or `./build_bili.sh`

- Run with `./run.sh com.bilibili.fgo` or `./run.sh com.aniplex.fategrandorder`
  
  - you could now see some logs (if codes like `console.log(...)` is uncommented) when starting battle

- Have fun.

### Features

![in battle](/images/0.jpg)

#### active

1. Increase player's servant hp (by 500,000)

2. Set all skills lv to 10 (both servant's and master's)

3. Change all skills charge turn down (to 1)

4. Try to disable codestage anti-cheat engine (notice that there is **NO WAY** to cheat safely cause anyway the battle data will be sent to game server)

#### inactive

1. Never die (reset hp every time servant dies)

2. Increase attack & defence np rate

3. Increase attack star rate

### Issues

- Could not ran with X64-based simulator such as MuMu (tested) at this time ([frida-il2cpp-bridge](https://github.com/vfsfitvnm/frida-il2cpp-bridge) version 0.7.9)

- Features in [inactive](#inactive) may invoked too frequently that may cause some leakage
