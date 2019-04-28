# 同时使用github和gitlab

## 生成密钥

gitbush切换到`.ssh`目录下
`$cd ~/.ssh/`

### 生成github密钥

```git{.line-numbers}
$ ssh-keygen -t rsa -C "demo@163.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/xxx/.ssh/id_rsa): github_id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in github_id_rsa.
Your public key has been saved in github_id_rsa.pub.
The key fingerprint is:
SHA256:MV7M1OCoiiIazgV8D52uNxhs9zi840PbT2CfOFDLhYQ demo@163.com
The key's randomart image is:
+---[RSA 2048]----+
|     .. ....     |
|    E. + +.      |
|      + = +      |
| .   = = +       |
|  j.= C S        |
|   +** + .       |
|   +oO= +        |
| .ooOi=o         |
| .o=B* o.        |
+----[SHA256]-----+
```

### 生成gitlab密钥

```git{.line-numbers}
$ ssh-keygen -t rsa -C "demo@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/xxx/.ssh/id_rsa): gitlab_id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in gitlab_id_rsa.
Your public key has been saved in gitlab_id_rsa.pub.
The key fingerprint is:
SHA256:2zfJw2RePrrGc6qKqIvH3qylCmzimIpiUKKAWB87A98 demo@qq.com
The key's randomart image is:
+---[RSA 2048]----+
|ooo +  o+        |
|+. + = o.o .     |
|. . + + D +      |
|   . + + +       |
|    . . S .      |
|o  .   . .       |
|o+. ...          |
|Xii=  o..        |
|/B=c+ooo         |
+----[SHA256]-----+
```

## 写入github和gitlab SSH密钥

打开github和gitlab页面，将github_id_rsa.pub和gitlab_id_rsa.pub中密钥分别写入对应的平台SSK

## 配置config文件

```
$cd ~/.ssh/
$vim config
```

config内容如下：

```git{.line-numbers}
# github
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/github_id_rsa
# gitlab
Host gitlab.com
HostName gitlab.com
User git
IdentityFile ~/.ssh/gitlab_id_rsa
```

Host相当于HostName的别名
例如远程仓库地址：`git@github.com:pengpengpengyu/notebook.git`
通过config配置其中github.com即为github配置的Host,如果github配置的Host是

```{.line-numbers}
# github
Host GITHUB
HostName github.com
User git
IdentityFile ~/.ssh/github_id_rsa
```

则远程仓库地址应该写为`git@GITHUB:pengpengpengyu/notebook.git`

## 测试连通性

```
$ ssh -T github.com
Hi pengpengpengyu! You've successfully authenticated, but GitHub does not provide shell access.
```


## 配置仓库

新建两个目录(名字随便取)
>githubRespository
gitlabRespository

### github配置

```git{.line-numbers}
# github
$ cd ~/githubRespository
$ git config --global user.name 'githubdemo'
$ git config --global user.email 'demo@163.com'
```

### gitlab配置

```git{.line-numbers}
# github
$ cd ~/gitlabRespository
$ git config --local user.name 'gitlabdemo'
$ git config --local user.email 'demo@qq.com'
```

## clone项目

原本为`$git clone git@github.com:pengpengpengyu/notebook.git`
现在为`$git clone git@`<font color="red">配置的houstname</front>`:pengpengpengyu/notebook.git`