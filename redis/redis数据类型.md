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

  

### DEMO

```bash
ali:0>hset myhash name pywang # set一个具体的key-value
"1"
ali:0>hset myhash age 28 
"1"
ali:0>hset myhash sex 男
"1"
ali:0>hget myhash name # 获取hash中某个字段的值
"pywang"
ali:0>hget myhash sex
"男"
ali:0>hmset myhash field1 hello field2 world # set多个值
"OK"
ali:0>hget myhash field1
"hello"
ali:0>hmget myhash field1 field2 # 获取多个值
1) "hello"
2) "world"
ali:0>hgetall myhash
1)  "name"
2)  "pywang"
3)  "age"
4)  "28"
5)  "sex"
6)  "男"
7)  "field1"
8)  "hello"
9)  "field2"
10) "world"
###############################################
ali:0>hdel myhash field1 # 删除指定key字段，对应的value也会被删除
"1"
ali:0>hget myhash field1
null
##############################################
ali:0>hlen myhash # 获取键值对的数量
"4"
ali:0>hgetall myhash
1) "name"
2) "pywang"
3) "age"
4) "28"
5) "sex"
6) "男"
7) "field2"
8) "world"

###############################################
ali:0>hexists myhash field1 # 判断hash中指定字段是否存在
"0"
ali:0>hexists myhash name
"1"
##############################################
ali:0>hkeys myhash # 获取hash中的所有key
1) "name"
2) "age"
3) "sex"
4) "field2"
ali:0>hvals myhash # 获取hash中的所有value
1) "pywang"
2) "28"
3) "男"
4) "world"
##############################################

ali:0>hset myhash field1 1 
"1"
ali:0>hincrby myhash field1 1 # 指定值增量
"2"
ali:0>hget myhash field1
"2"
ali:0>hincrby myhash field1 -1 
"1"
ali:0>hget myhash field1
"1"
ali:0>hsetnx myhash field2 hello # 如果field不存在则可以设置
"1"
ali:0>hsetnx myhash field2 hello # 如果field存在则不能设置
"0"
ali:0>hsetnx myhash field1 hello
"0"
ali:0>hgetall myhash
1) "field1"
2) "1"
3) "field2"
4) "hello"
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
#####################################
ali:0>EXISTS list # 判断list是否存在
"0"
ali:0>lset list 0 item # 将列表中指定下标的值替换为另一个值，如果不存在列表则会报错
"ERR no such key"
ali:0>lpush list value1
"1"
ali:0>lrange list 0 0
1) "value1"
ali:0>lset list 0 item # 如果存在则更新当前下标的值
"OK"
ali:0>lrange list 0 0
1) "item"
ali:0>lset list 1 other # 如果不存在下标，则会报错
"ERR index out of range"
###############################################
ali:0>rpush mylist hello
"1"
ali:0>rpush mylist " world"
"2"
ali:0>linsert mylist befroe world other # 将other字符串插入到mylist 中world字符串之前，如果不存在world字符串则报错
"ERR syntax error"
ali:0>linsert mylist before " world" other
"3"
ali:0>lrange mylist 0 -1
1) "hello"
2) "other"
3) " world"
ali:0>linsert mylist after world new
"-1"
ali:0>lrange mylist 0 -1
1) "hello"
2) "other"
3) " world"
ali:0>linsert mylist after " world" new # 将new字符串插入到mylist 中world字符串之后，如果不存在world字符串则报错
"4"
ali:0>lrange mylist 0 -1
1) "hello"
2) "other"
3) " world"
4) "new"
ali:0>
```



### 小结

- LIST实际上是一个链表，before Node after,left, right 都可以插入值
- 如果key不存在，创建新的链表
- 如果key存在，新增内容
- 如果同意除了所有值，空链表，也代表不存在
- 在两边插入或改动值，效率最高。中间元素，相对来说效率第一点



## Set

- 新的存储需求：存储大量的数据，在查询方面提供更高的效率
- 需要存储结构：能够保存大量的数据，搞笑的内部存储机制，便于查询
- set类型：与存储结构完全相同，仅存储key
- SET中的值是无序的

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



### DEMO

```bash
ali:0>sadd myset hello # 插入值
"1"
ali:0>sadd myset pywang
"1"
ali:0>sadd myset loveStudy
"1"
ali:0>smembers myset # 遍历SET
1) "pywang"
2) "loveStudy"
3) "hello"
ali:0>sismember myset hello # 判断某个值是否在set中
"1"
ali:0>smembers myset
1) "pywang"
2) "loveStudy"
3) "hello"
ali:0>scard myset # 获取set中元素的个数
"3"
##########################
ali:0>srem myset hello # 移除set中的指定元素
"1"
ali:0>scard myset
"2"
###################################################
ali:0>smembers myset
1) "pywang"
2) "hello1"
3) "hello"
4) "loveStudy"
5) "hello2"
ali:0>srandmember myset # 随机抽选元素
"hello1"
ali:0>srandmember myset
"hello"
ali:0>srandmember myset
"loveStudy"
ali:0>srandmember myset 2 # 随机抽取多个元素
1) "hello2"
2) "pywang"
####################################################
ali:0>smembers myset
1) "loveStudy"
2) "hello"
3) "hello1"
4) "hello2"
ali:0>spop myset # 随机删除元素
"hello"
ali:0>smembers myset
1) "loveStudy"
2) "hello1"
3) "hello2"
#####################################################

ali:0>sadd key1 a
"1"
ali:0>sadd key1 b
"1"
ali:0>sadd key1 c
"1"
ali:0>sadd key2 c
"1"
ali:0>sadd key2 d
"1"
ali:0>sadd key2 e
"1"
ali:0>sdiff key1 key2 # 求差集
1) "a"
2) "b"
ali:0>sinter key1 key2 # 求交集，例如共同好友
1) "c"
ali:0>sunion key1 key2 # 求并集
1) "c"
2) "e"
3) "a"
4) "b"
5) "d"

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
  



### DEMO

```bash
ali:0>zadd myset 1 one  # 添加一个值one在第1个位置
"1"
ali:0>zadd myset 2 two
"1"
ali:0>zadd myset 3 three 4 four # 添加多个值
"2"
ali:0>zrange myset 0 -1 # 遍历
1) "one"
2) "two"
3) "three"
4) "four"
##############################################
ali:0>zrangebyscore myset -inf +inf # 按照score从小到大排序
1) "one"
2) "two"
3) "three"
4) "four"
ali:0>zrevrangebyscore myset +inf -inf # 按照score从大到小排序
1) "four"
2) "three"
3) "two"
4) "one"
ali:0>zrangebyscore myset -inf +inf withscores # 显示排序字段
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
7) "four"
8) "4"
ali:0>zrangebyscore myset -inf 3 withscores # 显示排序字段<=3的数据
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
##################################################
ali:0>zrange myset 0 -1
1) "one"
2) "two"
3) "three"
4) "four"
ali:0>zrem myset one # 删除one元素
"1"
ali:0>zrange myset 0 -1
1) "two"
2) "three"
3) "four"
###################################################
ali:0>zrange myset 0 -1
1) "two"
2) "three"
3) "four"
ali:0>zcard myset # 获取set中元素数量
"3"
#################################################
ali:0>zrange myset 0 -1
1) "two"
2) "three"
3) "four"
ali:0>zcount myset 0 1
"0"
ali:0>zcount myset 1 3 # 获取指定区间的元素数量
"2"

```

