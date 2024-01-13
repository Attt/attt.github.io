---
title: "Fuck China Mobile"
date: 2024-01-13T20:22:55+08:00
tags: ["fix", "network", "github", "tsukkomi"]
draft: false
---

中国移动网络提交github出现

```bash
kex_exchange_identification: Connection closed by remote host
Connection closed by 20.205.243.166 port 22
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

### 首先查询ip的归属，看看是不是被中间人了

```bash
20.205.243.166
AS8075
MICROSOFT-CORP-MSN-AS-BLOCK
Microsoft Azure
```

发现并不是。

### 再修改github的ssh端口

~/.ssh/config:

```bash
Host github.com
Port 443
User git
HostName ssh.github.com
```

通了...应该是移动封掉了22端口。

`为什么会有人用移动这种弱智宽带...`