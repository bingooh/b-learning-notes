### 参考文档
- [Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)
- [Git简明指南](http://rogerdudler.github.io/git-guide/index.zh.html)
- [Github工作流程](https://guides.github.com/introduction/flow/)

### 常用命令
```
#配置git  
git config --global user.name  XXX   
git config --global user.email XXX
git config --global http.proxy http://127.0.0.1:1080

git init            创建本地仓库
git status          查看状态
git clone XXX       克隆远程仓库到本地目录
git clone XXX -b xx 克隆远程仓库指定版本或分支
git remote -v       查看远程仓库信息，远程仓库默认名称为origin
git remote add origin <server> 添加远程仓库地址

git log            查看提交日志
git log --graph    查看分支合并图
git reflog         查看全部历史日志

git add -A .       添加所有修改的文件到暂存区
git commit -m XXX  提交所有修改的文件到本地仓库
git pull           把远程仓库的修改拉到本地仓库
git push           把本地仓库的修改推到远程仓库

#拉取代码
git fetch          仅拉取远程仓库的修改，不做合并
git pull           等价于git fetch && git merge
git pull --rebase  拉取修改后，使用rebase代替merge

#拉取远程仓库origin/b1分支到本地仓库b2分支
git pull origin b1:b2
```

### 分支命令
```
# 如需操作远程仓库分支，添加选项"-r"
git branch              查看分支,默认分支标记为*
git branch -a           查看所有分支，包括远程分支
git branch <b1>         创建分支
git branch -d <b1>      删除分支
git branch -D <b1>      删除未合并的分支
git checkout <b1>       切换分支
git checkout -b <b1>    创建&切换分支
git checkout -b <b1> dev        以dev分支为起点，创建&切换到b1分支
git checkout -b <b1> tag1       以tag1标签为起点，创建&切换到b1分支
git checkout -b <b1> origin/b2  创建&切换到b1分支，并关联到远程仓库b2分支

#关联本地分支到远程分支b1
git branch --set-upstream origin <b1>

#关联b1分支到远程仓库2分支，关联后可拉取远程仓库
git branch --set-upstream <b1> origin/<b2>

git push origin <b1>    推送b1分支到远程仓库
git push origin <:b1>   推送已删除的b1分支到远程仓库

#把b1分支合并到当前分支，默认使用Fast Forward模式
#通常为主分支合并功能分支
#FF模式：直接把当前分支指向b1分支，这样删除b1分支将丢失提交日志
#可理解为：git merge b1 into current
git merge b1

#把<name>分支合并到当前分支，不使用Fast Forward模式
#NO-FF模式：合并后产生1个新的提交，然后当前分支指向此提交
git merge --no-ff -m "XXX" <name>

#将b1分支作为当前分支的“基”
#通常为功能分支"变基"到主分支最新提交日志
#默认只保留当前分支最后1个提交日志？
#可理解为：git rebase current onto b1
git rebase <b1>

#以交互模式“变基”，可指定提交模式
git rebase -i <b1>
```

### 标签命令
```
git tag              显示所有标签
git show -q <v1>     显示标签详情
git tag <v1>         对HEAD提交打标签v1
git tag <v1> <log1>  对log1提交打标签v1
git tag -a <v1> -m xx <log1> 对log1提交打标签v1，并指定注释

git tag -d <v1>         删除本地仓库标签v1
git push --tags         推送所有标签
git push origin <v1>    推送标签v1
git push origin :refs/tags/<v1> 删除远程仓库标签v1，需先删除本地标签v1

#切换当前分支到v1标签对应的版本
#运行此命令将导致HEAD detached,即此情况提交的任何修改
#如果没有合并到另一分支，那么在切换到其他分支时将丢失
#
#运行git checkout <b1>将取消HEAD detached
#
#建议checkout标签时创建分支
git checkout <v1> 
```
### 工作空间
- 工作空间：工作区》暂存区》本地仓库》远程仓库
- `git add/rm` 工作区=>暂存区
- `git commit` 暂存区=>本地仓库
- `git pull`   远程仓库=>本地仓库
- `git push`   本地仓库=>远程仓库
- `git checkout -- <file>` 放弃工作区修改
- `git reset HEAD <file>`  从暂存区恢复指定文件到工作区
- `git reset --hard <log>` 恢复到指定日志，从最新日志`undo`到指定日志
- `git cherry-pick <log>`  仅恢复指定日志的操作，实际会创建1个新的提交日志

### 操作提交日志
- 操作日志的时机
    - `git merge`     使用命令行参数
    - `git rebase -i` 交互式选择日志操作
- 日志操作选型(部分)
    - `pick`    保留提交日志
    - `squash`  合并到上1条提交日志
    - `fixup`   丢弃当前提交日志

### 分支
- Git仅包含1个按日期排序的提交日志队列
    - 提交日志记录了文件修改操作等内容
    - 同步日志会对文件执行日志里保存的修改操作
    - `git pull/push`同步本地仓库和远程仓库的日志队列
- Git分支数据结构可理解为`(start,logs,latest)`
    - `start`  指向分支的起始提交日志
    - `latest` 指向分支的最新提交日志
    - `logs`   此分支包含的所有提交日志
    - 一般分支指针理解为分支指向的最新提交日志
    - 提交日志都在唯一的日志队列里，不存在多个日志队列
    - `git branch xx` 创建/删除分支实际只是操作分支指针
- `HEAD`指向当前分支
    - Git大部分操作仅针对当前分支 
    - `git checkout`      修改`HEAD`指向的分支 
    - `git commit/reset`  移动当前分支指向的最新提交日志
    - `git merge  b1`     将当前分支`latest`指向2个分支里最新的日志
    - `git rebase b1`     将当前分支`start`指向`b1`分支的`latest`
- 注：
    - `merge/rebase`都会对文件执行合并后的日志所保存的修改操作
    - `rebase`可以保存单一主线提交日志，而`merge`将保存所有提交日志
        - 如果干分支提交日志不重要，建议`rebase`到主分支
        - 如果干分支提交日志很重要，建议`merge`到主分支
    - 分支`b1`合并`b2`，`b1`将包含`b2`的全部日志。如果`b1`提交更新，那么`b1`的`latest`不变
    - 合并分支，如果无冲突则默认使用`FF`模式
        - 实际上合并后，当前分支将包含被合并分支所有日志，其`latest`指向最新的日志
        - 如果有冲突，则解决冲突后创建1个新的提交，此时`latest`将指向此(合并)日志

### 最简分支策略
- 本地仓库总是使用自己的分支进行开发
- 开发完成后提交PR请求合并回原分支
- Github项目应先`fork`，然后`clone`到本地
```
#假设远程仓库包含master/dev分支
#本地仓库需要基于dev分支开发新功能

#以dev分支为起点，创建新分支my-dev
git checkout -b my-dev dev

#在my-dev进行开发并提交修改
git commit -m xxx

#切换回dev分支，并拉取最新修改
#本地未对dev分支进行修改，所以不会有任何冲突
git checkout dev
git pull --rebase

#切换回my-dev
git checkout my-dev

#如果需保留my-dev提交日志，则merge
git merge dev

#如果不需要保留my-dev提交日志，则rebase
git rebase dev

#如果需要编辑/合并my-dev提交日志，则rebase -i
git rebase -i dev

#解决merge/rebase冲突，并提交修改
git commit -m xxx

#提交my-dev分支到远程仓库
git push --set-upstream origin my-dev

#新建PR，请求合并回远程仓库的dev分支
#在Github/Gitlab页面上创建Pull Request/Merge Request
#指定Assignee为dev分支的负责人/代码审核者

#Assignee选择接受或拒绝PR，可选是否删除my-dev

#注意：my-dev仅限本地使用，建议处理完PR后删除远程仓库my-dev分支
#Github做法是先fork原项目，my-dev只能推送到fork出的项目
```