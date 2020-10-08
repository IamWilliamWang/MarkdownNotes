* <font size="4">`git init`</font>
git初始化

* <font size="4">`git add .`</font>
监控工作区的状态树，使用它会把工作时的**所有变化**提交到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件

* <font size="4">`git add -u`</font>
仅监控**已经被add的**文件（即tracked file），他会将被修改的文件提交到暂存区。

* <font size="4">`git add -a`</font>
是上面两个功能的合集

* <font size="4">`git diff`</font>
工作区与暂存区或上一次提交进行比较

* <font size="4">`git diff --cached`</font>
比较暂存区与本地库中最近一次提交的内容

* <font size="4">`git status -s`</font>
精简的方式显示文件状态

* <font size="4">`git commit`</font>
提交代码

* <font size="4">`git commit -m 'comment'`</font>
提交代码且注释为"comment"

* <font size="4">`git commit --amend --date="Thu Aug 9 23:04:57 2018 -0700"`</font>
重写最新提交的时间

* <font size="4">`git branch (branch name)`</font>
创建分支

* <font size="4">`git branch -d`</font>
删除分支

* <font size="4">`git branch -r`</font>
查看remote的所有分支

* <font size="4">`git branch -a`</font>
查看本地和remote的所有分支（包括上游项目）

* <font size="4">`git checkout`</font>
切换分支

* <font size="4">`git checkout -b (branch name)`</font>
等价于branch+checkout

* <font size="4">`git checkout -b (branch name) (remote branch name)`</font>
将远程的分支下载后命名为branch name，然后checkout。注意远端的分支和git branch -r里的一致

* <font size="4">`git checkout -t (remote branch name)`</font>
上面的简化版，branch name使用与远端一致的名称

* <font size="4">`git merge -m '' commit名`</font>
合并分支

* <font size="4">`git merge --no-ff`</font>
强制禁用快进模式。[详解网页](https://segmentfault.com/q/1010000002477106)

* <font size="4">`git archive -o filename.zip 版本号`</font>
打包到指定版本的所有代码

* <font size="4">`git log`</font>
输出日志

* <font size="4">`git log --oneline`</font>
简明输出日志，一次commit只显示一行

* <font size="4">`git log --oneline --graph`</font>
附加，显示各个分支之间的关系

* <font size="4">`git show`</font>
显示每次commit的diff之处

* <font size="4">`git show (commit_id)`</font>
显示单次不同之处

* <font size="4">`git tag v1.0`</font>
给HEAD加注释

* <font size="4">`git tag -a v1.0 85fc7e7 -m '1.0版本'`</font>
给版本加带注释的标签

* <font size="4">`git fetch origin master`</font>
github上获取最新版代码，可使用git log -p master..origin/master查看不同之处

* <font size="4">`git pull`</font>
等价于fetch+merge

* <font size="4">`git push`</font>
推送

* <font size="4">`git push origin :分支名`</font>
删除github该分支（要在Settings中修改Default branch为master）

* <font size="4">`git push -u origin master`</font>
设置远程端为origin，将master推入origin

* <font size="4">`git push origin :[tag]`</font>
在github上删除tag

* <font size="4">`git push --amend`</font>
修改最后一次commit的文字

* <font size="4">`git config --system receive.denyNonFastForwards true`</font>
禁止强制更新

* <font size="4">`git config --system receive.denyDeletes true`</font>
规避上述制约可通过删除分支并push新引用。所以加上这一行禁止删除分支和标签

* <font size="4">`touch`</font>
创建空的文本文件

* <font size="4">`git rm`</font>
删除文件

* <font size="4">`git reset --hard <commit_id>`</font>
删除工作空间的改动代码，撤销commit，撤销git add .

* <font size="4">`git reset --soft HEAD^`</font>
不删除工作空间改动代码，撤销commit，不撤销git add .

* <font size="4">`git reset --soft HEAD~2`</font>
不删除工作空间改动代码，撤销两次commit，不撤销git add .

* <font size="4">`git reset --mixed HEAD^`</font>
不删除工作空间改动代码，撤销commit，并且撤销git add .

* <font size="4">`git remote add origin https://github.com/IamWilliamWang/RegexEngine.git`</font>
设置远程git仓库

* <font size="4">`git remote rm origin`</font>
远程仓库置空

* <font size="4">`git remote -v`</font>
查看remote仓库的所有地址（包含上游项目）

* <font size="4">`git remote add upstream *.git`</font>
添加remote仓库的上游项目，比如该项目是从另一个项目fork而来

* 强制将指针指向远程仓库的HEAD
<font size="4">
```git
git reset --hard origin/master
git pull
```
</font>

* <font size="4">`git pull origin master --allow-unrelated-histories`</font>
忽略历史的不同，覆盖本地

* <font size="4">`git stash `</font>
可用来暂存当前正在进行的工作

* <font size="4">`git stash pop `</font>
从Git栈中读取最近一次保存的内容

* <font size="4">`git stash list `</font>
显示Git栈内的所有备份

* <font size="4">`git stash clear `</font>
清空Git栈

* <font size="4">`git stash apply stash@{1} `</font>
可以将你指定版本号为stash@{1}的工作取出来

* <font size="4">`git filter-branch --index-filter 'git rm --cached --ignore-unmatch bin/*.docx' 8f1bf21..HEAD`</font>
从8f1bf21下一个版本开始到HEAD，删除git历史中所有bin/\*.docx文件（慎用，操作不可逆）

* <font size="4">`git for-each-ref --format='delete %(refname)' refs/original | git update-ref --stdin`</font>
移除本地仓库中指向旧提交的剩余refs

* 合并上游项目的commits
```
git remote add upstream *.git
git fetch upstream
git merge upstream/master
git push origin master
```

* 应用.gitignore
<font size="4">
```
git rm -r --cached .       //清空缓存
git add .                  //重新提交
git commit -m ""           //暂存本地
git push                   //推送远端
```
</font>