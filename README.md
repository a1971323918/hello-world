1 安装及配置（windows）
官网下载安装Git

开始菜单找到git → git bash（或者在某个文件夹下面 右键 -> git bash here）

配置/修改用户信息
$ git config --global user.name "Your Name" // --global参数表示全局配置，也可以针对不同仓库配置不同用户名和email
$ git config --global user.email "email@example.com"
查看用户信息：
$ git config --global user.name
$ git config --global user.email

2 创建一个Git仓库
$ mkdir learngit // 新建一个文件
$ cd learngit // 进入文件
$ git init // 把文件目录变成git仓库

在learngit下新建文件readme.txt
$ git add readme.txt // 将文件添加到仓库
$ git commit -m "add readme file" // 提交文件

3 版本管理
3.1 提交文件
修改readme.txt文件
$ git status // 查看当前状态
$ git diff // 查看修改的内容
$ git add filename
$ git commit -m "some describe for this commit"

3.2 版本回退
$ git log // 查看日志
$ git log --pretty=oneline // 一条记录显示在一行
$ git reset --hard HEAD^ // 回到上一个提交，HEAD^^回到上上个提交，
$ git reset --hard HEAD^^ // 回到上上个提交
$ git reset --hard 8adc // 回到commit id开头为8adc的版本
$ git reflog // 查看操作记录

4 工作原理
一个git目录可以分为两个部分：

工作区：能看到的文件
版本库：.git文件里，不可见
版本库包含暂存区stage、master分支（包含一个HEAD指针）。
提交文件必须先提交到stage再提交到分支上。
$ git add filename // 将工作区的文件提交到stage  git diff HEAD -- filename // 查看工作区和分支上代码的区别

图片.png
5 撤销修改
工作区：git checkout -- filename // 清理工作区（距离上次add或commit后所做的修改）

stage：
1. git reset HEAD filename // stage → 工作区（把上次add但未提交的stage中的内容打回工作区）
2. 清理工作区

分支：
git reset --soft HEAD^ // HEAD^表示上一个版本，也可以写作HEAD~1；soft表示不删除工作区改动代码，撤销commit，不撤销git add .；harg表示删除工作区改动代码，撤销commit，撤销git add .
git commit --amend // 只修改注释，进入vim编辑器，修改完注释后保存即可

6 删除文件
新建一个文件test → add → commit 此时git分支保存了新文件test，删除文件的几种方式如下：

$ rm test.txt → add → commit
或者在文件夹手动删除test.txt → add / (git rm filename) → commit
$ git rm test.txt → commit
恢复文件的方法同【撤销修改】

7 远程仓库
7.1 添加远程库
添加远程仓库需要两步操作：本地生成SSH Key添加到远程仓库、本地仓库与远程仓库关联。

1.创建SSH Key
$ ssh-keygen -t rsa -C "your@email.com" // 生成一个公钥id_rsa.pub和一个私钥id_rsa
$ cd ~ // 进入用户主目录
$ cd .ssh
$ ls // 列出所有文件
$ cat id_rsa.pub // 查看文件内容
复制内容到 git/github -> Account Setting -> SSH -> 粘贴

2.关联远程仓库
git/github创建仓库
$ git remote add origin https://github.com/example/learngit.git // 本地仓库关联远程仓库
$ git push -u origin master // 推送本地仓库到远程仓库master分支 -u表示第一次推送
$ git push origin master // 后续推送到远程分支

以上失败时：
$ git remote rename oroigin old-origin
$ git remote add origin git @.../project-name.git
$ git push -u origin --all

7.2 克隆远程仓库
git/github新建仓库 → Clone or download → 复制链接（HTTPS or SSH）
$ git clone git@github.com:example/gitskills.git
$ cd gitskills

克隆到本地后的仓库只有master分支，需要新建dev分支并和远程分支关联
$ git checkout -b dev origin/dev
$ git branch --set-upstream-to=origin/dev dev

