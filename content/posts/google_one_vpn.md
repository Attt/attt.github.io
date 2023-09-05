---
title: "Hyper-V搭Google One VPN局域网代理"
date: 2023-09-05T20:29:47+08:00
searchHidden: false
tags: ["vpn", "proxy", "Google One", "Google One VPN"]
draft: false
---

## prerequirements
- 带VPN的Google One账号
- Google One可使用区域的梯子
- Hyper-V
- 软路由或者带梯子的路由器

## 安装&设置
### 在Hyper-V中部署[android-x86](https://www.android-x86.org)

- 始终自动启动:
![auto boot](/images/google_one_vpn/scrshot02.png)

- 跟随宿主机强制停止:
![auto stop](/images/google_one_vpn/scrshot03.png)

### 在androidx86中安装Google One, Every Proxy

- 代理方式
![every proxy](/images/google_one_vpn/scrshot04.png)

- 自动启动
![auto start](/images/google_one_vpn/scrshot04-1.png)

### 在androidx86所在网关（路由器）配置TCP规则, 目的是让Google One VPN地区检测通过

**Google One VPN会在初次连接/每消耗10G流量时检测当前的地区是否合法**, 总之`cloud.cupronickel.goog`这个规则很重要, 并且如果不想消耗机场流量UDP流量最好直连。

```yaml
# clash
rules:
  - DOMAIN-KEYWORD,cloud.cupronickel.goog,🇯🇵 日本
  - DOMAIN-KEYWORD,-pa.googleapis.com,🇯🇵 日本
  - DOMAIN-KEYWORD,g-tun,🇯🇵 日本
  - DOMAIN-KEYWORD,gstatic,🇯🇵 日本
```

### 添加到局域网客户端的代理配置
```javascript
// clash for windows mixin in javascript
module.exports.parse = ({ content, name, url }, { yaml, axios, notify }) => {
  // content.rules.unshift("DOMAIN-KEYWORD,bilibili,DIRECT"); // domain keyword
  // content.rules.unshift("PROCESS-NAME,qbittorrent.exe,DIRECT"); // process name
  content.rules.unshift("GEOIP,CN,DIRECT"); // geoip

  // add google one proxy
  googleOneProxies = []
  
  content.proxies.push({
    "name": "Google One",
    "type": "socks5", // every proxy 协议
    "port": "1080", // every proxy 端口
    "server": "192.168.2.231" // androidx86局域网地址
  })
  googleOneProxies.push("Google One")

  content['proxy-groups'].push({
      'name': 'Google One G',
      'type': 'fallback',
      'proxies': googleOneProxies,
      'interval': 300,
      'url': 'http://cp.cloudflare.com/generate_204'
  })
  content['proxy-groups'][0].proxies.unshift("Google One G");

  return content;
}
```

## 检查

### （日本）延迟
![cfw config](/images/google_one_vpn/scrshot05.png)

### 出口检测
![ip test](/images/google_one_vpn/scrshot06.png)

## 速度测试

速度的话，湾湾的还行，小日子的延迟高些

![speed test](/images/google_one_vpn/scrshot07.png)

---

***[备注]***

1. Google One VPN可用区域
![available districts](/images/google_one_vpn/scrshot01.png)
[使用Google One VPN 提高在线安全性- Android](https://support.google.com/googleone/answer/7582172?hl=zh-Hans&co=GENIE.Platform%3DAndroid)

2. 正常情况下只代理TCP连接，那么Google One VPN流量不会走机场。具体是因为连接时先通过443的TCP（HTTPs upper）去检测地区，但实际会通过2153的UDP连接VPN服务器。（VPN服务器能直连）

3. PC端和IOS端连接VPN服务器是走443的UDP，下行流量会被墙掐掉。表现为不停发送连接信息，但是无法成功确认连接。

---

***[参考]***

- [https://www.android-x86.org/installhowto.html](https://www.android-x86.org/installhowto.html)
- [使用Google One VPN 提高在线安全性- Android](https://support.google.com/googleone/answer/7582172?hl=zh-Hans&co=GENIE.Platform%3DAndroid)
- [https://ip.skk.moe/](https://ip.skk.moe/)
- [https://speed.cloudflare.com/](https://speed.cloudflare.com/)
- [Google ONE VPN 折腾记录 - v2ex](https://v2ex.com/t/910836?p=1)