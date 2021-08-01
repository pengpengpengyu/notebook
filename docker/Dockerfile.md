# Dockerfile

## 简介

docker file 是用来构建docker镜像的文件，命令参数脚本。

构建步骤：

1. 编写一个dockerfile文件
2. `docker build`命令构建成为一个镜像
3. `docker run ` 运行镜像
4. `docker push ` 发布镜像（DockerHub、阿里云镜像仓库）

##  构建过程

### 基础知识

1. 每个保留关键字（指令）尽量大写
2. 执行顺序从上到下
3. #表示注释
4. 每个指令都会创建提交一个新的镜像层并提交

## 指令

```bash
FROM		# 基础镜像，一切从这里开始 centos
MAINTAINER	# 镜像作者，姓名+邮箱
RUN			# 镜像构建时需要运行的指令
ADD			# 步骤，tomcat镜像，这个tomcat压缩包，添加内容
WORKDIR		# 镜像的工作目录
VOLUMN		# 挂载容器卷的目录
EXPOSE		# 暴漏端口
CMD			# 指定这个容器启动的时候要运行的命令，多个只执行最后一个
ENTRYPOINT	# 指定容器启动时要执行的命令，可以追加命令
ONBUILD		# 当构建一个被继承 Dockerfile 的时候就会运行ONBUILD指令
COPY		# 类似ADD命令，将文件保存到文件中
ENV			# 构建镜像时设置环境变量,该变量一直存在
ARG			# 构建是设置环境变量，构建结束变量删除
```

## 实战测试

```bash
# 1. 编写Dockerfile文件
[root@pywang Dockerfile]# cat myDockerfile
FROM centos
MAINTAINER pywang<985218143@qq.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN yum -y install vim && yum -y install net-tools
EXPOSE 80
CMD echo $MYPATH
CMD echo "======end====="
CMD /bin/bash

# 2. 构建镜像
# 命令  docker build -t 镜像名:[tag] -f DockerFile文件路径 .
# 构建成功后返回
Successfully built 84415c4d4fdb
Successfully tagged mycentos:latest
```

### CMD 和ENTRYPOINT 区别

#### CMD

```bash
## CMD
# 1. 编写Dockerfile
FROM alpine
CMD ["ls", "-a"]

# 2. 构建镜像
 docker build -t testcmd:0.1 -f mytestCmdDockerfile .
 
# 3. 运行容器
[root@pywang Dockerfile]# docker run -it --rm testcmd:0.1
.           dev         media       root        sys
..          etc         mnt         run         tmp
.dockerenv  home        opt         sbin        usr
bin         lib         proc        srv         var

# 4. 执行成功
# 如果docker run时想追加命令则会失败
[root@pywang Dockerfile]# docker run -it --rm testcmd:0.1 -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:367: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.
```

#### ENTRYPOINT

```bash
## ENTYPOINT
# 1. 编写Dockerfile
FROM alpine
ENTRYPOINT ["ls", "-a"]

# 2. 构建镜像
 docker build -t testentrypoint:0.1 -f Dockerfile-entrypoint .
# 3. 运行容器
[root@pywang Dockerfile]# docker run -it --rm testentrypoint:0.1
.           dev         media       root        sys
..          etc         mnt         run         tmp
.dockerenv  home        opt         sbin        usr
bin         lib         proc        srv         var
# 如果docker run时想追加命令，则成功
[root@pywang Dockerfile]# docker run -it --rm testentrypoint:0.1 -l
total 8
drwxr-xr-x    1 root     root             6 Jul 27 15:13 .
drwxr-xr-x    1 root     root             6 Jul 27 15:13 ..
-rwxr-xr-x    1 root     root             0 Jul 27 15:13 .dockerenv
drwxr-xr-x    2 root     root          4096 Jun 15 14:34 bin
drwxr-xr-x    5 root     root           360 Jul 27 15:13 dev
drwxr-xr-x    1 root     root            66 Jul 27 15:13 etc
drwxr-xr-x    2 root     root             6 Jun 15 14:34 home
drwxr-xr-x    7 root     root           247 Jun 15 14:34 lib
drwxr-xr-x    5 root     root            44 Jun 15 14:34 media
drwxr-xr-x    2 root     root             6 Jun 15 14:34 mnt
drwxr-xr-x    2 root     root             6 Jun 15 14:34 opt
dr-xr-xr-x  113 root     root             0 Jul 27 15:13 proc
drwx------    2 root     root             6 Jun 15 14:34 root
drwxr-xr-x    2 root     root             6 Jun 15 14:34 run
drwxr-xr-x    2 root     root          4096 Jun 15 14:34 sbin
drwxr-xr-x    2 root     root             6 Jun 15 14:34 srv
dr-xr-xr-x   13 root     root             0 Jul 27 15:13 sys
drwxrwxrwt    2 root     root             6 Jun 15 14:34 tmp
drwxr-xr-x    7 root     root            66 Jun 15 14:34 usr
drwxr-xr-x   12 root     root           137 Jun 15 14:34 var
```

## 发布镜像到DockerHub

1. 注册DockerHub账号

2. 在服务器登录账号

   ```shell
   [root@pywang testCentos]# docker login -u985218143
   Password:
   WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
   Configure a credential helper to remove this warning. See
   https://docs.docker.com/engine/reference/commandline/login/#credentials-store
   
   Login Succeeded
   [root@pywang testCentos]#              
   ```

3. 提交镜像

   ```shell
   docker push 985218143/centos:1.0
   ```

   

