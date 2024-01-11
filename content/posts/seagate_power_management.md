---
title: "希捷硬盘PowerChoice(EPC)电源管理"
date: 2024-01-11T12:11:51+08:00
searchHidden: false
tags: ["linux", "seagate", "power management", "PowerChoice", "EPC", "shell", "manual"]
draft: false
---

# 功耗对比

![scr01](/images/seagate_power_management/scr01.png)

# 切换模式

1. 下载工具seachest

link: [https://www.seagate.com/ca/en/support/software/seachest/](https://www.seagate.com/ca/en/support/software/seachest/)


2. 查看当前模式

```bash
./SeaChest_PowerControl -d /dev/sdb --showEPCSettings
```
![scr02](/images/seagate_power_management/scr02.png)

可以看到默认状态`idle_a`和`idle_b`同时启用，优先使用`idle_a`，这个模式下磁盘仍然保持全转速。

3. 修改模式

```bash
# 禁用idle_a
./SeaChest_PowerControl -d /dev/sdb --idle_a disable
# 启用/修改idle_b(⚠️注意这里是毫秒)
./SeaChest_PowerControl -d /dev/sdb --idle_b 60000
```

![scr03](/images/seagate_power_management/scr03.png)


4. 实践

如果在pve中`/dev/sdb`整块作为冷储存设备直通给`VM1`这个虚拟机，可以在pve宿主机设置待机后进入`standby_z`模式，这个模式下磁盘电机也会停转。达到的效果是当冷储存处理专用的VM关机状态时，磁盘最终完全停转，降噪+节能。

![scr05](/images/seagate_power_management/scr05.png)


ps. seagate的EXOS盘都是不支持APM的


![scr04](/images/seagate_power_management/scr04.png)

---
***[参考]***

- [https://www.seagate.com/old-support-files/seachest/SeaChest_Combo_UserGuides.html#file11](https://www.seagate.com/old-support-files/seachest/SeaChest_Combo_UserGuides.html#file11)

- [https://www.seagate.com/www-content/product-content/enterprise-hdd-fam/enterprise-capacity-3-5-hdd/enterprise-capacity-3-5-hdd/en-us/docs/100791104c.pdf](https://www.seagate.com/www-content/product-content/enterprise-hdd-fam/enterprise-capacity-3-5-hdd/enterprise-capacity-3-5-hdd/en-us/docs/100791104c.pdf)

- [https://www.reddit.com/r/DataHoarder/comments/po7fr1/apm_not_supported_by_seagate_exos_drives/](https://www.reddit.com/r/DataHoarder/comments/po7fr1/apm_not_supported_by_seagate_exos_drives/)

- [https://www.cnblogs.com/zphj1987/p/13559673.html](https://www.cnblogs.com/zphj1987/p/13559673.html)