Git支持多种协议，默认的git://使用ssh，但也可以使用https等其他协议
使用https速度慢，每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用ssh协议而只能用https

如果在git上新建了一个分支，只需在本地运行：
$ git fetch
$ git checkout branchName

8 分支管理
8.1 创建与合并分支
查看分支：git branch
创建分支：git branch <name>
切换分支：git checkout <name>
创建+切换分支：git checkout -b <name>
合并某分支到当前分支：git merge <name>
删除分支：git branch -d <name>
强行删除：git branch -D <name>

分支开发策略：master用来发布新版本 dev用来开发 在dev上新建分支开发，完成后合并到dev分支，再新开分支

8.2 解决冲突
8.2.1 merge冲突
如果两个分支同时修改了一个文件，merge的时候会冲突
$ git merge feature1
在IDE里解决完冲突 → add → commit （此时master就是修改后的，feature1是修改前的）
$ git checkout feature1
$ git merge master // feature1和master同步了，此时不会冲突

合并分支策略
$ git merge dev // Fast forward模式，将master指针指向dev的最新一次commit，此时master和dev的commit id相同
$ git merge --no--ff -m "提交信息" dev // 禁用Fast foward模式，merge时会生成一个新的commit

8.2.2 push冲突
当同时修改了同一个文件时会导致git push失败，必须先git pull → 解决冲突 → git push
git pull如果提示no tracking information说明没有创建本地分支与远程对应分支的链接关系，通过以下方式设置：
$ git branch --set-upstream-to=origin/dev dev // 本地分支dev关联远程分支origin/dev
再git pull

8.3 stash
git stash用于将“修改过的被追踪的文件和暂存的变更”及stage和工作区的修改暂存起来。

stash的范围：
Changes to be committed（stage）
Changes no staged for commit（工作区）

$ git stash // 存档当前的改动并清理工作区和stage
$ git stash list // 罗列所有的存档
$ git stash apply // 读取最近的一次存档
$ git stash apply stash@{0} // 指定读取某次存档
$ git stash drop // 删除存档
$ git stash pop // 读取并删除最近一次存档

8.4 rebase
这部分内容的参考教程：
http://gitbook.liuhui998.com/4_2.html https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%82%A8%E8%97%8F%EF%BC%88Stashing%EF%BC%89

步骤	图示
假设有两个分支：master、experiment	

切换到experiment分支执行rebase
$ git checkout experiment
$ git rebase master
取出experiment分支上的提交C4接到master分支上的最新提交C3后面，将experiment指向C4'。此时experiment分支的进度是最新的，而master分支无变化。
rebase时如果出现了冲突：
打开IDE解决冲突
$ git add .
$ git rebase --continue
如果要终止rebase：
$ git rebase --abort	

切换到master分支执行merge $ git checkout master $ git merge experiment
此时master将指向最新的commit	

9 标签管理
可以给每次commit打一个标签来增加辨识度。

$ git tag v1.0 // 给当前分支的最新提交打标签
$ git tag v0.9 commitid // 给某次commit打标签
$ git tag -a v0.1 -m "version 0.1 released" commitid // 给某次提交创建带有说明的标签 -a指定标签名 -m指定说明文字
$ git tag // 查看所有标签
$ git show v0.9 // 查看某个标签的详细信息
$ git push origin v1.0 // 将某个标签推送到远程
$ git push origin --tags // 将全部本地标签推送到远程
$ git pull origin --tags // 拉取远程标签
$ git tag -d v0.1 // 删除本地某个标签
删除远程标签：
$ git tag -d v0.9
$ git push origin :refs/tags/v0.9

10 其他常用命令
$ git log --graph --pretty=oneline --abbrev-commit // git log --graph查看分支合并图
$ git remote // 查看远程命名
$ git remote -v // 查看远程库信息
$ pwd // 显示当前文件路径
$ ls // 查看该目录下的所有文件
$ cat filename // 查看文件内容
$ vi filename // 进入文件编辑模式


