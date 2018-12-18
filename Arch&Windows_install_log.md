Arch Windows10 双系统安装日志
 ---

### 分区
	// 分区大小	硬盘大小： 1TB
	/esp 	512M	FAT16	// EFI 引导分区
	/msr 	128M	MSR	// Windows 保留分区
	C:	60G	NTFS	// Windows 系统盘
	/oem 	786M	NTFS	// Windows 自动分配 系统恢复分区
	/	10G	BTRFS	// Linux 根目录
	swap 	8G	SWAP	// Linux 交换分区
	/boot	300M	EXT2	// Linux 引导分区
	/usr	20G	BTRFS	// Linux 用户应用安装
	/home	20G	BTRFS	// Linux 用户主页
	/var	15G	BTRFS	// Linux 可变数据
	D:	200G	NTFS	// Windows Linux共享工作区
	E:	600G	NTFS	// Windows happy区

 ---
### 系统安装及配置
> 1.  winPE 进入系统，划分esp分区，512M。划分msr分区，作为windows保留分区
> 2.  安装Windows10 系统，windows自动化分oem分区
> 3.  引导进入Arch linux安装

	# lsblk -l // 显示硬盘及分区

	# parted	// 进入parted分区
	#(parted) unit MB	// 以MB为单位显示
	#(parted) print -l 	// 显示当前分区
	#(parted) select /dev/sda
	#(parted) mkpart boot $startM $endM	// 分配boot分区
	#(parted) unit GB
	#(parted) mkpart primary $startG $endG	// 分配swap分区
	#(parted) mkpart root $startG $endG	// 分配root分区
	#(parted) mkpart var $startG $endG	// 分配var分区
	#(parted) mkpart usr $startG $endG	// 分配usr分区
	#(parted) mkpart home $startG $endG	// 分配home分区
	#(parted) q

	# mkfs.ext2 /dev/sda5	// 格式化boot分区为ext2文件系统
	# mkswap /dev/sda6	//创建交换分区
	# swapon /dev/sda6	//激活swap分区
	# mkfs.btrfs -f /dev/sda7	// -f 强制格式化
	# mkfs.btrfs -f /dev/sda8
	# mkfs.btrfs -f /dev/sda9
	# mkfs.btrfs -f /dev/sda10

	# mount /dev/sda7 /mnt	// 挂载root分区到/mnt
	# cd /mnt
	# mkdir -v boot usr home var
	# ls
	# mount /dev/sda5 /mnt/boot
	# mount /dev/sda8 /mnt/var
	# mount /dev/sda9 /mnt/usr
	# mount /dev/sda10 /mnt/home
	# mkdir -v /mnt/boot/efi	// 创建efi目录用来挂载esp分区
	# mount /dev/sda1 /mnt/boot/efi

	# nano /etc/pacman.d/mirrorlist	// 打开pacman镜像列表，将China源放在最上面，可以不改，但改了以后下载会快
	# wifi-menu	// 链接wifi
	# ping www.baidu.com	// 测试网络是否连通
	# pacstrap -i /mnt base base-devel	// 安装基础系统以及常用开发工具
	# genfstab -U /mnt >> /mnt/etc/fstab //生成fstab文件，定义分区及文件系统
	# nano /mnt/etc/fstab	// 将分区atime参数设置为noatime，桌面用户优化，阻止读写数据时产生记录

	# arch-chroot /mnt	// 切换根路径
	# nano /etc/locale.gen	// 修改编码，去掉en_US.UTF-8，zh_CN.UTF-8，zh_TW.UTF-8前的#
	# locale-gen	// 重新生成编码表
	# echo LANG=en_US.UTF-8 > /etc/locale.conf	// 设置语言，务必设置为en，字体问题可能导致图形界面乱码（小方格）
	# timedatectl status	// 查看系统时间，主要看RTC时间
	# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime	// 设置时区
	# hwclock --systohc
	# passwd //直接回车设置root密码
	# useradd -m -g users -G $groupname -s /bin/bash $username // 添加用户设置组名，用户名
	# passwd $username	// 为用户设置密码
	# nano /etc/mkinitcpio.conf	// /usr分区是独立分区，修改配置文件重启后才能进入系统
	>    把 MODULES=() 改为 MODULES=(ahci btrfs)    // 开启AHCI模式，优化问题，可以不改
	>    在 HOOKS 结尾添加 usr shutdown   
	>    去掉 COMPRESSION="xz" 前的#    // xz压缩效率更高
	# pacman -S btrfs-progs	// 安装btrfs 文件系统环境，安装完成后会重新生成img文件

	# pacman -S dialog wpa_supplicant // 安装联网
	# pacman -S xf86-input-synaptics	// 触摸板驱动
	# pacman -S xf86-video-intel	// intel显卡驱动，默认安装了vesa驱动
	# pacman -S grub efibootmgr os-prober	// grub引导
	# grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub	// grub使用UEFI，挂载esp分区，设置引导id为grub
	# grub-mkconfig -o /boot/grub/grub.cfg	// 生成grub引导文件
	# pacman -S xorg	// 安装图形驱动
	# pacman -S plasma	// 安装KDE的plasma桌面
	# pacman -S sddm	// 安装显示管理器，用于启动桌面
	# systemctl enable sddm.service	// 开机自启动服务
	# pacman -S networkmanager	// 安装图形网络管理器
	# systemctl enable NetworkManager.service	// 开机启动网络管理
	# pacman -S tlp tlp-rdw	// 安装tlp电源管理器
	# systemctl enable tlp.service
	# systemctl enable tlp-sleep.service

	# exit	// 退出chroot
	# reboot	// 重启电脑

