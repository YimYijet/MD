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
#### UEFI + GPT
1.   

