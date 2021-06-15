# Redis常用命令

## db基本操作

- 切换数据库

  ```bash
  select index 
  ```

- 其他操作

  ```bash
  quit
  ping
  echo message
  ```

- 数据移动

  ```bash
  move key db
  ```

- 数据清除

  ```bash
  dbsize 
  flushdb 清除当前db数据
  flushall 清除所有数据
  ```


- ``` bash
  DBSIZE # 当前数据库中key的数量
  keys * # 查看数据库中所有的key
  EXISTS key # 判断key是否存在，返回1 存在，0 不存在
  EXPIRE key time # 设置过期时间，单位秒
  TTL key # 查看key的有效时间，-2表示已失效
  type key # 查看当前key的类型
  ```
  
  