> 4.  重启默认引导为Windows，进入winPE，通过引导修改工具禁用Windows Boot Manager

 ---
### 应用安装及配置
> *   Ctrl + Alt + F2进入新的tty

	# root // 管理员进入，输入密码
	# pacman -Qs sudo // 如果安装了base-devel元件组，可以看到sudo
	# pacman -S sudo // 没有sudo就安装一个
	# nano /etc/sudoers
	>    root ALL=(ALL) ALL
	>    $username ALL=(ALL) NOPASSWD: ALL   // 给普通用户su权限，并且不需要密码验证
	# pacman -S konsole	// 安装终端工具，KDE应用

> *   Ctrl + Alt + F1切换到图形界面
> *   Alt + Space唤出搜索，输入konsole

	# sudo pacman -S atom	// github官方编辑器，一个字 好使！
	# sudo atom /etc/pacman.conf	// 修改pacman配置文件
	>    [archlinuxcn]
	>    Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
	>    [core]
	>    Server = https://mirrors.ustc.edu.cn/archlinux/core/os/$arch
	>    [extra]
	>    Server = https://mirrors.ustc.edu.cn/archlinux/extra/os/$arch
	>    [community]
	>    Server = https://mirrors.ustc.edu.cn/archlinux/community/os/$arch
	# sudo pacman -S haveget	// 随机数生成器，用于生成熵，减少pacman秘钥初始化时间
	# sudo systemctl start haveged.service
	# sudo pacman-key --init	// 初始化pacman秘钥
	# sudo pacman-key --populate archlinux	// 验证主密钥
	# sudo pacman-key --refresh-keys	// 更新开发者秘钥
	# sudo pacman -Sy archlinuxcn-keyring	// 安装archlinuxcn GPG key

	# sudo pacman -S fcitx fcitx-im fcitx-configtool	// 安装小企鹅输入法
	# sudo atom ~/.xprofile	// 配置.xprofile, 图形界面启动
	# sudo atom ~/.xinitrc	// 配置.xinitrc, 非图形界面启动
	>    export GTK_IM_MODULE=fcitx
	>    export QT_IM_MODULE=fcitx
	>    export XMODIFIERS="@im=fcitx"
	# sudo pacman -S fcitx-sogoupinyin
	# sudo pacman -S wqy-microhei	// 安装文泉驿微米黑字体
	# sudo pacman -S chromium	// chromuim浏览器
	# sudo pacman -S dolphin	// dolphin文件管理器
	# sudo pacman -S rar unrar ark	// 压缩及解压缩工具
	# sudo pacman -S wget	// 安装wget
	# sudo pacman -S vlc	// vlc播放器
	# sudo pacman -S gwenview	// 图片查看
	# sudo pacman -S netease-cloud-music	// 网易云音乐
	# sudo pacman -S gimp	// 图像处理

 ---
