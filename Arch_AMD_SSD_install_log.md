MagicBook Archlinux安装日志
---

### GPT分区
path/type|name|size|fs|start|end|flags
:-:|:-:|:-:|:-:|:-:|:-:|:-:
/boot|boot|1G|fat32|2048s|2099199s|boot,esp
swap|swap|2G|swap|2099200s|6293503s|-
/|root|10G|btrfs|6293504s|27265023s|-
/var|var|2G|btrfs|27265024s|31459327s|-
/usr|usr|30G|btrfs|31459328s|94373887s|-
/home|home|remain|btrfs|94373888s|100%|-

---
### 系统安装
> *   SSD 4K对齐分区

	# lsblk -l	// 显示硬盘及分区
	
	# parted	// 进入parted分区
	#(parted) print -l	// 显示当前分区
	#(parted) select /dev/sda
	#(parted) mkpart primary fat32 $start $end	// 划分esp分区
	#(parted) set 1 boot on	// 设置esp默认引导
	#(parted) mkpart primary $start $end	// 划分剩余分区
	#(parted) name 1 boot	// 给分区命名（非必要）
	#(parted) q
	# fdisk -lu	// 查看4k对齐，start除以8是否整除

> *   格式化文件系统

	# mkswap /dev/sda2	// 创建交换分区
	# swapon /dev/sda2	// 激活交换分区
	# mkfs.btrfs -f /dev/sdaX // -f 强制格式化
	
> *   挂载分区

	# mount /dev/sda3 /mnt	// 挂载root分区到/mnt
	# cd /mnt
	# mkdir -v boot usr home var
	# ls
	# mount /dev/sda1 /mnt/boot
	# mount /dev/sda4 /mnt/var
	# mount /dev/sda5 /mnt/usr
	# mount /dev/sda6 /mnt/home

> *   安装前准备

	# nano /etc/pacman.d/mirrorlist	// 打开pacman镜像列表，将China源放在最上面，可以不改，但改了以后下载会快
	# wifi-menu	// 链接wifi
	# ping www.baidu.com	// 测试网络是否连通

> *   系统安装

	# pacstrap -i /mnt base base-devel	// 安装基础系统以及常用开发工具
	# genfstab -U /mnt >> /mnt/etc/fstab // 生成fstab文件，定义分区及文件系统
	# nano /mnt/etc/fstab	// 将分区atime参数设置为noatime，桌面用户优化，阻止读写数据时产生记录，将esp分区移到最上方

---
### 系统配置
> *   系统编码

	# arch-chroot /mnt	// 切换根路径
	# nano /etc/locale.gen	// 修改编码，去掉en_US.UTF-8，zh_CN.UTF-8，zh_TW.UTF-8前的#
	# locale-gen	// 重新生成编码表
	# echo LANG=en_US.UTF-8 > /etc/locale.conf	// 设置语言，务必设置为en，字体问题可能导致图形界面乱码（小方格）

> *   系统时间

	# timedatectl status	// 查看系统时间，主要看RTC时间
	# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime	// 设置时区
	# hwclock --systohc

> *   用户

	# passwd //直接回车设置root密码
	# useradd -m -g users -G wheel -s /bin/bash $username // 添加用户设置组名，用户名
	# passwd $username	// 为用户设置密码

> *   修改内存初始化的脚本

	# nano /etc/mkinitcpio.conf

	按如下设置：
	1.  把 MODULES=() 改为 MODULES=(ahci btrfs)    // 开启AHCI模式，优化问题，可以不改
	2.  在 HOOKS 结尾添加 usr
	3.  去掉 COMPRESSION="xz" 前的#    // xz压缩效率更高
	
	# mkinitcpio -p linux	// 更改mkinitcpio配置后，需要手动重新生成镜像（但下一步安装btrfs-progs时会自动更新.img，故非必用）

> *   安装btrfs用户空间工具

	# pacman -S btrfs-progs	// 安装btrfs 文件系统环境，安装完成后会重新生成img文件

