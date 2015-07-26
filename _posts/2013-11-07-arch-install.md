---
layout: blog
categories: linux
tags: Archlinux Linux 操作系统
title: 安装 Arch Linux
redirect_from:
  - /linux/arch-install.html
  - /2013/11/07/arch-install/
---

本文介绍如何安装 Arch Linux，一个轻量级、简单的 Linux 发行版。

相关链接  
[Beginner's Guide](https://wiki.archlinux.org/index.php/Beginners'_Guide)  
[汉化](https://wiki.archlinux.org/index.php/Arch_Linux_Localization_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

## 制作镜像并启动

在arch官方或者bjtu下载到镜像（X86和64）是同一镜像。然后刻录安装盘：

* linux:   

    ```bash
    dd if=/path/to/iso of=/dev/sdc  # 确认 sdc 为你的U盘
    ```
* windows:
    1. 下载 dd4dos
    2. `dd if=/path/to/iso of="\\.\G:"   # 确认G盘为你的U盘`


> 另外，启动不了很正常，和U盘有关。 


## 连接网络

* 无线网络

    ```bash
    netcfg(wifi-menu连接无线网）
    cd /etc/network.d
    cp ./examples/wireless_XXX ./    
    vi wireless_XXX  #修改参数
    netcfg wireless_XXX
    ```
* 对于内核没有支持的无线网卡，安装ndiswrapper使用windows驱动  
    1. 安装    `pacman -S ndiswrapper  `
    2. 加载    `ndiswrapper -i winXP_driver.inf(可以从windows目录下或驱动安装包找到) `
    3. 确认    `ndiswrapper -l ` 
    4. 载入模块    `modprobe ndiswrapper`  
    5. 若无线网卡的状态指示灯不亮，重新载入网卡    `cardctl eject && cardctl insert`

> 使用 `iwconfig` 可查看网络设备(wlan0,eth0等)，若没有识别请参照wiki

<!--more-->

## 准备硬盘

1. 磁盘分区：

    ```bash
    cfdisk /dev/sda #至少一个主分区。
    ```
2. 格式化：

    ```bash
    mkfs.ext4 /dev/sda1，mkswap /dev/sda2（之后可用 lsblk /dev/sda 查看分区）
    ```
3. 挂载：

    ```bash
    mount /dev/sda1 /mnt，swapon /dev/sda2
    ```
4. 安装

    ```bash
    /etc/pacmand./mirrorlist    #配置源
    pacstrap -i /mnt base base-devel    #安装
    ```
5. 更新静态文件系统信息

    ```bash
    genfstab -U -p /mnt >> /mnt/etc/fstab
    ```

### 可能出现的问题

* GPGME error  

    > No data：error: failed to update core (invalid or corrupted database (PGP signature))

    在 `/etc/pacman.conf` 的 `[options]` 加入 `SigLevel = Never`

* 使用交换文件

    ```bash
    # 建立swap文件
    fallocate -l 512M /swapfile
    # 或者
    dd if=/dev/zero of=/swapfile bs=1M count=512
    
    # 设置swap
    chmod 600 /swapfile
    mkswap /swapfile
    swapon /swapfile

    # 更新静态文件系统信息 /etc/fstab 中加入：
    /swapfile none swap defaults 0 0
    ```

## 配置系统

1. 进入新系统
        
    更换主目录：`arch-chroot /mnt`
    更新系统：`配置/etc/pacman.d/mirrorlist并pacman -Syu`

2. 设置区域
    * 设置地区：`ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`
    * 设置硬件时钟：`hwclock --systohc --utc`
        * 自动同步时间：`sudo systemctl enable ntpd（需要ntp）`
        * fix windows:   
            * add a DWORD value with hexadecimal value 1 to the registry:
                `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\RealTimeIsUniversal `
            * disable time auto sync

3. 设置语言
    * 设置语言选项：修改 `/etc/locale.gen` 并 `locale-gen`
    * 设置语言：`echo LANG=en_US.UTF-8 > /etc/locale.conf`
    * 设置主机：`echo myhostname > /etc/hostname`
        
4. 配置网络
    * 安装netcfg用到的工具包：`pacman -S wireless_tools wpa_supplicant wpa_actiond ifplugd dialog`
    * 启用自动连接
        * 单一网络：`systemctl enable dhcpcd@eth0.service`
        * 变化网络：自动连接在 `/etc/network.d` 下的网络

            ```bash
            systemctl enable net-auto-wired.service
            systemctl enable net-auto-wireless.service
            ```

5. 设置用户
    * 设置root密码：`passwd`
    * 添加用户：`useradd -m -g users -s /bin/bash harttle`，设置用户密码：`passwd harttle`
    * 删除用户：`userdel -r harttle`

## 重建引导

以grub为例，可选syslinux

* 安装grub：`pacman -S grub-bios`
* 写入主引导：`grub-install --target=i386-pc --recheck /dev/sda`
* 设置grub区域：

    ```bash
    cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
    ```

* 更新启动列表
    * 搜索windows：`pacman -S os-prober`
    * 更新列表：`grub-mkconfig -o /boot/grub/grub.cfg`
* 重启：`exit;umount /mnt{boot,home,};reboot`

> `/etc/default/grub`中添加`GRUB_SAVEDEFAULT="TRUE"`后再次`grub-mkconfig`将会默认选中上次启动的系统。

## 安装工具

* AUR

    ```bash
    # /etc/pacman.conf 中加入
    [archlinuxfr]
    SigLevel = Optional TrustAll
    Server = http://repo.archlinux.fr/$arch
    pacman -Syu yaourt
    ```

* `sudo`

    安装：`pacman -S sudo`
    配置：`/etc/sudoers` 添加 `harttle ALL=(ALL) ALL`，使harttle可以使用sudo

* `bash` 自动补全：`bash-completion`

* `vim`

## 安装图形界面

以KDE为例，可选gnome

1. 安装驱动：`mesa(3D)`,`xf86-video-vesa(Default)`,`xf86-video-nouveau(open nvidia)`,`nouveau-dri`(open nvidia)

1. 安装图形界面的底层协议实现：`pacman -S xorg`

1. 安装kde及其语言包：`pacman -S kde kde-l10n-zh_cn`
    * 最小安装：`kdebase`，`kde-l10n-zh_cn`

1. 设置启动
    * 采用 `startx`：`~/.xinitrc` 加入 `exec ck-launch-session startx`
    * 采用kdm(kde)，可选gdm(gnome)
        * 安装 `kdebase-workspace`，编辑 `~/.xprofile`
        * 设置 kdm 启动：`systemctl enable kdm`

1. kde的gtk支持：安装 `oxygen-gtk2`，`oxygen-gtk3`，`kde-gtk-config`(AUR) 进入系统设置->公共外观行为->应用程序外观->gtk configuration相关设置

1. kde网络管理

    ```bash
    pacman -S networkmanager kdeplasma-applets-networkmanagement
    systemctl enable NetworkManager.service 
    ```

1. 设置登录屏幕主题：`archlinux-themes-kdm(AUR)`，kde systemsettings 中的设置不起作用

1. 在fstab加入开机挂载的分区，需要 `ntfs-3g`(AUR)

    ```bash
    /dev/hda1        /mnt/winC        ntfs-3g iocharset=utf8,umask=022,noatime 0 0
    ```
        
## 汉化

* 设置X默认字符集：在 `~/.bashrc`、`~/.xinitrc` 或 `~/.xprofile` 中加入

    ```bash
    LOCALE=en_US.UTF-8
    export LANG=zh_CN.UTF-8
    export LC_ALL="zh_CN.UTF-8"
    ```

* 安装字体库

    `wqy-bitmapfont`，`wqy-zenhei`，`ttf-arphic-ukai`，`ttf-arphic-uming`，`ttf-fireflysung`，`wqy-microhei`（AUR），`wqy-microhei-lite`（AUR）

* 输入法
    1. 安装 fcitx

        ```bash
        pacman -S fcitx fcitx-gtk2 fcitx-gtk3 fcitx-qt
        ```
    2. 安装配置工具：`fcitx-configtool`(基于gtk3), `fcitx-configtool-gtk2`(基于gtk2), `kcm-fcitx`(基于KDE的KCM)
    3. 设置X的输入法：在 `~/.xinitrc` 或 `~/.xprofile` 中加入：

        ```bash
        export XMODIFIERS=@im=fcitx
        export GTK_IM_MODULE=fcitx
        export QT_IM_MODULE=fcitx
        ```
    4. 设置启动：`cp /etc/xdg/autostart/fcitx-autostart.desktop ~/.config/autostart/`

    > 输入法对不同语言的键盘映射在`~/.config/fcitx/data/punc.mb.<LANG>`，可以手动更改（例如中文中括号）。

* 终端输入法：fbterm

    * 安装
    
        ```bash
        yaourt -S fbterm fcitx-fbterm
        sudo gpasswd -a YOUR_USERNAME video #非根用户运行fbterm
        sudo setcap 'cap_sys_tty_config+ep' /usr/bin/fbterm 或：sudo chmod u+s /usr/bin/fbterm #非根用户可使用键盘快捷方式
        ```
    
    * 配置
    
        ```bash
        # ~/.fbtermrc
        font-names = Consolas（Monaco）,微软雅黑
        font-size=15

        # ~/.bashrc
        if [ "$TERM" = "linux" ]; then  
        alias fbterm='LANG=zh_CN.UTF-8 fbterm'  
        fbterm  
        fi
        ```

## 字体

可在AUR中直接安装 Ubuntu字体、开源字体、Adobe字体，甚至 Windows 字体。

以效果奇佳的Ubuntu字体设置为例：

1. 安装 `cairo-ubuntu` `ttf-dejavu` `ttf-liberation` `wqy-zenhei` `ttf-arphic-ukai` `ttf-arphic-uming` 
2. 在`～/.config/fontconfig/fonts.conf`配置字体:

    ```xml
    <?xml version='1.0'?>
    <!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
    <fontconfig>
     <match target="font">
      <edit name="spacing"> <int>0</int> </edit>
      <edit mode="assign" name="antialias"> <bool>true</bool> </edit>
      <edit mode="assign" name="hinting"> <bool>true</bool> </edit>
      <edit mode="assign" name="hintstyle"> <const>hintslight</const> </edit>
      <edit mode="assign" name="autohint"> <bool>false</bool> </edit>
      <edit mode="assign" name="rh_prefer_bitmaps"> <bool>false</bool> </edit>
      <edit mode="assign" name="rgba"> <const>rgb</const> </edit>
      <edit name="embeddedbitmap"> <bool>false</bool> </edit>
      <edit mode="append" name="lcdfilter"> <const>lcddefault</const> </edit>
     </match>
     <alias>
      <family>serif</family>
      <prefer>
       <family>WenQuanYi Zen Hei</family>
       <family>Bitstream Vera Serif</family>
       <family>DejaVu Serif</family>
      </prefer>
     </alias>
     <alias>
      <family>sans-serif</family>
      <prefer>
       <family>WenQuanYi Zen Hei</family>
       <family>DejaVu Sans</family>
       <family>Bitstream Vera Sans</family>
      </prefer>
     </alias>
     <alias>
      <family>monospace</family>
      <prefer>
       <family>WenQuanYi Zen Hei Mono</family>
       <family>DejaVu Sans Mono</family>
       <family>Bitstream Vera Sans Mono</family>
      </prefer>
     </alias>

     <!-- To substitute some famous Chinese fonts -->
     <match target="pattern">
      <test name="family"> <string>宋体</string> </test>
      <edit mode="assign" name="family"> <string>SimSun</string> </edit>
     </match>
     <match target="pattern">
      <test name="family"> <string>新宋体</string> </test>
      <edit mode="assign" name="family"> <string>SimSun</string> </edit>
     </match>
     <match target="pattern">
      <test name="family"> <string>楷体</string> </test>
      <edit mode="assign" name="family"> <string>KaiTi</string> </edit>
     </match>
     <match target="pattern">
      <test name="family"> <string>楷体_GB2312</string> </test>
      <edit mode="assign" name="family"> <string>KaiTi</string> </edit>
     </match>
     <match target="pattern">
      <test name="family"> <string>黑体</string> </test>
      <edit mode="assign" name="family"> <string>SimHei</string> </edit>
     </match>
     <match target="pattern">
      <test name="family"> <string>微软雅黑</string> </test>
      <edit mode="assign" name="family"> <string>SimHei</string> </edit>
     </match>
     <alias binding="strong">
      <family>SimSun</family>
      <accept> <family>AR PL UMing CN</family> </accept>
     </alias>
     <alias binding="strong">
      <family>KaiTi</family>
      <accept> <family>AR PL UKai CN</family> </accept>
     </alias>
     <alias binding="strong">
      <family>SimHei</family>
      <accept> <family>WenQuanYi Zen Hei</family> </accept>
     </alias>

     <!-- To substitute some English fonts -->
     <alias binding="strong">
      <family>BookAntiqua</family>
      <accept> <family>URW Palladio L</family> </accept>
     </alias>
     <alias binding="strong">
      <family>Georgia</family>
      <accept> <family>Liberation Serif</family> </accept>
     </alias>
     <alias binding="strong">
      <family>Verdana</family>
      <accept> <family>Liberation Sans</family> </accept>
     </alias>
     <alias binding="strong">
      <family>Calibri</family>
      <accept> <family>Liberation Sans</family> </accept>
     </alias>
    </fontconfig>
    ```

> Ubuntu 的`hintstyle`选择`slighthint`较好。有些桌面环境的字体配置模块会在桌面启动时对`~/.config/fontconfig/fonts.conf`进行修改，此时应保持字体模块的设置与此相同。

对于konsole终端字体，可以在`~/.kde4/share/apps/konsole/Shell.profile`配置（设置页面的配置文件)，在页面中只允许更改monospace字体，可以在这里任意修改字体，如：

```bash
#    1                        2    3  4 5  6 7 8 9 10
Font=WenQuanYi Micro Hei Mono,10.5,-1,5,62,0,0,0,0,0
```

该配置的含义没有文档！执行`konsole --list-profile-properties`可以看到字体的类型为`QFont`文档见：http://qt-project.org/doc/qt-4.8/qfont.html 。经测试，第1字段为Family，第2字段为Size，第5字段为Weight，第6字段为Italic(0或1)。

Constant        | Value | Description
---             | ---   | ---
QFont::Light    | 25    |  25
QFont::Normal   | 50    |  50
QFont::DemiBold | 63    |  63
QFont::Bold     | 75    |  75
QFont::Black    | 87    |  87
