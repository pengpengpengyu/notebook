# centos安装mysql5.7

## 1.添加Mysql5.7仓库

`sudo rpm -ivh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm`

## 2.确认Mysql仓库成功添加

`sudo yum repolist all | grep mysql | grep enabled`

如果展示像下面,则表示成功添加仓库:

> mysql-connectors-community/x86_64  MySQL Connectors Community    enabled:     51
mysql-tools-community/x86_64       MySQL Tools Community         enabled:     63
mysql57-community/x86_64           MySQL 5.7 Community Server    enabled:    267


## 3.开始安装Mysql5.7

`sudo yum -y install mysql-community-server`


## 4.启动Mysql

1. 启动

`sudo systemctl start mysqld`

2. 设置系统启动时自动启动

`sudo systemctl enable mysqld`

3. 查看启动状态

`sudo systemctl status mysqld`

## 5.Mysql的安全设置
CentOS上的root默认密码可以在文件/var/log/mysqld.log找到，通过下面命令可以打印出来

`cat /var/log/mysqld.log | grep -i 'temporary password'`

执行下面命令进行安全设置，这个命令会进行设置root密码设置，移除匿名用户，禁止root用户远程连接等

`mysql_secure_installation`

## 6.设置数据库编码为utf8

1. 打开配置文件

`sudo vim /etc/my.cnf`

2. 在[mysqld]，[client]，[mysql]节点下添加编码设置

> [client]
default-character-set=utf8

> [mysql]
default-character-set=utf8

> [mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8

3. 重启Mysql即可

`sudo systemctl restart mysqld`

参考地址 https://juejin.im/post/5c088b066fb9a049d4419985