MagicBook ![archlinux logo](./archlinux-logo-dark.png) 安装日志
---


### GPT分区
path/type|name|size|fs|start|end|flags
:-:|:-:|:-:|:-:|:-:|:-:|:-:
/boot|boot|1G|fat32|2048s|1050623s|boot,esp
swap|swap|2G|swap|1050624s|5244927s|-
/|root|10G|btrfs|5244928s|100%|-

---
### 系统安装
> *   SSD 4K对齐分区

	# lsblk -l					// 显示硬盘及分区

	# parted					// 进入parted分区
	#(parted) print -l				// 显示当前分区
	#(parted) select /dev/sda
	#(parted) mkpart primary fat32 $start $end	// 划分esp分区
	#(parted) set 1 boot on				// 设置esp默认引导
	#(parted) mkpart primary $start $end		// 划分剩余分区
	#(parted) name 1 boot				// 给分区命名（非必要）
	#(parted) q
	# fdisk -lu					// 查看4k对齐，start除以8是否整除

> *   格式化文件系统

	# mkfs.vfat -F 32 /dev/sda1	// 格式化esp分区为fat32文件系统
	# mkswap /dev/sda2		// 创建交换分区
	# swapon /dev/sda2		// 激活交换分区
	# mkfs.btrfs -f /dev/sdaX 	// -f 强制格式化

> *   挂载分区

	# mount /dev/sda3 /mnt		// 挂载root分区到/mnt
	# cd /mnt
	# mkdir -v boot
	# ls
	# mount /dev/sda1 /mnt/boot

> *   安装前准备

	# nano /etc/pacman.d/mirrorlist		// 打开pacman镜像列表，将China源放在最上面，可以不改，但改了以后下载会快
	# wifi-menu				// 链接wifi
	# ping www.baidu.com			// 测试网络是否连通

> *   系统安装

	# pacstrap -i /mnt base base-devel linux linux-firmware	// 安装基础系统以及常用开发工具
	# genfstab -U /mnt >> /mnt/etc/fstab 	// 生成fstab文件，定义分区及文件系统
	# nano /mnt/etc/fstab			// 将分区atime参数设置为noatime，桌面用户优化，阻止读写数据时产生记录，将esp分区移到最上方

---
### 系统配置
> *   系统编码

	# arch-chroot /mnt				// 切换根路径
	# nano /etc/locale.gen				// 修改编码，去掉en_US.UTF-8，zh_CN.UTF-8，zh_TW.UTF-8前的#
	# locale-gen					// 重新生成编码表
	# echo LANG=en_US.UTF-8 > /etc/locale.conf	// 设置语言，务必设置为en，字体问题可能导致图形界面乱码（小方格）

> *   系统时间

	# timedatectl status						// 查看系统时间，主要看RTC时间
	# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime	// 设置时区
	# hwclock --systohc

> *   用户

	# passwd 						//直接回车设置root密码
	# useradd -m -g users -G wheel -s /bin/bash $username 	// 添加用户设置组名，用户名
	# passwd $username					// 为用户设置密码

> *   修改内存初始化的脚本

	# nano /etc/mkinitcpio.conf

	按如下设置：
	1.  把 MODULES=() 改为 MODULES=(amdgpu ahci btrfs)    	// 开启AHCI模式，优化问题，可以不改
	2.  在 HOOKS 结尾添加 btrfs
	3.  去掉 COMPRESSION="xz" 前的#    		// xz压缩效率更高

	# mkinitcpio -p linux				// 更改mkinitcpio配置后，需要手动重新生成镜像（但下一步安装btrfs-progs时会自动更新.img，故非必用）

> *   安装btrfs用户空间工具

	# pacman -S btrfs-progs		// 安装btrfs 文件系统环境，安装完成后会重新生成img文件

