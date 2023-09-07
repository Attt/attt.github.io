---
title: "通过cf zero trust tunnel访问windows server内网服务"
date: 2023-09-01T09:28:53+08:00
searchHidden: false
tags: ["cloud flare", "zero trust tunnel", "NAT traversal", "windows server"]
draft: false
---

## 配置
### 控制台创建tunnel
![create tunnel](/images/ipv6_only_nas/scrshot01.png)

### 下载cloudflared
[https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.msi](https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.msi)

### 安装服务
```powershell
cloudflared.exe service install {token}
```
这一步会通过Registry Editor安装windows服务，安装后：

![service](/images/ipv6_only_nas/scrshot02.png)

稍等一段时间控制台会显示：

![status](/images/ipv6_only_nas/scrshot03.png)

### 保证连通性

确保从server上能够访问：

![network](/images/ipv6_only_nas/scrshot04.png)

### 修改服务参数

定位到注册表`HKEY_LOCAL_MACHINE > SYSTEM > CurrentControlSet > Services > cloudflared`, 修改`ImagePath`的value:

```cmd
:: 默认
"C:\Program Files (x86)\cloudflared\cloudflared.exe" tunnel run --token {token}
```

*参数参考[tunnel arguments](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/install-and-setup/tunnel-guide/local/local-management/arguments/)*，例：
```cmd
:: quic协议/info logging/logging to file/10 times retries
"C:\Program Files (x86)\cloudflared\cloudflared.exe" tunnel --protocol quic --logfile "C:\Program Files (x86)\cloudflared\info.log" --loglevel info --retries 10 run --token {token}
```

## rdp服务

### 在客户端下载cloudflared
同上

### 配置public hostname

![create public hostname](/images/ipv6_only_nas/scrshot05.png)

### 创建通道

假设rdp端口为默认`3389`，在客户端执行：

```cmd
cloudflared access rdp --hostname rdp.example.com --url rdp://localhost:3389
```

然后连接`localhost:3389`

## web服务

### 配置public hostname

可以设置`host`header，配合内网反向代理分流：

![create public hostname1](/images/ipv6_only_nas/scrshot06.png)

--- 

***[参考]***

- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/install-and-setup/tunnel-guide/remote/remote-management/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/install-and-setup/tunnel-guide/remote/remote-management/)
- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/install-and-setup/ports-and-ips/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/install-and-setup/ports-and-ips/)
- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/install-and-setup/tunnel-guide/local/local-management/arguments/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/install-and-setup/tunnel-guide/local/local-management/arguments/)
- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/use-cases/rdp/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/use-cases/rdp/)


