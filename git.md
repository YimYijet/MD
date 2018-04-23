###创建一个本地库
	git init 

---
###查看版本修改
	git status	//可以查看修改了那些文件
	git diff 某个文件	//具体查看某个文件修改内容

---
###提交代码
	git add .	//添加全部修改的文件到暂存区
	git add 某个文件	//添加修改的某个文件到暂存区
	git commit -m '提交说明'	//提交修改到当前branch

---
###查看日志
	git log	//查看提交的历史记录
	git log --preety=oneline	//精简显示信息

---
###版本回退
	git reset --hard HEAD^	//回退相当于HEAD指针指向某一个提交节点, ^代表上一个节点，^^代表上上个
	git reset --hard HEAD~100	//回退到100个版本前
	git reset --hard 3628164	//3628164是commit id, 如果知道某次提交的commit id, 可以直接reset


 