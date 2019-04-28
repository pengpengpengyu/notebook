# git常用命令

[官方文档地址](https://git-scm.com/book/zh/v2/)

git操作文件，切换路径等命令与linux相同.

## 初始化版本库

**初始化版本库:** `git init`

## 向缓存区添加文件

**向缓存区添加修改：** `git add [filename，...]`

**提交文件：** `git commit -m "此处是注释"`
-m参数后为注释内容
> <font color="red">可添加多个文件后统一提交</font>

**向远程版本库推送:** `git push origin master`
> origin 是远程仓库名称，master为分支名称

## 查看日志

**查看提交日志:** `git log --pretty=oneline`
**查看提交内容差异:** `git log -p -2`
> 最近2次提交内容差异

## 版本退回

**版本退回：** `git reset --hard HEAD^`
> `^`表示退回上个版本`^^`表示退回到上上个版本，上100个写法为`HEAD~100`
`git reset --head 1049` 退回到具体版本id

**查看恢复版本日志：** `git reflog` 
> $git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
e475afc HEAD@{3}: commit: add distributed
eaadf4e HEAD@{4}: commit (initial): wrote a readme file

## 工作区、缓冲区数据修改

**查看工作区和版本库里的文件差异：** `git diff HEAD -- readme.txt`

**撤销工作区的修改：** `git checkout -- file`
命令`git checkout -- readme.txt`意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：

一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次git commit或git add时的状态。

**撤销缓存区修改：** `git reset HEAD <file>`
> 该命令把缓存区修改回退到工作区

## 删除文件

`git rm <file>`
> 删除文件同样需要`git commit`提交

## 远程仓库

**关联远程仓库** `git remote add origin git@pywang.gitlab.example.com/test.git`

**取消远程关联** `git remote remove origin`

**复制远程库** `git clone git@10.10.11.59:root/sgstexpert.git`
**复制远程分支** `git clone -b dev git@10.10.11.59:root/sgstexpert.git`
> -b 表示从分支下载
dev 分支名称

**重命名远程仓库:** `git remote rename source desc`

## 分支

**创建分支：** `git branch dev`
**创建分支并切换到该分支:** `git checkout -b dev`
**切换分支:** `git checkout dev`
**合并分支：** `git master dev`
> demo:
git checkout master
git merged dev

**查看已经合并到当前分支上的分支:** `git branch --merged`
**查看未合并到当前分支上的分支:** git branch --no-merged
**删除分支:** `git branch -d dev`
**删除远程分支：** `git push origin --delete dev`

## 变基

说明： 将提交到某一分支上的所有修改都移至另一分支上
命令：`git rebase master`
> demo: 将experiment分支上的修改转移到master分支 
git checkout experiment
git rebase master

## 添加SSH密钥

参考以下连接：[SSH密钥生成及部署](https://blog.csdn.net/fancancan/article/details/82258326)

## 常见问题

* **`git status`显示中文编码问题：**

> Git 命令窗口下输入  `git config --global core.quotepath false`