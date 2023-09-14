---
title: "用nssm.exe创建windows服务"
date: 2023-09-14T20:56:49+08:00
tags: ["nssm.exe", "windows server", "windows service", "manual"]
draft: false
---
## 下载
[https://nssm.cc/](https://nssm.cc/)

## 使用

### help doc

```cmd
PS C:\Users\Administrator> nssm -h
NSSM: The non-sucking service manager
Version 2.24-103-gdee49fc 64-bit, 2017-05-16
Usage: nssm <option> [<args> ...]

To show service installation GUI:

        nssm install [<servicename>]

To install a service without confirmation:

        nssm install <servicename> <app> [<args> ...]

To show service editing GUI:

        nssm edit <servicename>

To retrieve or edit service parameters directly:

        nssm dump <servicename>

        nssm get <servicename> <parameter> [<subparameter>]

        nssm set <servicename> <parameter> [<subparameter>] <value>

        nssm reset <servicename> <parameter> [<subparameter>]

To show service removal GUI:

        nssm remove [<servicename>]

To remove a service without confirmation:

        nssm remove <servicename> confirm

To manage a service:

        nssm start <servicename>

        nssm stop <servicename>

        nssm restart <servicename>

        nssm status <servicename>

        nssm statuscode <servicename>

        nssm rotate <servicename>

        nssm processes <servicename>
```
### 通用配置

- directory = 运行环境，配置文件或者日志一类的如果用相对路径的话，这个目录就是相对路径的起始路径
- arguments = 参数(Unix: -x, GNU: --xx, BSD: xy)

### 可执、脚本行文件服务化（exe、bat、ps）

![exe](/images/nssm_manual/scrshot01.png)

- path = 可执行文件

### Java服务化（jar）

![jar](/images/nssm_manual/scrshot02.png)

- path = java.exe




