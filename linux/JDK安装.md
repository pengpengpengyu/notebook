# JDK安装

## 1.下载JDK并拷贝到安装目录

## 2.解压安装包 

>cp -r  /root/tools/jdk-8u161-linux-x64.tar.gz    /usr/local/java
>tar -zxvf jdk-8u161-linux-x64.tar.gz

## 3.配置环境变量

`vim /etc/profile`

加入以下配置

```{.line-numbers}
export JAVA_HOME=/usr/local/java/jdk1.8.0_161
export JRE_HOME=/$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

使配置生效
`source /etc/profile`

## 4.测试

`java -version`
>java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161)
Java HotSpot(TM) 64-Bit Server VM (build 24.60-b09, mixed mode)

## 遇到问题

### 配置无效

之前安装过其他版本JDK重新安装时，配置玩环境变量之后输入`java -version`，显示版本仍然是之前的版本

**解决办法**
到 `/usr/bin` 找到`java`文件，将其删除或改名然后再执行`source /etc/profile`即可