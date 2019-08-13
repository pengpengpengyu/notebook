# 初识Docker

## docker run

使用`docker run`在容器内运行一个程序：
输出hello world
`docker run ubuntu:15.10 /bin/echo "hello world"`
各个参数解析：

>* docker: Docker 的二进制执行文件。
>* run:与前面的 docker 组合来运行一个容器。
>* ubuntu:15.10指定要运行的镜像，Docker首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。
>* /bin/echo "Hello world": 在启动的容器里执行的命令

## 运行交互式的容器

我们通过docker的两个参数 -i -t，让docker运行的容器实现"对话"的能力

`docker run -i -t ubuntu:15.10`
>root@dc0050c79503:/#

参数解析：

>* -t:在新容器内指定一个伪终端或终端。
>* -i:允许你对容器内的标准输入 (STDIN) 进行交互。

此时我们已进入一个 ubuntu15.10系统的容器

我们尝试在容器中运行命令 cat /proc/version和ls分别查看当前系统的版本信息和当前目录下的文件列表
>rssoot@c25b6c3f2165:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@c25b6c3f2165:/#

使用`ctrl+p` 或`exit`退出容器

## 启动容器（后台模式）

`docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"`

>2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63

上方字符串是容器ID,每个容器ID是唯一的

使用`docker ps` 查看运行中的容器
>[root@localhost pywang]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
f136e4da007d        ubuntu:15.10        "/bin/sh -c 'while t…"   34 seconds ago      Up 33 seconds                           relaxed_keller


`docker logs 容器ID` 查看容器内的标准输出

>[root@localhost pywang]# docker logs f136
hello world
hello world
hello world
hello world
hello world
hello world
hello world
hello world
hello world

## 停止容器

`docker stop 容器ID`
>[root@localhost pywang]# docker stop f136
f136
