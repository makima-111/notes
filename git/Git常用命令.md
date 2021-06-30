# Git

## 一、创建版本库和添加文件

* git init 初始化当前路径为git仓库
* git add  *fillname* 添加文件到仓库
* git commit -m *"message"* 提交已添加的文件到仓库

## 二、版本控制

* git status 显示版本状态
* git diff 显示修改内容
* git diff HEAD -- readme.txt 显示工作区和版本库里面最新版本的区别

* git log  查看提交日志 --pretty=online 减少输出内容

### 1、版本回退

* git reset --hard HEAD^ 回退到上一个版本
* git reset --hard *commit_id*   回退到目标版本

* git reflog 显示执行过的命令

### 2、撤销修改

* git checkout -- *file*  将文件恢复到最近一次commit 或 add时的状态
* git reset HEAD  *file*  将暂存区文件丢弃

### 3、删除文件

* 删除 test文件后
* git rm test.txt 删除文件，commit提交
* git checkout -- test.txt还原文件

## 三、远程仓库

* git remote add  <name origin >  <url> 将当前目录库
* git push <name origin > <branchName> 将分枝推到远程库
* git remote rm <name>
* git remote -v查看远程库信息
* git remote rm <name> 删除远程库
* git clone git@github.com:michaelliao/gitskills.git 使用SSH克隆库

## 四、分支管理

### 1、创建合并分支

* git switch -c dev 创建dev分支并切换 -c表示创建并切换
* git branch dev 创建dev分支
* git switch dev 切换dev分支
* git branch 查看当前分支
* git merge dev 将dev分支合并到当前分支
* git branch -d dev 删除dev分支
* git log --graph --pretty=oneline --abbrev-commit 查看分支树

### 2、分支管理策略

* git merge --no-ff -m "message"  dev 合并分支不适用Fast forward 模式
* 不用Fast forward模式，就能在分支树中看出合并动作

### 3、Bug分支

* git stash 保存工作现场（用于需要切换分支，但是不能提交当前分支的情况）
* git stash list 查看保存现场列表
* git stash pop 恢复现场并删除
* git stash apply <stash_name> 恢复内容不删除
* git stash drop 删除内容
* git cherry-pick <commitId> 将特定提交修改应用到当前分支

### 4、Feature分支

* 新建分支 
* git branch -d <name> 强行删除

### 5、多人协作

1. git clone -b [branchname] git@github.com:michaelliao/gitskills.git 使用SSH克隆库
2. git switch -c  dev origin/dev 创建本地分支
3. git checkout -b dev origin/dev创建本地分支
4. 修改文件
5. git add enc.txt
6. git commit -m "add env" 提交修改
7. git push origin dev 将本地提交的修改推到远程仓库上
8. 如果推送失败
9. git branch --set-upstream-to <branch-name> origin/<branch-name> 将本地分支和远程分支对应起来
10. git pull 把修改拉到本地
11. 手动解决冲突
12. git commit 提交
13. git push origin dev 提交修改到远程仓库

### 6、rebase

* git rebase 将分支提交历史整理为一条直线

### 7、subtree 子树

子树允许子项目包含在主项目的子目录中，可选择包含子项目的整个历史。

不要将子树与用于相同任务的子模块混淆。与子模块不同，子树不需要任何特殊结构（如 .gitmodules 文件或 gitlinks）出现在您的存储库中，并且不会强迫存储库的最终用户做任何特殊的事情或了解子树的工作原理。子树只是一个子目录，可以以任何您想要的方式与您的项目一起提交、分支和合并。

它们也不应与使用子树合并策略相混淆。主要区别在于，除了将另一个项目合并为一个子目录之外，您还可以从您的项目中提取子目录的整个历史记录并使其成为一个独立的项目。与子树合并策略不同，您可以在这两个操作之间来回交替。如果独立库更新，您可以自动将更改合并到您的项目中；如果您更新项目中的库，您可以再次“拆分”更改并将它们合并回库项目。

例如，如果您为一个应用程序创建的库最终在其他地方有用，您可以提取其整个历史并将其发布为自己的 git 存储库，而不会意外地混合应用程序项目的历史记录。

```shell
git subtree add   -P <prefix> <commit>
git subtree add   -P <prefix> <repository> <ref>
git subtree pull  -P <prefix> <repository> <ref>
git subtree push  -P <prefix> <repository> <ref>
git subtree merge -P <prefix> <commit>
git subtree split -P <prefix> [OPTIONS] [<commit>]
```



## 五、标签管理

* tag相当于为特定的commitId起一个别名，方便使用。

### 1、创建标签

#### 最新版本打标签

1. 切换到分支上
2. git tag <name> 打标签
3. git tag 查看标签

#### 历史版本打标签

1. git log --pretty=oneline --abbrev-commit 查看历史版本
2. git tag  <name> <commitId>
3. git tag -a <name> -m <message> <commit-id>
4. git show <name> 展示标签信息

### 2、删除标签

* git tag -d v0.1 删除本地标签
* git push origin v1.0 推送标签到远程
* git push origin --tags 全部推送
* git push origin :refs/tags/v0.9 删除远程标签

## 六、其他功能

### 1、忽略特殊文件

* 通过编写 .gitignore文件忽略特定中间文件
* https://github.com/github/gitignore 常用配置
* git add -f 强制添加忽略文件
* git check-ignore -v app.class 检查忽略配置

### 2、配置命令别名

* git config --global alias.st status 用st代表status
* git config --global alias.co checkout
* git config --global alias.ci commit
* git config --global alias.br branch
* git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit" 方便显示查看
* 全局配置文件在 ~/.gitconfig 库配置文件在 .git/config