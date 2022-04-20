---
title: Arch linux Setup
date: 2022-04-19 11:23:08
tags:
---
> https://wiki.archlinux.org/

## 基本系统安装
### 前置准备
- 检查网络
  ```bash 
  ip link
  ```
- 更新系统时间
  ```bash
  timedatectl set-ntp true
  ```
### 准备磁盘
- 分区
  - 查看当前磁盘信息
  ```bash
  fdisk -l
  ```
  - 进入分区
  ```bash
  fdisk /dev/the_disk_to_be_partitioned（要被分区的磁盘）
  ```
  uefi参考：
  | 挂载点 | 分区 | 分区类型 | 大小 |
  | -- | -- | -- | -- |
  | /mnt/boot/efi| /dev/efi_system_partition | EFI系统分区 | 300M |
  | \[swap\]| /dev/swap_partition/ |Linux swap (交换空间) | > 512M (2G if physical ram > 4G) |
  | /mnt| /dev/root_partition | Linux x86-64 根目录 (/) | 剩余空间 |

  - 格式化分区
  ```bash
   mkfs.ext4 /dev/root_partition（根分区）
   mkswap /dev/swap_partition（交换空间分区）
   mkfs.fat -F 32 /dev/efi_system_partition
  ```
  - 挂载分区&启用交换空间
  ```bash
  mount /dev/root_partition（根分区） /mnt
  mount /dev/efi_system_partition /mnt/boot/efi
  swapon /dev/swap_partition（交换空间分区）
  ```
### 安装系统
  - 设置cn源
    
    第一种方法
      ```bash
      #!/bin/bash
      curl 'https://archlinux.org/mirrorlist/?country=CN&prototcol=http&protocol=https&ip_version=4' > /etc/pacman.d/mirrorlist && \
      echo /etc/pacman.d/mirrorlist && \
      sudo vim /etc/pacman.d/mirrorlist && \
      echo 'pacman mirror list updated'
      ```
      然后将`/etc/pacman.d/mirrorlist`中注释去掉
        
    第二种方法

    在`/etc/pacman.d/mirrorlist`中搜索CN，放到最前面，推荐aliyun的源

  - 安装系统&软件包
  ```bash
  pacstrap /mnt base base-devel linux linux-firmware vim vi sudo git
  ```
## 系统配置
### 系统引导
  - 配置fstab
  ```bash
  genfstab -U /mnt >> /mnt/etc/fstab
  ```
  - 安装grub
  ```bash
  pacman -S efibootmgr grub
  # uefi
  grub-install --target=x86_64-efi --efi-directory={挂载点,如/mnt/boot/efi} --bootloader-id=GRUB
  # bios(非GPT分区无需其他空间，但GPT分区需要有一块1M的BIOS boot分区)
  grub-install --target=i386-pc /dev/sdX
  ```
  - 配置grub.cfg
  ```bash
  grub-mkconfig -o /boot/grub/grub.cfg
  ```
### 系统初始化设置

  - 切换到已经安装的系统
  ```bash
  arch-chroot /mnt
  ```
  - 设置时区
  ```bash
  ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
  hwclock --systohc
  ```
  - 本地化
  ```bash
  locale-gen
  ```
  创建`/etc/locale.conf`,设置`LANG=en_US.UTF-8`
  - hostname
  创建`/etc/hostname`,写入主机名
  - root密码
  ```bash
  passwd
  ```
  - 安装网络工具
  
  *networkmanager已经不使用dhcpcd, 因此需要安装dhclient
  ```bash
  pacman -S iw wpa_supplicant dialog dhcpcd dhclient netctl networkmanager network-manager-applet gnome-keyring
  # 设置网络开机启动
  systemctl enable NetworkManager.service
  ```
  - 新增用户
  ```bash
  useradd -m -G users,audio,lp,optical,storage,video,wheel,power -s /bin/bash 普通用户名
  passwd 普通用户名
  ```
  运行`visudo`, 去掉`%wheel ALL=(ALL) ALL`注释

**到这里基本已经完成最小系统的配置和安装，在继续之前，改启动项然后reboot**

## DE/WM

  - 安装xorg
  ```bash
  sudo pacman -S xorg xorg-xinit
  ```
  - 安装字体
  ```bash
  sudo pacman -S terminus-font 
  ```
  - 安装DE
    - Mate
      - 安装mate
      ```bash
      sudo pacman -S mate mate-extra
      ```
      - 配置xinitrc
      ```bash
      cp /etc/X11/xinit/xinitrc ~/.xinitrc
      ```
      把最后的启动换成`exec mate-session`
    - XFCE
      - 安装xfce
      ```bash
      sudo pacman -S xfce xfce4-goodies
      ```
      - 配置xinitrc
      ```bash
      cp /etc/X11/xinit/xinitrc ~/.xinitrc
      ```
      把最后的启动换成`exec startxfce4`

  - 安装WM
    - lightdm
    ```bash
      sudo pacman -S lightdm-gtk-greeter
    ```

## 其他
### 支持hyper-v增强模式
  ```bash
  git clone https://www.github.com/Microsoft/linux-vm-tools
  cd linux-vm-tools/arch
  # 要先开代理
  ./make...
  ./install...
  ```
  **如果无法通过增强模式连接**

  [arch wiki [loginctl or systemctl --user not working]](https://wiki.archlinux.org/title/Xrdp#loginctl_or_systemctl_--user_not_working)
  
  Try commenting out all the references to `systemd-home` in `/etc/pam.d/system-auth`, see [github issue](https://github.com/neutrinolabs/xrdp/issues/1684)

### alias
```bash
alias s='sudo'
alias ls='ls --color=auto'
alias la='ls -a --color=auto'
alias ll='ls -l --color=auto'
alias lla='ls -l -a --color=auto'
alias pac='pacman -S'
alias pacyu='pacman -Syu'
alias spac='sudo pacman -S'
alias spacyu='sudo pacman -Syu'
alias v='vim'
alias sv='sudo vim'

```
### rsync备份
- 安装rsync
```bash
sudo pacman -S rsync
```
- backup.sh
```bash
#!/bin/bash
echo '==> start to backup... type device:' && \
	read device && \
	echo '==> type mount dir:' && \
	read mount_dir && \
	echo '==> umount $device...'; \
	umount $device;\
	echo '==> umount /mnt/$mount_dir...'; \
	umount /mnt/$mount_dir; 
	echo '==> remove /mnt/$mount_dir...'; \
	rm -rf /mnt/$mount_dir && \
	echo '==> create directory /mnt/$mount_dir...'; \
	mkdir /mnt/$mount_dir; \
	echo '==> mount $device to /mnt/$mount_dir...'; \
	mount $device /mnt/$mount_dir; \
	echo '==> create direcotry /mnt/$mount_dir/system_bakcup...'; \
	mkdir /mnt/$mount_dir/system_backup;
	echo '==> start to sync files to /mnt/$mount_dir/system_backup...'; \
	rsync --archive --one-file-system --hard-links \
	--acls --xattrs --sparse \
	--human-readable --numeric-ids --delete --delete-excluded \
	--itemize-changes --verbose --progress \
	--exclude='*~' \
	--exclude='$device' \
	--exclude='/mnt' \
	--exclude='/dev' \
	--exclude='/var/tmp' \
	--exclude='/tmp' \
	/ /mnt/$mount_dir/system_backup && \
echo '==> finished.'
```