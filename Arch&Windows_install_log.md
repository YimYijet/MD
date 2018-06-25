Arch Windows10 双系统安装日志
---

### 分区
	// 分区大小	硬盘大小： 1TB
	/esp 	512M	// EFI 引导分区
	/msr 	128M	// 自动分配
	C:	 	60G		// Windows 系统盘
	/恢复 	786M	// Windows 自动分配 系统恢复分区
	/	 	10G		// Linux 根目录
	swap 	8G		// Linux 交换分区
	/boot	300M	// Linux 引导分区
	/usr	20G		// Linux 用户应用安装
	/home	20G		// Linux 用户主页
	/var	15G		// Linux 可变数据
	D:		200G	// Windows Linux共享工作区
	E:		600G	// Windows happy区
		
### 安装
> 1.   winPE 进入系统，划分esp分区，512M。划分msr分区，作为windows保留分区
> 2.   安装Windows10 系统，windows自动化分oem分区
> 3.   引导进入Arch linux安装

---
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
	# echo LANG=en_US.UTF-8 > /etc/local.conf	// 设置语言，务必设置为en，字体问题可能导致图形界面乱码（小方格）
	# timedatectl status	// 查看系统时间，主要看RTC时间
	# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime	// 设置时区
	# hwclock --systohc
	# nano /etc/mkinitcpio.conf	// /usr分区是独立分区，修改配置文件重启后才能进入系统
	>    把 MODULES=() 改为 MODULES=(ahci btrfs)    // 开启ACHI模式，优化问题，可以不改
	>    在 HOOKS 结尾添加 usr shutdown   
	>    去掉 COMPRESSION="xz" 前的#    // xz压缩效率更高
	# pacman -S btrfs-progs	// 安装btrfs 文件系统环境，安装完成后会重新生成img文件

	# pacman -S dialog wpa_supplicant // 安装联网
	# pacman -S grub efibootmgr os-prober
							
	

	
			

