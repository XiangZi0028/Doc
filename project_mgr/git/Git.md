# Git

###  工作流程

git的主要工作流程

1.在本地项目文件中做一些修改、删除、添加文件

2.将需要进行版本管理的文件放入到暂存区域；git add file

3.将暂存区域的文件提交到git仓库； git commit 

4.如果像再推到远程仓库中； git push

![63651-20170905201033647-1915833066](E:\ZEngine\ZEngine\doc\63651-20170905201033647-1915833066.png)

![image-20211123173942462](E:\ZEngine\ZEngine\doc\image-20211123173942462.png)

### Git文件操作

版本控制就是对文件的版本控制，要对文件进行修改、提交等操作，首先要知道当前文件的状态，不然可能会提交了现在还不想提交的文件，或者想要提交的文件没有提交上。

①Untracked：未跟踪状态，此文件在文件夹中，但并没有加入到git库，不参与版本控制。通过git add 把状态变为staged

②Unmodfify：文件已经入库，未修改，即版本库中的文件快照内推与文件夹中的完全一致。这种类型文件有两种取出，如果它被修改，则变为Modified状态。如果使用git rm移除版本库中，则成为Untracked状态。

③Modified：文件已修改，仅仅是修改灭有其它的操作。这个文件有2个去处，通过git add可进入暂存stage状态，使用git checkout 则丢弃修改过，返回到unmodify状态，这个checkout即从库中取出文件，覆盖当前的修改！

④Staged：暂存状态，执行git commit则将修改同步到库中，这时库中的文件和本地的变得文件一致，文件为Unmodify状态，执行git reset Head filename取消暂存，文件状态为modified。



![image-20211123131101920](E:\ZEngine\ZEngine\doc\image-20211123131101920.png)

当我们在本地做了修改之后，git add会把我们的修改内容存到暂存区，就是一个记录文件，通过git commit 提交到本地的仓库中，最后在git push到远程仓库

本地空间，工作目录

![image-20211123132443168](E:\ZEngine\ZEngine\doc\image-20211123131915984.png)

index ----> 暂存区

Head----->仓库区



### 配置

所有的配置文件都保存在本地的

--system 系统级别的都在安装目录下的config文件里 Git\etc\config

--global 所有的用户配置文件在user\config

--local 项目的所有配置文件在项目的.git\config

##### git config

#####查看

git config --list #显示当前所有配置

git config --system|--global|--local --list #查看配置

#####添加删除编辑项

git config --system|--global|--local selection.key value #添加配置项

git config --system|--global|--local --unset selection.key #删除配置项

git config -e --system|--global|--local #配置文件的编辑模式

git config --system|--global|--local -e #同上

##### git archive

git archive #生成一个可供发布的压缩包

### 创建本地仓库

##### git init

git init #初始化当前项目，也就是在当前目录中生成初始化一个git项目

git init [newDir]  #新建一个目录，并且将它初始化为git代码库

##### git clone

git clone [url] #下载一个项目和它整个历史代码

### 增删文件-暂存区

##### git add

git add [filename1] [filename2] .....   # 添加文件到暂存区

git add .   # 添加所有文件到暂存区

git add [dir] #添加指定目录到暂存区，包括子目录

git add -p #添加每个变化前，都会要求确认，对于同一个文件的多出变化，可以实现分次提交

##### git rm

git rm [filename1] [filename2] ....  #删除工作区文件，并将这次删除放如暂存区

git rm --cached [file] #停止追踪指定文件，但该文件会保留在工作区

##### git mv

git mv [fileName] [fileRename] #改名文件，并且将这个改名放入到暂存区

### 分支branch

多个分支如果并行执行，就会导致我们的代码不冲突，也就是同时存在多个版本！

#### 查看-删除-新建

##### git branch

git branch #查看所有本地的分支

git branch -r #查看所有远程分支

git branch -a #查看所有分支 远程+本地

git branch branchName #新建一个分支，但是操作依然停留在当前分支

git branch -b branchName #新建一个分支，并且切换到该分支上去

git branch [branch] [commit] #新建一个分支，指向指定的commit

git branch --track [branch] [remote-branch] #新建一个分支，与指定的远程分支建立追踪关系

git branch --set-upstream [branch] [remote-branch] #建立追踪关系，在现有分支与指定分支之间

git branch -d branchName #删除一个分支

git branch -dr branchName   #删除一个远程分支

git push origin --delete [branchNanme] #删除一个远程分支

#### 切换分支

##### git checkout

git checkout [branch] #切换到指定分支

git checkout -  #切换到上一个分支

#### 合并分支

##### git cherry-pick [commit]

git cherry-pick [commit] #选择一个commit合并到当前分支

git merge branch #合并指定分支到当前分支

### merge冲突 

如果同一个文件在不同的分支都被修改了，那么在合并分支的时候都被修改了就会引起冲突

