---
title: "换盘后修复windows server中混合布局的storage pool"
date: 2024-01-08T10:50:54+08:00
searchHidden: false
tags: ["windows server", "storage pool", "repair", "manual"]
draft: false
---

书接上回: [在windows server中创建混合布局的storage pool](/posts/windows_storage_pool_plan/)

这次为了鼓捣J4125，挪用了win server上128G的sandisk，在原盘位更换上海鲜市场买的1T*洋垃圾*M600。

sandisk在存储池中的分配为：10G simple + 100G mirror ssd tier，其中simple是和j.zao组成的逻辑下载盘，拔出来的时候里面是空的，作为ssd tier的那部分也不用管，非simple都有修复能力。

# 修复过程

1. 删除逻辑卷(volume)

删除逻辑卷必须通过powershell操作，图形操作会报错。

![scrshot02](/images/windows_storage_pool_repair/scr02.png)

1. 删除simple类型的虚拟磁盘(virtual disk)
2. 删除失联的物理磁盘(physical disk)
3. 修复其他报⚠️的虚拟磁盘

修复前要把物理盘切为'auto'模式

![scrshot03](/images/windows_storage_pool_repair/scr03.png)

开始修复：

![scrshot01](/images/windows_storage_pool_repair/scr01.png)

powershell查看进度

```powershell
Get-StorageJob
```

4. 创建新的simple盘

创建前最好还是把相关物理盘切回'manual'模式

---

***[参考]***

- [https://learn.microsoft.com/zh-cn/powershell/module/storage/set-volume?view=windowsserver2022-ps](https://learn.microsoft.com/zh-cn/powershell/module/storage/set-volume?view=windowsserver2022-ps)