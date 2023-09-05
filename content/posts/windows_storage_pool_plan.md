---
title: "在windows server中创建混合布局的storage pool"
date: 2023-08-31T15:12:40+08:00
searchHidden: false
tags: ["windows server", "storage pool"]
draft: false
---

### 目标
- SSD分层加速的raid1存放重要文件
- 大容量raid0存放HTPC的媒体文件

### 物理磁盘
|#|容量 | 类型|
|--| -- | -- |
| #1| 128 GB | SSD |
| #2| 240 GB | SSD |
| #3| 1 TB | SSD |
| #4| 8 TB | HDD |
| #5| 8 TB | HDD |
| #6| 10 TB | HDD |

![scrshot](/images/windows_storage_pool_plan/scrshot02.png)

### 目标存储布局
|#| 可用容量 | 虚拟类型| 物理磁盘(组合) |
|--| -- | -- | -- |
| #1| ~120 GB | simple (简单raid0) | #2(130 GB) + #1(18 GB)|
| #2| ～1 TB | layering + mirror (分层镜像raid1) | #6(1 TB) + #5(1 TB) + #2(110 GB) + #1(110 GB)|
| #3| ～22 TB | simple (简单raid0) | #6(9 TB) + #5(7 TB) + #4(8 TB) |

![scrshot](/images/windows_storage_pool_plan/scrshot01.png)

### 实现步骤
**序号为物理磁盘序号*
1. 创建pool
2. 按照Manual方式添加#5到pool中
3. 基于#5创建simple虚拟磁盘，分配预留1 TB空间用于后面的镜像虚拟盘
4. 修改#5的类型为Auto
5. #1，#2，#6按照Auto方式加入pool
6. 用#5的剩余空间和#1，#2，#6创建mirror虚拟磁盘，使用两块SSD做存储分层（这一步所有盘必须是Auto）
7. 按照Manual方式加入剩余的物理盘，并且修改所有的盘为Manual
8. 由于两块SSD大小不同，用剩余的部分创建新的simple虚拟盘
9. 对基于#5创建的simple虚拟磁盘进行扩展，把剩余的盘空间加入（预留大约5 GB存放虚拟盘的记录数据）
10. 新建卷（默认的卷簇或者群集是4096，支持的单卷存储上限很低，目标如果是百T级别可以调整到32K）

![scrshot](/images/windows_storage_pool_plan/scrshot03.png)

### 使用命令
1. 修改pool中的磁盘使用类型
```powershell
Get-PhysicalDisk -SerialNumber "{serial_number}" | Set-PhysicalDisk -Usage "AutoSelect"
Get-PhysicalDisk -SerialNumber "{serial_number}" | Set-PhysicalDisk -Usage "ManualSelect"
```

---------

**[参考]*

- [https://learn.microsoft.com/zh-cn/powershell/module/storage/set-physicaldisk?view=windowsserver2022-ps](https://learn.microsoft.com/zh-cn/powershell/module/storage/set-physicaldisk?view=windowsserver2022-ps)