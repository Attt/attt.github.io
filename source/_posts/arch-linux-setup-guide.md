---
title: Arch Linux 安装笔记
date: 2022-04-19 11:23:08
categories: 
	- 伸展运动
  - Linux
tags:
  - guide
	- setup
	- Arch Linux
	- Linux
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
  | /mnt/boot/EFI| /dev/efi_system_partition | EFI系统分区 | 300M |
  | \[swap\] **移动设备建议不要**| /dev/swap_partition/ |Linux swap (交换空间) | > 512M (2G if physical ram > 4G) |
  | /mnt| /dev/root_partition | Linux x86-64 根目录 (/) | 剩余空间 |

  - *移动设备的情况下:*
    | 挂载点 | 分区 | 分区类型 | 大小 |
    | -- | -- | -- | -- |
    | *none* | *none* | BIOS启动分区 | 1M |
    | /mnt/boot/EFI| /dev/efi_system_partition | EFI系统分区 | 300M |
    | /mnt| /dev/root_partition | Linux x86-64 根目录 (/) | 剩余空间 |

  - 格式化分区
  ```bash
   mkfs.ext4 /dev/root_partition（根分区）
   # 移动设备关闭日志 mkfs.ext4 -O "^has_journal" /dev/root_partition（根分区）
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
  # 移动设备 可以使用基于BFQ的io调度内核linux-zen，把linux换成linux-zen
  ```
  *参考：[BFQ](https://www.kernel.org/doc/html/latest/block/bfq-iosched.html)*

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
  grub-install --target=x86_64-efi --efi-directory={挂载点,如/mnt/boot/EFI} --bootloader-id=GRUB

  # bios(非GPT分区无需其他空间，但GPT分区需要有一块1M的BIOS boot分区)
  grub-install --target=i386-pc /dev/sdX
  ```
  *移动设备的情况下*

  假设分区顺序是：
    - BIOS
    - EFI
    - root
  要先在这块设备上写入引导信息，运行`gdisk /dev/sdX`

  ```bash
  # gdisk /dev/sdX

  Command (? for help): r
  Recovery/transformation command (? for help): h

  WARNING! Hybrid MBRs are flaky and dangerous! If you decide not to use one,
  just hit the Enter key at the below prompt and your MBR partition table will
  be untouched.

  Type from one to three GPT partition numbers, separated by spaces, to be added to the hybrid MBR, in sequence: 1 2 3
  Place EFI GPT (0xEE) partition first in MBR (good for GRUB)? (Y/N): N

  Creating entry for GPT partition #1 (MBR partition #1)
  Enter an MBR hex code (default EF): 
  Set the bootable flag? (Y/N): N

  Creating entry for GPT partition #2 (MBR partition #2)
  Enter an MBR hex code (default EF): 
  Set the bootable flag? (Y/N): N

  Creating entry for GPT partition #3 (MBR partition #3)
  Enter an MBR hex code (default 83): 
  Set the bootable flag? (Y/N): Y

  Recovery/transformation command (? for help): x
  Expert command (? for help): h
  Expert command (? for help): w

  Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
  PARTITIONS!!

  Do you want to proceed? (Y/N): Y
  ```
  
  > **如果不需要兼容，不用做这一步，混合分区会在设备上存在两个分区表，系统识别这个设备分区的时候具体的行为完全依赖于系统的实现，非常不稳定**

  > **注意：写完hybrid之后，默认的windows中就只认MBR分区，假设分区按上面的例子，且还有一个4号分区，那么由于MBR的限制**
  > **在windows中第四个分区是无法正确识别文件系统的，如果要使用第四个分区且要通过MBR引导，那么这里就要在MBR写入134分区**
  > **ESP分区可以不用写到MBR里**


  ```bash
  pacman -S efibootmgr grub amd-ucode intel-ucode

  # uefi + MBR(BIOS)混合启动
  grub-install --target=x86_64-efi --efi-directory={挂载点,如/mnt/boot/EFI} --removable
   --bootloader-id=GRUB --boot-directory={root挂载点,如/mnt}/boot

  grub-install --target=i386-pc --recheck --boot-directory=/DATA_MOUNTPOINT/boot /dev/sdX #注意这里是整个设备

  # 作为保险
  grub-install --target=i386-pc --recheck --boot-directory=/DATA_MOUNTPOINT/boot /dev/{root的那个分区}
  ```
  - 切换到已经安装的系统
  ```bash
  arch-chroot /mnt
  ```

  - 配置grub.cfg
  ```bash
  grub-mkconfig -o /boot/grub/grub.cfg
  ```
### 系统初始化设置

  - 设置时区
  ```bash
  ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
  hwclock --systohc
  ```
  - 本地化
  ```bash
  locale-gen
  ```

  -- 移动设备最小化磁盘访问,把日志放到内存
  修改`/etc/systemd/journald.conf.d/usbstick.conf`
  写入
  ```bash
  [Journal]
  Storage=volatile
  RuntimeMaxUse=30M
  ```

  - initramfs
  如果是移动设备，将`/etc/mkinitcpio.conf`中`HOOKS`里面的`block`和`keyboard`放到`autodetect`前面，然后
  ```bash
  mkinitcpio -P
  ```

  创建`/etc/locale.conf`,设置`LANG=en_US.UTF-8`
  - hostname
  创建`/etc/hostname`,写入主机名
  - root密码
  ```bash
  passwd
  ```
  - hosts
  向`/etc/hosts`写入
  ```bash
  127.0.0.1 localhost
  ::1 localhost
  127.0.0.1 {hostname}
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

## DE/WM/DM

  - 安装xorg
  ```bash
  sudo pacman -S xorg xorg-xinit
  ## 移动设备要安装
  # xf86-input-synaptics 支持触控板
  # xf86-video-vesa, xf86-video-ati, xf86-video-intel, xf86-video-amdgpu, xf86-video-nouveau and xf86-video-fbdev. 大多数的开源显卡驱动
  #  libeatmydata  可以禁用fsync，比如对于firefox：eatmydata firefox
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
      sudo pacman -S xfce4 xfce4-goodies
      ```
      - 配置xinitrc
      ```bash
      cp /etc/X11/xinit/xinitrc ~/.xinitrc
      ```
      把最后的启动换成`exec startxfce4`

  - 仅安装WM
    
    *(略)*
    
  - 安装DM
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
然后记得在win中用管理员设置
```cmd
Set-VM -VMName <your_vm_name>  -EnhancedSessionTransportType HvSocket
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

### 中文及字体设置
- 语言配置
去掉`/etc/locale.gen`中的`zh_CN.UTF-8 UTF-8`和`en_US.UTF-8 UTF-8`的注释

根据需要在`~/.xinitrc`或者`~/.bashrc`配置

**在`~/.xinitrc`中配置要注意在`exec xxx`之前**
```bash
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
```
- 字体配置
```bash
sudo pacman -S wqy-bitmapfont wqy-microhei noto-fonts noto-fonts-emoji
```
设置`~/.config/fontconfig/fonts.conf`
```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
<fontconfig>

  <!-- 显示器使用的像素排列方式 -->
  <match target="font">
    <edit mode="assign" name="rgba">
      <const>rgb</const>
    </edit>
  </match>

  <!-- 字体微调的程度, 可选 hintnone, hintslight (默认), hintmedium, hintfull. -->
  <!-- 更高的 hinting 等级可使字体更锐利，同时也会损失更多的细节 -->
  <!-- 如果显示器的 DPI 高得不像话 (>=300), 可关闭 hinting, 字体会自然对齐像素 -->
  <match target="font">
    <edit mode="assign" name="hintstyle">
      <const>hintslight</const>
    </edit>
  </match>

  <!-- 抗锯齿. 除非屏幕DPI奇高否则建议开启 -->
  <match target="font">
    <edit mode="assign" name="antialias">
      <bool>true</bool>
    </edit>
  </match>


<!-- 全局默认字体　-->
  <match>
    <edit mode="prepend" name="family">
      <string>Terminus</string>
    </edit>
  </match>

<!-- 全局默认英文字体 -->
  <match>
    <test compare="contains" name="lang">
      <string>en</string>
    </test>
    <edit mode="prepend" name="family">
      <string>Terminus</string>
    </edit>
  </match>

<!-- 全局默认中文字体 -->
  <match>
    <test compare="contains" name="lang">
      <string>zh</string>
    </test>
    <edit mode="prepend" name="family">
      <string>WenQuanYi WenQuanYi Bitmap Song</string>
    </edit>
  </match>

<!-- 默认无衬线字体 -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>sans-serif</string></test>
    <edit name="family" mode="prepend" binding="same">
      <string>Terminus</string>
    </edit>
  </match>

<!-- 默认衬线字体 -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>serif</string>
    </test>
    <edit name="family" mode="prepend" binding="same">
      <string>Terminus</string>
    </edit>
  </match>

<!-- 默认等宽字体 -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>monospace</string>
    </test>
    <edit name="family" mode="prepend" binding="same">
      <string>Terminus</string>
    </edit>
  </match>

  <!-- 字体替换 -->
  <!--SimSun, Microsoft YaHei, SimHei -> WenQuanYi Micro Hei -->
  <match target="pattern">
      <test name="family">
          <string>宋体</string>
      </test>
      <edit name="family" mode="assign" binding="strong">
          <string>WenQuanYi WenQuanYi Bitmap Song</string>
          <!-- <string>Noto Serif CJK SC</string> -->
      </edit>
  </match>
  <match target="pattern">
      <test name="family">
          <string>SimSun</string>
      </test>
      <edit name="family" mode="assign" binding="strong">
          <string>WenQuanYi WenQuanYi Bitmap Song</string>
          <!-- <string>Noto Serif CJK SC</string> -->
      </edit>
  </match>
  <match target="pattern">
      <test qual="any" name="family">
          <string>SimSun-18030</string>
      </test>
      <edit name="family" mode="assign" binding="same">
          <string>WenQuanYi WenQuanYi Bitmap Song</string>
         <!-- <string>Noto Serif CJK SC</string> -->
      </edit>
  </match>

  <match target="pattern">
    <test qual="any" name="family">
        <string>Microsoft YaHei</string>
    </test>
    <edit name="family" mode="assign" binding="same">
          <string>WenQuanYi WenQuanYi Bitmap Song</string>
      </edit>
  </match>

<match target="pattern">
    <test qual="any" name="family">
        <string>SimHei</string>
    </test>
    <edit name="family" mode="assign" binding="same">
          <string>WenQuanYi WenQuanYi Bitmap Song</string>
      </edit>
  </match>

  <match target="pattern">
      <test name="family">
          <string>Times New Roman</string>
      </test>
      <edit name="family" mode="append" binding="strong">
          <string>WenQuanYi WenQuanYi Bitmap Song</string>
      </edit>
  </match>
  <match target="pattern">
      <test name="family">
          <string>Terminus</string>
      </test>
      <edit name="family" mode="append" binding="strong">
          <string>WenQuanYi WenQuanYi Bitmap Song</string>
      </edit>
  </match>


<!-- 备用字体优先顺序 -->
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Terminus</family>
      <family>WenQuanYi WenQuanYi Bitmap Song</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
    </prefer>
  </alias>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Terminus</family>
      <family>WenQuanYi WenQuanYi Bitmap Song</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Terminus</family>
      <family>WenQuanYi WenQuanYi Bitmap Song</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
    </prefer>
  </alias>

</fontconfig>
```
设置字体偏好:`/etc/fonts/conf.d/64-language-selector-prefer.conf`
```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
<fontconfig>  
<alias>
    <family>sans-serif</family>
    <prefer>
      <family>Terminus</family>
      <family>WenQuanYi WenQuanYi Bitmap Song</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
    </prefer>
  </alias>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Terminus</family>
      <family>WenQuanYi WenQuanYi Bitmap Song</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Terminus</family>
      <family>WenQuanYi WenQuanYi Bitmap Song</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
    </prefer>
  </alias>
</fontconfig>


```

### 安装fcitx5
```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-mozc
```

### xfce的声音
```bash
 sudo pacman -S alsa-utils pulseaudio pulseaudio-alsa \
  pavucontrol      #与xfce4 panel里面的插件配合使用,必须要有这个进程
```
xfce4 panel设置里面添加一个pulseaudio plugin的插件

`alsamixer` F6选择声卡,然后把mm的M键去掉,上下箭头调节,auto-mute disable了以后,loopback enable. reboot重启,这时候xfce4的插件就可以调节声音了

### 安装chrome
```bash
#AUR 助手
git clone https://aur.archlinux.org/yay-git.git
cd yay-git
makepkg -si

#yay不要用root
yay -S google-chrome

#升级
yay -Syu
```

### 安装clash

看[这](https://blog.linioi.com/posts/clash-on-arch/)
或者[class for windows](https://github.com/Fndroid/clash_for_windows_pkg/releases)


### 支持ntfs
```bash
sudo pacman -S ntfs-3g
```