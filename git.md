[Git-log]:https://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2

### 创建一个本地库
	git init
	git config --global user.name [USERNAME]	//将全局用户名设置为USERNAME
	git config --global user.email [EMAIL_PATH]	//将全局用户邮箱设为EMAIL_PATH
	git config user.name [USERNAME]	//将当前项目用户名设置为USERNAME
	git config user.email [EMAIL_PATH]	//将当前项目用户名设置为EMAIL_PATH

---
### 查看版本修改
	git status	//可以查看修改了那些文件
	git diff [FILE_PATH]	//具体查看FILE_PATH修改内容
	git diff -staged	//查看工作区与暂存区的修改

---
### 提交代码
	git add .	//添加全部修改的文件到暂存区
	git add [FILE_PATH]	//添加修改的FILE_PATH到暂存区
	git reset HEAD [FILE_PATH] | . //从暂存区移除FILE_PATH | .
	git rm [FILE_PATH]	//从当前暂存区移除FILE_PATH
	git rm --cached [FILE_PATH]	//从当前暂存区移除FILE_PATH, 但本地保留
	git commit -m '提交说明'	//提交修改到当前branch
	git commit --amend	//整合到上次提交
	git rebase -i [START_POINT] [END_POINT]	//弹出交互界面让用户编辑在START_POINT到END_POINT范围内的合并操作

---
### 查看日志
	git log	//查看提交的历史记录, 按Q退出log
	git log --preety=oneline	//精简显示信息
	git log --preety=format: "%h -- %an, %ar: %s"	//显示简短hash -- 作者, 修订时间: 提交说明

>更多log参数 参照[Git-log][Git-log]

---
### 版本回退
	git reset --hard HEAD^	//回退相当于HEAD指针指向某一个提交节点, ^代表上一个节点，^^代表上上个
	git reset --hard HEAD~100	//回退到100个版本前
	git reset --hard 3628164	//3628164是commit id, 如果知道某次提交的commit id, 可以直接reset
	git reflog	//查看所有命令操作, 可以用来查看commit id

---
### 撤销修改
	git checkout -- [FILE_PATH]	//从工作区撤销修改(原理就是恢复到最近版本库的数据), --必须有

---
### 远端
	ssh-keygen -t rsa -C '[EMAIL_PATH]'	//创建SSH key, 公共key在id_rsa.pub文件中, 添加到GIT库SSH keys中
	git remote add [NAME] [URL]	//添加远程仓库URL本地命名为NAME
	git remote -v	//查看添加的远程仓库
	git remote rm [NAME]	//移除远程仓库NAME
	git remote rename [NAME] [NEW NAME]	//重命名远程仓库
	git remote show [NAME]	//查看远程仓库信息
	git pull --rebase [NAME] [BRANCH]	//将远程库NAME的文件拉取的本地BRANCH分支上(在云端新建库时,
		可能会出现readme.md文件,由于本地库不存在, 需要先拉取)
	git push -u [NAME] [BRANCH]	//将当前分支BRANCH推送到远程库NAME, -u可以把本地BRANCH分支与远端BRANCH分支关联起来

---
### 分支
	git checkout -b [BRANCH]	//创建并切换到一个新的分支BRANCH, 相当于 git branch [BRANCH] git checkout [BRANCH]
	git branch	//查看当前分支
	git branch -a	//查看远端分支
	git checkout [BRANCH]	//切换到BRANCH分支
	git merge [BRANCH]	//合并BRANCH到当前分支
	git branch -d [BRANCH]	//删除BRANCH分支
	git merge --no-ff -m '[DESC]' [BRANCH]	//关闭fast forward模式合并分支, --no-ff指令关闭了ff, -m合并时提交描述DESC
	git branch -D [BRANCH]	//未合并的分支BRANCH无法用-d删除, -D可以强制删除
	git checkout -b [BRANCH] origin/[BRANCH]	//在本地建立与远端向对应的分支BRANCH
	git branch --track [BRANCH] origin/[BRANCH]	//建立本地分支与远端的关联

---
### 储藏
	git stash	//存储当前变更
	git stash list	//查看当前储藏
	git stash apply	//应用最新的储藏
	git stash@{[NUM]} apply	//应用当前NUM前的储藏
	git stash apply --index	//应用储藏, 并更新暂存区
	git stash drop stash@{NUM}	//移除储藏
	git stash pop	//应用最新储藏并将其移除
	git stash show -p stash@{NUM} | git apply -R	//取消应用储藏NUM

---
### 标签
	git tag [NAME]	//对最新的提交打标签NAME
	git tag [NAME] [COMMIT_ID]	//对提交COMMIT_ID打标签NAME
	git tag	//查看标签
	git show [NAME]	//查看NAME标签信息
	git tag -a [NAME] -m '[DESC]' [COMMIT_ID]	//-a指定标签名, -m指定说明文字
	git tag -s [NAME] -m '[DESC]' [COMMIT_ID]	//用PGP私钥签名一个标签, 如果没有找到GnuPG, 或没有GPG密钥对, 会报错
	git tag -d [NAME]	//删除一个标签
	git push origin [NAME]	//向远端推送一个标签
	git push origin --tags	//向远端推送所有本地定义标签
	git push origin :refs/tags/[NAME]	//删除远端标签, 必须先删除本地标签

---
### _.gitignore_
	git add -f [FILE_PATH]	//会强制添加
	git check-ignore -v [FILE_PATH]	//查看那条规则对FILE_PATH约束
