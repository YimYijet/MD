### MongoDB
> *   配置文件mongodb.conf

	dbpath=D:\MongoDB\data #数据库路径
	logpath=D:\MongoDB\logs\mongodb.log #日志输出文件路径
	logappend=true #错误日志采用追加模式，配置这个选项后mongodb的日志会追加到现有的日志文件，而不是从新创建一个新文件
	journal=true #启用日志文件，默认启用
	quiet=true #这个选项可以过滤掉一些无用的日志信息，若需要调试使用请设置为false
	port=27017 #端口号 默认为27017
> *   启动

	mongod --config 