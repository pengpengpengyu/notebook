# Docker网络

## 概念

### 查看网络列表

```shell
[root@pywang ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
19137ee4bd98   bridge    bridge    local
fb568127e81c   host      host      local
fc33d8b9a264   none      null      local 
```

### 网络模式

bridge：桥接模式, docker默认模式，自己创建也使用bridge模式

none：不配置网络

host：和宿主机共享网络

### 测试

```shell
# 直接启动的命令--network bridge,这个就是我们的docker0
docker run -d -P --name tomcat01 tomcat
docker run -d -P --name tomcat02 --network bridge tomcat

# docker0 特点：默认，容器名不能访问（即直接ping容器名称，无法找到Host）， --link可以打通连接

# 我们也可以自定义一个网络
# bridge 模式
# 子网地址 192.168.0.0 掩码 255.255.0.0
# 网关 192.168.0.1 
# 网络名称 redis
[root@pywang ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 redis
b128ce2e8caf83285d26efb1566ab7ac1eca1e1e14f013a120726dec640d6ed5
[root@pywang ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
19137ee4bd98   bridge    bridge    local
fb568127e81c   host      host      local
fc33d8b9a264   none      null      local
b128ce2e8caf   redis     bridge    local

# 查看详细信息
[root@pywang ~]# docker network inspect redis
[
    {
        "Name": "redis",
        "Id": "b128ce2e8caf83285d26efb1566ab7ac1eca1e1e14f013a120726dec640d6ed5",
        "Created": "2021-08-01T20:26:44.260436552+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]


# 启动容器测试
[root@pywang ~]# docker run -d -P --name tomcat01 --network redis tomcat:8.0
a1942877c348f0e4e1072ce05ca89355d744e2d4558d9156a8ce0a5ebad98072
[root@pywang ~]# docker run -d -P --name tomcat02 --network redis tomcat:8.0
e1558fd941ba8b8a3ac2b775c19bcb77c828ebb3c98d4f9918de4b6b714db1a2
# 已经可以通过容器名称ping通
[root@pywang ~]# docker exec -it tomcat01 ping tomcat02
PING tomcat02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat02.redis (192.168.0.3): icmp_seq=1 ttl=64 time=0.136 ms
64 bytes from tomcat02.redis (192.168.0.3): icmp_seq=2 ttl=64 time=0.116 ms
64 bytes from tomcat02.redis (192.168.0.3): icmp_seq=3 ttl=64 time=0.122 ms
64 bytes from tomcat02.redis (192.168.0.3): icmp_seq=4 ttl=64 time=0.106 ms
^C
--- tomcat02 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3049ms
rtt min/avg/max/mdev = 0.106/0.120/0.136/0.010 ms
```

## 网络连通

将一个容器连接到某个网络上

```shell
[root@pywang ~]# docker network connect --help

Usage:  docker network connect [OPTIONS] NETWORK CONTAINER

Connect a container to a network

Options:
      --alias strings           Add network-scoped alias for the container
      --driver-opt strings      driver options for the network
      --ip string               IPv4 address (e.g., 172.30.100.104)
      --ip6 string              IPv6 address (e.g., 2001:db8::33)
      --link list               Add link to another container
      --link-local-ip strings   Add a link-local address for the container
# 将容器tomcat03连接到网络redis上
[root@pywang ~]# docker network connect redis tomcat03
# 测试已经可以正常ping通
[root@pywang ~]# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat02.redis (192.168.0.3): icmp_seq=1 ttl=64 time=0.170 ms
64 bytes from tomcat02.redis (192.168.0.3): icmp_seq=2 ttl=64 time=0.127 ms
64 bytes from tomcat02.redis (192.168.0.3): icmp_seq=3 ttl=64 time=0.113 ms
^C
--- tomcat02 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2088ms
rtt min/avg/max/mdev = 0.113/0.136/0.170/0.027 ms
```

