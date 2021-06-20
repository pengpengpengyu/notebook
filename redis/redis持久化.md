# Redis持久化

- 将当前数据状态进行保存，快照形式，存储数据结果，数据格式简单，关注点在数据即RDB
- 将数据的操作过程进行保存，日志形式，存储操作过程，存储格式复杂，关注点在数据的操作过程即AOF（append file only）

## RDB

### RDB启动方式 

- 命令

  `save`

- 作用

  手动执行一次保存操作

#### 注意

手动执行save指令的执行会阻塞当前redis服务器，知道当前RDB过程执行完成为止，有可能会造成长时间的阻塞，线上环境不建议使用

### 后台启动方式

- 命令

  ```
  bagsave
  ```

- 作用

  手动启动后台保存操作，但不是立即执行

  

### 自动执行配置

- 配置

  ```
  save second changes
  ```

- 作用

  满足限定时间范围内key的变化数量达到指定数量即执行持久化命令

- 参数

  second:监控时间范围

  changes:监控key的变化量

- 位置

  在conf文件中进行配置

- 范例

  ```
  save 900 1
  save 300 10
  save 60 1000
  ```

### RDB特殊启动方式

- 服务器运行过程中重启

  `debug reload`

- 关闭服务器时指定保存数据

  `shutdown save`

### 恢复rdb文件

1.将rdb文件放在我们redis配置文件中指定的`dir`目录就可以，redis启动的时候会自动检查dump.rdb恢复其中的数据

2.查看需要存放的位置

```bash
ali:0>config get dir # 
1) "dir"
2) "/data" # 如果这个目录存在dum.rdb文件，redis将自动加载数据

```



### 优点

- RDB是一个紧凑的压缩二进制文件，存储效率高
- 内部存储的是redis在某个时间点的数据快照，非常适合数据备份，全量复制等场景
- 恢复数据的速度比AOF快
- 应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复。

### 缺点

- 无论是执行命令还是利用配置，无法做到时时持久化，具有较大的可能性丢失数据
- bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能
- Redis的众多版本未进行RDB文件格式的版本统一，有可能出现个版本服务之间数据格式无法兼容现象

## AOF

- append only file:以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令达到回复数据的目的。与RDB相比可以简单描述为记录数据为记录数据产生的过程。
- AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式

### AOF写数据策略

- always(每次)

  每次写入操作均同步到AOF文件中，数据零误差，性能较低。不建议使用

- everysec(每秒)

  每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高。再系统突然当即的情况下丢失1秒内的数据。建议使用也是默认设置

- no（系统控制）

  由操作系统控制每次同步到AOF的周期，整体过程不可控
  
- 配置

  ```
  appendfsync always|everysec|no
  ```

- 作用

  AOF写数据策略

### AOF功能开启

- 配置

  ```
  appendonly yes|no
  ```

  作用

  是否开启FOR持久化功能，默认为不开启状态


### AOF重写规则

- 进程内已超时的数据不再写入文件
- 忽略无效指令，重写是使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令，如del key1 hdel key2 srem key3等
- 对同一数据的多条写命令合并为一条命令，如lpush list1 a / lpush list1 b /lpush list1 c可以转化为lpush list1 a b c.为放直数据量过大造成客户端缓冲区溢出，对list/set/hash/zset等类型，每条指令最多写入64个元素

### AOF重写方式

- 手动重写

```
bgrewriteaof
```

- 自动重写

```
auto-aof-rewrite-min-size size
auto-aof-rewrite-percentage percentage
```

### AOF自动重写方式

- 自动重写触发条件设置

  ```
  auto-aof-rewrite-min-size size
  auto-aof-rewirte-percentage percent
  ```

- 自动重写出发比对参数（运行指令info persestence获取具体信息）

  ```
  aof_current_size
  aof_base_size
  ```

- 自动重写触发条件

  ```
  aof_current_size>auto-aof-rewrite-min-size
  aof_current_size-aof_base_size
  ______________________________>=auto-aof-rewrite-percentage
  aof_base_size
  ```


### 注意事项

- 如果aof文件有错误，redis是启动不起来的，需要修复aof文件，redis提供了一个工具`redis-check-aof --fix appendonly.aof` 

### 优点

- 每次修改都同步，数据备份的完整性更好
- 每秒同步一次，可能会丢i是一秒的数据
- 从不同步不，效率最高

### 缺点

- 相对于数据文件来说，aof远大于rdb,修复速度慢
- aof运行效率比rdb慢，所以redis默认持久化配置是rdb而不是aof