> *   安装必要驱动&服务

	# pacman -S xf86-input-synaptics	// 触摸板驱动
	# pacman -S xf86-video-ati	// amd显卡驱动
	# pacman -S mesa	// openGL驱动，提供DRI和3D加速
	# pacman -S dialog wpa_supplicant	// 安装联网软件
	# pacman -s networkmanager	// 安装图形网络管理器
	# systemctl enable NetworkManager.service	// 开机启动网络管理
	# pacman -S tlp tlp-rdw	// 安装tlp电源管理器
	# systemctl enable tlp.service
	# systemctl enable tlp-sleep.service

> *   安装图形界面
	
	# pacman -S xorg	// 安装图形驱动
	# pacman -S plasma	// 安装kde的plasma桌面
	# pacman -S sddm	// 安装显示管理器，用于启动桌面
	# systemctl enable sddm.service	// 开机自启动服务

> *   systemd-boot引导

	# bootctl install	// /boot分区挂载在esp分区直接安装
	# nano /boot/loader/loader.conf
	
	如下设置： 
	default arch
	timeout 0
	editor 0

	# nano /boot/loader/entries/arch.conf

	如下设置：
	title         Arch Linux
	linux         /vmlinuz-linux
	initrd        /initramfs-linux.img
	options       root=/dev/sda3 rw

	# exit
	# reboot

---
### 后续系统设置
> *   配置sudoers
> *   Ctrl + Alt + F2进入新的tty

	# root // 管理员进入，输入密码
	# pacman -Qs sudo	// 如果安装了base-devel元件组，可以看到sudo
	# pacman -S sudo // 没有sudo就安装一个
	# nano /etc/sudoers
	
	如下设置：
	root ALL=(ALL) ALL
	$username ALL=(ALL) NOPASSWD: ALL   // 给普通用户su权限，并且不需要密码验证

	# pacman -S konsole	// 安装终端工具，KDE应用

> *   配置pacman
> *   Ctrl + Alt + F1切换到图形界面
> *   Alt + Space唤出搜索，输入konsole

	$ sudo pacman -S code	// vscode
	$ sudo code /etc/pacman.conf	// 修改pacman配置文件
	
	末尾添加两行：
	[archlinuxcn]
	Server = https://cdn.repo.archlinuxcn.org/$arch
	
	$ sudo pacman -Sy archlinuxcn-keyring	// 安装archlinuxcn-keyring包以导入GPG key
	$ sudo pacman -S archlinuxcn-mirrorlist-git	// 安装archlinuxcn镜像列表
	
	$ sudo code /etc/pacman.d/mirrorlist	// 修改镜像源