> *   安装必要驱动&服务

	# pacman -S nano		// nano文本编辑
	# pacman -S xf86-input-synaptics		// 触摸板驱动
	# pacman -S xf86-video-amdgpu			// amd显卡驱动
	# pacman -S mesa				// openGL驱动，提供DRI和3D加速
	# pacman -S vulkan-radeon		// vulkan驱动
	# pacman -S dialog wpa_supplicant		// 安装联网软件
	# pacman -s networkmanager			// 安装图形网络管理器
	# systemctl enable NetworkManager.service	// 开机启动网络管理
	# pacman -S tlp tlp-rdw				// 安装tlp电源管理器
	# systemctl enable tlp.service

> *   安装图形界面

	# pacman -S xorg		// 安装图形驱动
	# pacman -S plasma		// 安装kde的plasma桌面
	# pacman -S sddm		// 安装显示管理器，用于启动桌面
	# systemctl enable sddm.service	// 开机自启动服务

> *   systemd-boot引导

	# bootctl install		// /boot分区挂载在esp分区直接安装
	# nano /boot/loader/loader.conf

	如下设置：
	default arch
	console-mode max
	auto-firmware 0
	timeout 0
	editor no

	# nano /boot/loader/entries/arch.conf

	如下设置：
	title         Arch Linux
	linux         /vmlinuz-linux
	initrd        /initramfs-linux.img
	options       root=/dev/sda3 rw quiet splash idle=nomwait amdgpu.noretry=0 iommu=soft rcu_nocbs=0-7

	# exit
	# reboot

---
### 后续系统设置
> *   配置sudoers
> *   Ctrl + Alt + F2进入新的tty

	# root 					// 管理员进入，输入密码
	# pacman -Qs sudo			// 如果安装了base-devel元件组，可以看到sudo
	# pacman -S sudo 			// 没有sudo就安装一个
	# nano /etc/sudoers

	如下设置：
	root ALL=(ALL) ALL
	$username ALL=(ALL) NOPASSWD: ALL   	// 给普通用户su权限，并且不需要密码验证

	# pacman -S konsole			// 安装终端工具，KDE应用

> *   配置pacman
> *   Ctrl + Alt + F1切换到图形界面
> *   Alt + Space唤出搜索，输入konsole

	$ sudo pacman -S visual-studio-code-bin libdbusmenu-glib	// vscode
	$ sudo code /etc/pacman.conf			// 修改pacman配置文件

	末尾添加两行：
	[archlinuxcn]
	Server = https://cdn.repo.archlinuxcn.org/$arch

	$ sudo pacman -Sy archlinuxcn-keyring		// 安装archlinuxcn-keyring包以导入GPG key
	$ sudo pacman -S archlinuxcn-mirrorlist-git	// 安装archlinuxcn镜像列表

	$ sudo code /etc/pacman.d/mirrorlist		// 修改镜像源

