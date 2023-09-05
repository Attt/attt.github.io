---
title: "Hyper-V搭Google One VPN局域网代理"
date: 2023-09-05T20:29:47+08:00
searchHidden: false
tags: ["vpn", "proxy", "Google One", "Google One VPN"]
draft: false
---

### prerequirements
- 带VPN福利的Google One

![available districts](/images/google_one_vpn/scrshot01.png)

- Google One可使用区域的梯子
- Hyper-V虚拟机环境
- 软路由或者带梯子的路由器

### 方法
- 在Hyper-V中部署[android-x86](https://www.android-x86.org)

配置始终自动启动:
![auto boot](/images/google_one_vpn/scrshot02.png)

配置强制停止:
![auto stop](/images/google_one_vpn/scrshot03.png)

- 在androidx86中安装Google One, Every Proxy

![every proxy](/images/google_one_vpn/scrshot04.png)

配置自动启动
![auto start](/images/google_one_vpn/scrshot04-1.png)

- 在路由器配置规则, 目的是让Google One VPN地区检测通过

**Google One VPN会在初次连接/每消耗10G流量时检测当前的地区是否合法**

```yaml
# clash
rules:
  - DOMAIN-KEYWORD,cloud.cupronickel.goog,🇯🇵 日本
  - DOMAIN-KEYWORD,-pa.googleapis.com,🇯🇵 日本
  - DOMAIN-KEYWORD,g-tun,🇯🇵 日本
  - DOMAIN-KEYWORD,gstatic,🇯🇵 日本
```

- 启动Google One VPN, Every Proxy

- 在局域网客户端的代理中配置
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

### Check
![cfw config](/images/google_one_vpn/scrshot05.png)

![ip test](/images/google_one_vpn/scrshot06.png)

速度的话，湾湾的还行，小日子的延迟高些

![speed test](/images/google_one_vpn/scrshot07.png)
---

**[参考]*

- [https://www.android-x86.org/installhowto.html](https://www.android-x86.org/installhowto.html)
- [使用Google One VPN 提高在线安全性- Android](https://support.google.com/googleone/answer/7582172?hl=zh-Hans&co=GENIE.Platform%3DAndroid)
- [https://ip.skk.moe/](https://ip.skk.moe/)
- [https://speed.cloudflare.com/](https://speed.cloudflare.com/)