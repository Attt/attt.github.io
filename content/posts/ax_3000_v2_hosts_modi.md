---
title: "修改华硕TUF-AX3000 V2的hosts域名解析"
date: 2023-09-11T01:03:08+08:00
tags: ["asus router", "router", "hosts", "domain resolve", "manual"]
draft: false
---
## shell登入路由器

### 新增`dnsmasq.conf.add`文件

```bash
# touch /jffs/configs/dnsmasq.conf.add
cd /jffs/configs
# touch dnsmasq.conf.add
vi dnsmasq.conf.add
```
写入

```bash
addn-hosts=/jffs/configs/custom_hosts
```

### 新增`custom_hosts`文件

```bash
# touch /jffs/configs/custom_hosts
cd /jffs/configs
# touch custom_hosts
vi custom_hosts
```
写入自定义hosts：

```bash
192.168.1.1 local.site
192.168.1.2 local1.site
# ....
```

### 重启`dnsmasq`

```bash
service restart_dnsmasq
```

## 其他

路由器要作为设备DNS或者存在于设备DNS链上。

## 官改固件
[https://fw.koolcenter.com/KoolCenter_ASUS_Official_Mod/TUF-AX3000_V2/](https://fw.koolcenter.com/KoolCenter_ASUS_Official_Mod/TUF-AX3000_V2/)
