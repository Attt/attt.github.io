---
title: "windows下通过7z打包备份文件到Google drive"
date: 2023-09-06T22:32:14+08:00
searchHidden: false
tags: ["backup", "batch", "Google Drive", "7z"]
draft: false
---
## Google drive客户端设置
设置要备份的目标目录为`可离线使用`，这样变动会实时同步，相当于本地挂载
![google drive config 1](/images/win_bat_backup_to_gcloud/scrshot01.png)

![google drive config 2](/images/win_bat_backup_to_gcloud/scrshot02.png)

## 脚本
### backup.bat
```bat
@echo off
:: set variables
set "work_dir=F:"
set "cloud_drive_backup_dir=G:\我的云端硬盘\Nas\utils"
set "files_dir=%work_dir%\utils"
set "archive_date=%date:~3,4%%date:~8,2%%date:~11,2%"
set "passwd=111222"

:: write new backup
echo "start backup ... compress %files_dir% to %work_dir%\archive_%archive_date%.7z"

:: copy files (snapshot)
xcopy "%files_dir%" "%files_dir%_snapshot" /s/e/i/y/q

:: compress
start /wait "" "C:\Program Files\7-Zip\7z.exe" a -xr!*.log -p"%passwd%" -t7z -mx4 "%work_dir%\archive_%archive_date%.7z" "%files_dir%_snapshot"

:: clean snapshot dir
rmdir /s/q "%files_dir%_snapshot"

:: clean older archives
echo clean older archives
del "%cloud_drive_backup_dir%\archive*"

:: move to cloud drive
echo "start move ... %work_dir%\archive_%archive_date%.7z to %cloud_drive_backup_dir%\archive_%archive_date%.7z"
move "%work_dir%\archive_%archive_date%.7z" "%cloud_drive_backup_dir%\archive_%archive_date%.7z"

echo finished!
```

### 解释
脚本首先复制了`files_dir`，等于创建了一个快照，防止文件的handler被占用。然后使用7z对这个快照压缩打包，打包的目的地还是当前工作目录`work_dir`，不直接写到cloud是因为打包过程中可能会产生部分写入。最后将打包完成的文件移动到cloud并且清理中间文件和cloud上的历史备份。



--- 

***[备注]***

```bat
set "archive_date=%date:~3,4%%date:~8,2%%date:~11,2%"
```
1. `%date:~x,y%`
   1.  x: char offset
   2. y: char count
   3. 通过`echo "%date%"`查看日期格式
![echo date](/images/win_bat_backup_to_gcloud/scrshot03.png)

```bat
xcopy "%files_dir%" "%files_dir%_snapshot" /s/e/i/y/q
```
2. `xcopy`
   1. `/i`: 目标文件夹不存在则会创建
   2. `/s`: 递归（非空文件夹）
   3. `/e`: 递归（含空文件夹）
   4. `/s/e`: 通常组合使用

3. `rmdir /s/q "%files_dir%_snapshot"`
   1. `/s`: 递归
   2. `/q`: 静默

```bat
start /wait "" "C:\Program Files\7-Zip\7z.exe" a -xr!*.log -p"%passwd%" -t7z -mx4 "%work_dir%\archive_%archive_date%.7z" "%files_dir%_snapshot"
```
1. `start /wait "" "%command%"`
   1. `start`: 开启一个新的command执行窗口
   2. `/wait`: 等待执行完成后回到当前流程

2. `"%7z_dir%\7z.exe" a -xr!*.log -p"%passwd%" -t7z -mx4 "%work_dir%\archive_%archive_date%.7z" "%files_dir%_snapshot"`
   1. `a`: 追加文件到archive
   2. `-x`: 排除文件
   3. `r`: 递归
   4. `!*.log`: wildcard, 还可以使用`-xr @files.txt`将排除的文件写到`files.txt`（换行分割）
   5. `-mx4`: 压缩率，越高处理越慢但archive体积越小
   6. `-p`: archive密码

---

***[参考]***

- [https://7-zip.opensource.jp/chm/cmdline/index.htm](https://7-zip.opensource.jp/chm/cmdline/index.htm)
- [windows-commands](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands)