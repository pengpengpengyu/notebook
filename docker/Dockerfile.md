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