### GRUB设置
> *   由于禁用了Windows Boot Manager，无法进入Windows，需要向grub添加Windows引导选项

	# sudo atom /boot/grub/grub.cfg	// 打开grub配置文件添加
	>    if [ "${grub_platform}" == "efi" ]; then
	>        menuentry "Windows 10" {
	>            insmod part_gpt
	>            insmod fat
	>            insmod search_fs_uuid
	>            insmod chain
	>            search --fs-uuid --set=root $hints_string $fs_uuid
	>            chainloader /EFI/Microsoft/Boot/bootmgfw.efi
	>        }
	>    fi
	# sudo grub-probe --target=fs_uuid /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi		// 获取 $uuid替换上面的的$fs_uuid
	# sudo grub-probe --target=hints_string /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi	// 获取 $hints_string替换

> *   Windows引导已修复，然后是grub美化
> *   安装 [grub主题](https://www.gnome-look.org/browse/cat/109/ord/latest/) ，选个中意的主题 [poly light](https://www.gnome-look.org/p/1176413/) 下载后解压到/boot/efi/EFI/grub/themes/下（保证主题文件夹在esp分区内，否则文件内容无法加载）

	# sudo atom /etc/default/grub
	>    GRUB_THEME="/boot/efi/EFI/grub/themes/poly-light-master/theme.txt"
	>    GRUB_GFXMODE="1920x1080x32"    // 分辨率自己调试
	# sudo grub-mkconfig -o /boot/grub/grub.cfg	// 更新配置文件

	# sudo pacman -S grub-customizer	// 安装grub图形配置工具

> *   进入grub customizer调整引导菜单，保存

 ---
### NTFS自动挂载
> *   进入Windows将剩下硬盘空间划分为NTFS分区（D:，E:）
> *   进入Arch，将D:盘自动挂载到/home/workspace

	# sudo pacman -S ntfs-3g	// 安装ntfs文件系统
	# sudo mount -w /dev/sda11 /home/workspace
	# sudo atom /etc/fstab	// 修改分区，文件系统配置文件
	>    /dev/sda11    /home/workspace    ntfs-3g    rw,users,noatime    0 0

 ---
### Windows Linux时间同步
> *   时间存储方式分为两类，一类为世界同一时间加时区（UTC，GMT），一类为本地时间LocalTime，linux系统使用UTC，Windows系统使用LT，因此会出现切换系统时，时间时大时小
> 1.  Windows修改为UTC时间，关闭Windows时间同步
> 2.  管理员权限打开命令行

	> reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f

> 3.  进入Arch

	# sudo timedatectl set-local-rtc false	// 设置硬件时钟显示方式为UTC
	# sudo timedatectl status

 ---
### 显卡配置
> *   禁用nvidia独显

	# sudo pacman -S bbswitch
	# sudo atom /etc/modules-load.d/bbswitch.conf	// 每次启动加载bbswitch模块
	>    bbswitch
	# sudo atom /etc/modprobe.d/bbswitch.conf	// bbswitch加载参数，默认关闭
	>    options bbswitch load_state=0
	# sudo atom /etc/modprobe.d/nouveau_blacklist.conf	// 关闭组织显卡关闭的占用模块
	>    blacklist nouveau
	>    blacklist nvidiafb
	# sudo atom /usr/lib/systemd/system-shutdown/nvidia_card_enable.sh	// 每次重启都启用显卡，防止windows找不到显卡设备
	>    #!/bin/bash
	>    case "$1" in
	>        reboot)
	>            echo "Enabling NVIDIA GPU"
	>            echo ON > /proc/acpi/bbswitch
	>        ;;
	>        *)
	>    esac
	# sudo chmod +x /usr/lib/systemd/system-shutdown/nvidia_card_enable.sh	// 添加执行权限

 ---
### 主题及美化
> *   