[镜像地址](https://www.archlinux.org/mirrorlist/)

> *   安装中文字体&输入法

	$ sudo pacman -S wqy-microhei	// 安装文泉驿微米黑字体
	$ sudo pacman -S fcitx fcitx-im fcitx-configtool	// 安装小企鹅输入法
	$ sudo code ~/.xprofile	// 配置.xprofile, 图形界面启动读取配置
	$ sudo code ~/.xinitrc	// 配置.xinitrc, 非图形界面启动读取配置

	如下设置：
	export GTK_IM_MODULE=fcitx
	export QT_IM_MODULE=fcitx
	export XMODIFIERS="@im=fcitx"

> *   修改fstab文件
> *   使用tempfs减少tmp和log读写ssd，ssd优化

	$ sudo code /etc/fstab

	末尾添加：
	tmpfs	/tmp	tmpfs	defaults,noatime,mode=1777	0 0
	tmpfs	/var/spool	tmpfs	defaults,noatime,mode=1777	0 0
	tmpfs	/var/log	tmpfs	defaults,noatime,mode=1777	0 0
	tmpfs	/var/cache/pacman/pkg	tmpfs	defaults,noatime,mode=1777	0 0
	tmpfs	/home/$username/.cache	tmpfs	defaults,noatime,mode=1777	0 0

> *   SSD启用TRIM功能 

	$ hdparm -I /dev/sda | grep TRIM
	$ sudo code /etc/fstab

	在物理分区noatime后添加 discard 如：
	/dev/sda1  /       ext4   defaults,noatime,discard   0  1


> *   配置zsh

	$ echo $SHELL	// 查看当前shell
	$ sudo pacman -S zsh	// 安装zsh
	$ sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"	// 安装 oh-my-zsh
	$ sudo pacman -S neofetch	// 装逼神器
	$ sudo code ~/.zshrc

	末尾添加：
	autoload -U compinit
	compinit

---
### 常用软件

	$ sudo pacman -S yay	// AUR包管理器
	$ yay -S git	// 安装git
	$ yay -S chromium
	$ yay -S dolphin dolphin-plugins	// dolphin文件管理器
	$ yay -S unrar rar zip unzip ark	// 压缩及解压缩工具
	$ yay -S dragon	// dragon 播放器
	$ yay -S okular // 文档阅读器
	$ yay -S kget wget	// 下载器
	$ yay -S gwenview	// 图片查看
	$ yay -S netease-cloud-music	// 网易云音乐
	$ yay -S fcitx-sogoupinyin	// 搜狗拼音
	$ yay -S latte-dock	// dock

---
### 美化

菜单：UMenu</br>
图标：[Papirus](https://store.kde.org/p/1166289/)</br>
应用风格：Breezemite</br>
桌面主题：Aex

---
### conky及配置

	$ yay -S conky
	$ code ~/.conkyrc

	修改如下：
	
	#Setting
	background yes
	alignment top_right
	gap_y 40
	cpu_avg_samples 2
	net_avg_samples 2
	minimum_size 300 5
	maximum_width 500
	no_buffers yes
	double_buffer yes
	out_to_console no
	own_window yes
	own_window_transparent yes
	own_window_type override
	update_interval 2
	use_xft yes
	xftalpha 0.8
	xftfont DejaVu Sans:size=12
	default_color grey
	draw_shades no
	draw_borders no
	draw_graph_borders no
	
	TEXT
	${color}${font DejaVu Sans:size=40}S${font DejaVu Sans:size=20}YSTEM
	${font}${voffset -10}${color}HOSTNAME${alignr}${color white}$nodename 
	${color}KERNEL${alignr}${color white}$kernel
	${color}UPTIME${alignr}${color white}$uptime 
	${color}TIME${alignr}${color white}${font}${time %H:%M} ${time %y/%m/%d}
	${color}UPDATE${alignr}${color white}${execpi 3600 checkupdates | wc -l} packages
	${color}${hr 2}
	${color}${font DejaVu Sans:size=40}C${font DejaVu Sans:size=20}PU\
	${font}${voffset -20}${alignr}${cpugraph cpu0 20,150}
	${color}FREQ${alignr}${color white}${freq_g}GHz
	${color}TEMP${alignr}${color white}${acpitemp}°C
	${color}LOAD${alignr}${color white}${loadavg}
	${color}USE${alignr}${color white}${cpu cpu0}%
	${color}${cpubar cpu0 6}
	${color}${font DejaVu Sans:size=40}M${font DejaVu Sans:size=20}EM\
	${font}${voffset -20}${alignr}${memgraph 20,150}
	${color}LOAD${alignr}${color white}$mem / $memmax
	${color}SWAP${alignr}${color white}$swap / $swapmax
	${color}USE${alignr}${color white}$memperc%
	${color}${membar 6}
	${color}Highest_use${alignc -40}CPU%${alignr}MEM%
	${top name 1}${alignc -30}${top cpu 1}${alignr}${top mem 1}
	${top name 2}${alignc -30}${top cpu 2}${alignr}${top mem 2}
	${top name 3}${alignc -30}${top cpu 3}${alignr}${top mem 3}
	${color}${hr 2}
	${color}${font DejaVu Sans:size=40}N${font DejaVu Sans:size=20}ET\
	${font}${voffset -20}${alignr}${downspeedgraph ppp0 20,100 grey grey}
	${color}Down${alignr}${color white}${downspeedf ppp0} k/s
	${color}Up${alignr}${color white}${upspeedf ppp0} k/s
	${color}${hr 2}
	${color}${font DejaVu Sans:size=40}D${font DejaVu Sans:size=20}ISK\
	${font}${voffset -20}${alignr}${diskiograph sda 20,150}
	${color}READ${alignr}${color white}${diskio_read sda}
	${color}WRITE${alignr}${color white}${diskio_write sda}
	${color}/ ${fs_used /} / ${fs_size /}${alignr}${color white}${fs_used_perc /}%
	${color}${fs_bar 6 /}
	${color}/home ${fs_used /home} / ${fs_size /home}${alignr}${color white}${fs_used_perc /home}%
	${color}${fs_bar 6 /home}
