# Centos下docker安装

## 更新yum包

`yum update`
速度很慢

## 下载安装脚本（centos）并执行脚本

`curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun`

## 启动Docker进程

`systemctl start docker`

## 验证是否安装成功

`docker run hello-world`
由于本地仓库没有hello-world镜像所以会到远程仓库拉取，并在容器内运行。
**安装结束**

## 镜像加速

更换国内镜像地址。
新版的 Docker 使用 /etc/docker/daemon.json（Linux） 或者 %programdata%\docker\config\daemon.json（Windows） 来配置 Daemon。

请在该配置文件中加入（没有该文件的话，请先建一个）：
>{
    "registry-mirrors": ["http://hub-mirror.c.163.com"]
}

 **重载此配置文件**
`systemctl daemon-reload`
 **重启 docker**
`systemctl restart docker`

#### 删除Docker CE

>$ sudo yum remove docker-ce

>$ sudo rm -rf /var/lib/docker

## 卸载Docker

> ```shell
> yum remove docker-ce docker-ce-cli containerd.io
>  rm -rf /var/lib/docker  # 删除docker默认安装路径
>  rm -rf /var/lib/containerd  ## 删除卷
> ```