[镜像地址](https://www.archlinux.org/mirrorlist/)

> *   安装中文字体&输入法

	$ sudo pacman -S wqy-microhei				// 安装文泉驿微米黑字体
	$ sudo pacman -S fcitx fcitx-im fcitx-lilydjwg-git kcm-fcitx	// 安装小企鹅输入法
	$ code ~/.xprofile					// 配置.xprofile, 图形界面启动读取配置
	$ code ~/.xinitrc					// 配置.xinitrc, 非图形界面启动读取配置

	如下设置：
	export GTK_IM_MODULE=fcitx
	export QT_IM_MODULE=fcitx
	export XMODIFIERS="@im=fcitx"

> *   修改fstab文件
> *   使用tempfs减少tmp和log读写ssd，ssd优化

	$ sudo code /etc/fstab

	末尾添加：
	tmpfs	/tmp			tmpfs	defaults,noatime,mode=1777	0 0
	tmpfs	/var/spool		tmpfs	defaults,noatime,mode=1777	0 0
	tmpfs	/var/log		tmpfs	defaults,noatime,mode=1777	0 0
	tmpfs	/var/cache/pacman/pkg	tmpfs	defaults,noatime,mode=1777	0 0
	tmpfs	/home/$username/.cache	tmpfs	defaults,noatime,mode=1777	0 0

> *   SSD启用TRIM功能

	$ hdparm -I /dev/sda | grep TRIM
	$ sudo code /etc/fstab

	在物理分区noatime后添加 discard 如：
	/dev/sda1  /       ext4   defaults,noatime,discard   0  1

> *   交换内存

	$ sudo code /etc/sysctl.d/99-sysctl.conf		// 减少内存交换

	如下设置：
	vm.swappiness=10

> *   内核快捷键

	$ sudo code /etc/sysctl.d/99-sysctl.conf		// 系统强制重启，Ctrl + Alt + SysRq + r, e, i, s, u, b

	如下设置：
	kernel.sysrq = 1

[死机解决](https://blog.csdn.net/openswc/article/details/9105071)

> *   openssh

	$ yay -S openssh
	$ sudo code /etc/ssh/sshd_config

	添加如下：
	AllowUsers    $username

	$ systemctl enable sshd.service

> *   配置zsh

	$ echo $SHELL			// 查看当前shell
	$ sudo pacman -S zsh		// 安装zsh
	$ sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"	// 安装 oh-my-zsh
	$ sudo pacman -S neofetch	// 装逼神器
	$ code ~/.zshrc

	修改如下：
	ZSH_THEME="bullet-train"

	plugins=(
		git
		# 命令快速跳转目录
		z
		# 在命令行中使用搜索引擎进行搜索
		web-search
		# nodejs 版本控制
		nvm
		# npm 及 npm 部分命令别称
		npm
		# node 及 node-docs 命令
		node
		# go 及 go 部分命令别称
		golang
		# 输入正确会绿色高亮显示
		zsh-syntax-highlighting
		# 给出建议的命令
		zsh-autosuggestions
		# 在终端里打开当前项目的远程仓库地址
		git-open
	)

	alias hosts='sudo wget https://raw.githubusercontent.com/googlehosts/hosts/master/hosts-files/hosts -O /etc/hosts'

	$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting		// 安装zsh-syntax-highlighting
	$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions			// 安装zsh-autosuggestions
	$ git clone https://github.com/paulirish/git-open.git $ZSH_CUSTOM/plugins/git-open								// 安装git-open

	$ source .zshrc	// 载入配置

[bullet-train](https://github.com/caiogondim/bullet-train.zsh)</br>
[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)</br>
[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md)</br>
[git-open](https://github.com/paulirish/git-open)

---
### 常用软件

	$ sudo pacman -S yay			// AUR包管理器
	$ yay -S git				// 安装git
	$ yay -S dolphin dolphin-plugins	// dolphin文件管理器
	$ yay -S unrar zip unzip ark		// 压缩及解压缩工具
	$ yay -S bomi				// bomi 播放器
	$ yay -S okular 			// 文档阅读器
	$ yay -S kget wget			// 下载器
	$ yay -S gwenview			// 图片查看
	$ yay -S netease-cloud-music		// 网易云音乐
	$ yay -S fcitx-sogoupinyin		// 搜狗拼音
	$ yay -S latte-dock			// 轻量级dock
	$ yay -S cmake extra-cmake-modules	// cmake

> *   nvm

	$ wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
	$ code ~/.zshrc

	末尾添加：
	export NVM_DIR="$HOME/.nvm"
	[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
	[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

[nvm](https://github.com/creationix/nvm)

---
### 美化
部件：
>	[Window AppMenu Applet](https://www.pling.com/p/1274975)</br>
	[Window Buttons Applet](https://www.pling.com/p/1272871/)</br>

图标：[Papirus](https://store.kde.org/p/1166289/)</br>
应用风格：Breezemite</br>
桌面主题：Macbreeze Shadowless
