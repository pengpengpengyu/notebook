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



