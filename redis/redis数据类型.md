# redis数据类型

## 常用命令

- 设置过期时间
`EXPIREAT key value`

## String

### 常用命令

`set key value`
`get key`

### 注意事项

- 最大大小512M
- 数值最大值java中long的最大值

### DEMO

```bash
ali:0>set name pywang # 设置值
"OK"
ali:0>get name # 获取值
"pywang"
ali:0>EXISTS name # 查看key是否存在
"1"
ali:0>APPEND name " hello" # 如果已经存在key则追加，否则新增key
"12"
ali:0>get name
"pywang hello"
ali:0>STRLEN name # value长度
"12"
ali:0>set views 0 
"OK"
ali:0>get views
"0"
ali:0>incr views # value +1 
"1"
ali:0>get views
"1"
ali:0>incr views
"2"
ali:0>get views
"2"
ali:0>decr views # value -1
"1"
ali:0>get views
"1"
ali:0>INCRBY views 10 # value + 10 指定增长
"11"
ali:0> get views
"11"
ali:0>DECRBY views 10 # value -10 指定减少
"1"
ali:0>get views
"1"
ali:0>set key2 abdcefg
"OK"
ali:0>get key2
"abdcefg"
ali:0>SETRANGE key2 1 xx # 从第2个字符开始替换xx
"7"
ali:0>get key2
"axxcefg"
ali:0>SETEX key3 30 "hello world" #设置过期时间， 设置key3的值为hello ,30秒后过期
"OK"
ali:0>get key3
"hello world"
ali:0>SETNX mykey "redis" # 不存在再设置（在分布式锁中经常使用），如果key不存在，创建key
"1"
ali:0>ttl key3
"1"
ali:0>ttl key3
"-2"
ali:0>SETNX mykey "MongoDB" # 如果key存在，则创建失败
"0"
ali:0>get mykey
"redis"
ali:0>MSET k1 v1 k2 v2 k3 v3 # 同时设置多个值
"OK"
ali:0>keys *
1) "k3"
2) "k1"
3) "k2"
ali:0>MGET k1 k2 k3 # 同时获取多个值
1) "v1"
2) "v2"
3) "v3"
ali:0>MSETNX k1 v1 k4 v4 # MSETNX是原子操作，要么都成功，要么都失败
"0"
ali:0>keys *
1) "k3"
2) "k1"
3) "k2"
ali:0>getset db redis # 如果不存在值，则返回nil，设置新值
null
ali:0>get db 
"redis"
ali:0>getset db mongodb # 如果不存值，获取原来的值，并设置新值
"redis"
ali:0>get db
"mongodb"
ali:0>
```



## Hash

- 对一系列存储的数据进行编组，方便管理，典型应用存储对象信息
- 需要存储结构：一个存储空间保存多个键值对
- hash类型：底层使用哈希表结构实现数据存储
- hash存储结构优化
  - 如果field数量较少，存储结构优化为类似数组结构
  - 如果field数量较多，存储结构使用HashMap结构

### 常用命令

- 添加/修改数据

  ```
  hset key field value
  ```

- 获取数据

  ```
  hget key field
  hgetall key
  ```

- 删除数据

  ```
  hdel keyt field1 [field2]
  ```

- 添加/修改多个数据

  ```
  hset key field1 value1 field2 value2
  ```

- 获取多个数据

  ```
  hmget key feild1 field2 ...
  ```

- 获取哈希表中的字段数量

  ```
  hlen key
  ```

- 获取哈希表中是否存在指定字段

  ```
  hexists key field
  ```

- 获取哈希表中所有的字段名或字段值

  ```
  hkeys key
  hvals key
  ```

- 设置指定字段的数值数据增加指定范围的值

  ```
  hincrby  key feild increment
  hincrbyfloat key field increment
  ```

  

### 注意事项

- hash类型下的value值能存储字符串，不允许存储其他数据类型，不存在嵌套现象。如果数据未找到,对应的值为nil
- 每个hash可以存储2^32-1个键值对
- hash类型十分贴近对象的存储形式，冰洁可以灵活添加删除对象属性。但hash设计初衷并不是为了存大量独享而设计的，切记不可滥用，更不可以将hash作为对象列表使用
- hetall操作可以获取全部属性，如果内部field过多，便利整体数据效率就会很低，有可能成为数据访问瓶颈



## list

- 数据存储需求：存储多个数据，并对数据进入存储空间的顺序进行区分
- 需要的存储结构：一个存储空间保存多个数据，并且通过数据可以体现进入顺序
- list类型：保存多个数据，底层使用双向链表存储结构实现

### 常用命令

