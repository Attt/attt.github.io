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
sudo pacman -S wqy-microhei noto-fonts-cjk
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
      <family>WenQuanYi WenQuanYi Bitmap Song</family>
      <family>WenQuanYi Micro Hei</family>
      <family>DejaVu Sans</family>
      <family>Roboto</family>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
      <family>Font Awesome 5 Free</family>
    </prefer>
  </alias>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Terminus</family>
      <family>WenQuanYi WenQuanYi Bitmap Song</family>
      <family>DejaVu Serif</family>
      <family>Noto Serif</family>
      <family>Noto Serif CJK SC</family>
      <family>Noto Serif CJK TC</family>
      <family>Noto Serif CJK JP</family>
      <family>Noto Serif CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
      <family>Font Awesome 5 Free</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Terminus</family>
      <family>Hack</family>
      <family>DejaVu Sans Mono</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
      <family>Font Awesome 5 Free</family>
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