解决办法：修改冲突文件后重新提交

master分支应该非常稳定，用来发布新版本，一般情况下不允许在上面工作，工作一般情况下新建(如：welink)的分支上，工作完成后，比如开发完成之后，要将welin分支合并到master上

### 标签 tag

##### git tag

git tag 查看所有的tag

git show [tag] #查看tag信息

git tag [tag] #新建一个tag在当前commit

git tag [tag] [commit] #新建一个tag在指定commit

git tag -d [tag] #删除本地tag

git push origin ：refs/tags/[tagName]  #删除远端的tag

git push [remote] [tag] #提交指定tag

git push [remote] --tags #提交所有tag

git checkout -b [branch] [tag] #新建一个分支，指向某个tag

### 查看信息

##### git status

git status #显示有变更的文件

##### git log

git log #显示当前分支的版本历史

git log --stat #显示commit历史，以及每次commit发生变更的文件

git log -S [keyword]  #搜索提交历史，更具关键词

git log [tag] HEAD --pretty=fromat:%s #显示某个commit之后的所有变动，每个commit占据一行

git log [tag] HEAD --grep feature #显示某个comiit之后的所有变动，其"提交说明"必须符合搜索条件

git log --follow [file] #显示某个文件的版本历史，包括文件改名

git log -p [file] #显示指定文件相关的每一次diff

git log -5 --pertty --online #显示过去5次提交

##### git shortlog -sn

git shortlog -sn #显示所有提交过的用户，按提交次数排序

##### git whatchanged [file] 

#和git log --follow [file]一样,显示某个文件的版本历史，包括文件改名

##### git blame [file] 

git blame [file] #查看什么人在什么时间修改过这个文件file

##### git diff

git diff #显示暂存区和工作区的差异

git diff --cached [file]  #显示暂存区和上一个commit的差异

git diff HEAD #显示工作区与当前分支最新commit之间的差异

git diff [firstBranch] [secondBranch] #显示两次提交之间的差异

git diff --shrotstat “@{0 day ago}” # 显示今天写了多少行代码

##### git show

git show [commit] #显示某次提交的元数据和内容变化

git show --name-only [commit] #显示某次提交时，发生变化的文件

git show [commit]:[filename] #显示某次提交时，某个文件的内容

##### git reflog

git reflog #显示当前分支最近几次的提交

### 远程同步

##### git fetch [remote]

git fetch [remote] #下载远程仓库中的所有变动

##### git remote

git remote -v #显示所有远程仓库

git remote show [remote] #显示某个远程仓库的信息

git remote add [shortname] [url] #增加一个新的远程仓库并命名

##### git pull

git pull [remote] [branch] #取回远程仓库并和本地分支合并

##### git push

git push [remote] [branch]  #上传指定分支到远程仓库，

git push [remote] --force #强行推送当前分支到远程仓库，即使有冲突

git push [remote] -all #推送所有分支到远程仓库

### 撤销

##### git checkout

git checkout [file] #恢复暂存区的指定文件到工作区

git checkout [commit] [file] #恢复某个commit的指定文件到暂存区和工作区

git checkout .  #恢复暂存区的所有文件到工作区

##### git reset

git reset [file] #重置暂存区的指定文件，与上一次commit保持一致，单工作区不变

git reset --hard #重置暂存区与工作区，与上一次commit保持一致

git reset [commit] #重置当前分支到指定的commit，同时重置暂存区，但工作区不变

git reset --hard [commit] #重置当前分支的HEAD为指定commit，同时重置暂存区和工作区与指定commit一致

git reset --keep [commit] #重置当前HEAD为指定commit，但是保持暂存区和工作区不变

##### git revert

还不太理解！！！

git revert commit 

#新建一个commit，用来撤销指定commit

#后者的所有变化都将被前者抵消，并且应用到当前分支

##### git stash

git stash 

git stash pop

暂时将未提交的变化移除，稍后再移入

### ssh公钥

当前用户目录有一个.ssh目录

###### ssh-keygen

ssh -keygen -t  rsa  rsa是选择的加密算法(官方推荐算法)

### Git忽略文件

在主目录下添加.gitignore文件 

其中可以使用Linux的通配符，*代表任意多个字符，？代表一个字符，[abc]代表可选字符范围，{string1，string2}代表可选字符串等。

| *.txt     | 忽略所有.txt结尾的文件                                       |
| --------- | ------------------------------------------------------------ |
| !lib.txt  | 但是kib.txt除外                                              |
| /temp     | 仅忽略项目根目录下的TODO文件，不包括其它目录temp             |
| build/    | 忽略build目录下的左右文件                                    |
| doc/*.txt | 忽略doc目录下的所有.txt文件，但是doc的二级目录下的文件不会被忽略 也是就是 doc/server/arch.txt不会被忽略 |

/路径 和 路径/的区别要区分一下