- 添加/修改数据

  ```
  lpush key value1 ...
  rpush key value1 ...
  ```

- 获取数据

  ```
  lrange key start top
  lindex key index
  llen key
  ```

- 获取并移除数据

  ```
  lpop key
  rpop key
  ```

### 注意事项

- list中保存的数据都是Strign类型的数据总 容量是有限的，最多2^32-1个元素
- list具有索引的概念，但是操作数据时通常一队列形式进行入队出队操作，或以栈的形式进行入栈出栈操作
- 获取全部数据操作，结束索引设置为-1
- list可以对数据进行分页操作，通常第一页的信息来自于list，第2页及更多的信息通过数据库形式加载

### DEMO

```bash
ali:0>LPUSH list one # 将一个值或多个值插入到列表头部（左）
"1"
ali:0>LPUSH list two 
"2"
ali:0>LPUSH list three
"3"
ali:0>LRANGE list 0 -1 # 获取list中的值
1) "three"
2) "two"
3) "one"
ali:0>LRANGE list 0 1 # 通过区间获取具体的值
1) "three"
2) "two"
ali:0>RPUSH list righr # 将一个或多个值插入到列表的尾部（右）
"4"
ali:0>LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "righr"
ali:0>LPOP list # 从左侧移除值
"three"
ali:0>RPOP list # 从右侧移除值
"righr"
ali:0>LRANGE list 0 -1
1) "two"
2) "one"
ali:0>lindex list 1 # 通过下标获取list中的某一个值
"one"
ali:0>lindex list 0
"two"
ali:0>llen list # 返回列表的长度
"2"
ali:0>
#########################################
ali:0>lpush list one
"1"
ali:0>lpush list two
"2"
ali:0>lpush list three
"3"
ali:0>lpush list three
"4"
ali:0>lrange list 0 -1
1) "three"
2) "three"
3) "two"
4) "one"
ali:0>lrem list 1 three # 移除list中的1个three,精确匹配
"1"
ali:0>lrange list 0 -1
1) "three"
2) "two"
3) "one"
ali:0>lpush list three
"4"
ali:0>lrem list 2 three # 移除list中的2个three
"2"
ali:0>lrange list 0 -1
1) "two"
2) "one"
##############################################
ali:0>rpush mylist  hello
"1"
ali:0>rpush mylist hello1
"2"
ali:0>rpush mylist hello2
"3"
ali:0>rpush mylist hello3
"4"
ali:0>ltrim mylist 1 2 # 通过下标截取指定的长度，这个list已经被改变了，只剩下截取的元素
"OK"
ali:0>lrange mylist 0 -1
1) "hello1"
2) "hello2"
#############################################
rpoplpush # 移除列表中的最后一个元素，将他移动到新的列表中

ali:0>rpush mylist hello
"1"
ali:0>rpush mylist hello1
"2"
ali:0>rpush mylist hello2
"3"
ali:0>rpush mylist hello3
"4"
ali:0>rpoplpush mylist myotherlist # 移除列表中的最后一个元素，将他移动到新的列表中
"hello3"
ali:0>lrange mylist 0 -1
1) "hello"
2) "hello1"
3) "hello2"
ali:0>lrange myotherlist 0 -1 # 查看移动目标列表，确实存在该值
1) "hello3"
```



## Set

- 新的存储需求：存储大量的数据，在查询方面提供更高的效率
- 需要存储结构：能够保存大量的数据，搞笑的内部存储机制，便于查询
- set类型：与存储结构完全相同，仅存储key

### set类型数据的基本操作

- 添加数据

  ```
  sadd key member1 ...
  ```

- 获取全部数据

  ```
  smember key
  ```

- 删除数据

  ```
  srem keyh number1 ...
  ```

## Sorted_set

- 存储需求：数据排序有利于数据的有效展示，需要提供一种可以根据自身特征进行排序的方式
- 存储结构：新的存储模型，可以保存可排序的数据
- sorted_set类型：在set的存储结构基础上添加可排序字段

### 常用命令

- 添加数据

  ```
  zadd key scorel meber1 [score2 member2]
  ```

- 获取全部数据

  ```
  zrange key starty stop [withscores]
  zrevrange key start stop [withscores]
  ```

- 删除数据

  ```
  zrem key member ...
  ```

- 按条件获取数据

  ```
  zrangebyscore key min max [withscores] [limit]
  zrevrangebyscore key max min [withscores]
  ```

- 条件删除数据

  ```
  zremrangebyrank key start stop 
  zremrangebyscore key min max
  ```

- 获取集合数据总量

  ```
  zcard key
  zcount key min max
  ```

