# Redis常用命令

## db基本操作

- 切换数据库

  ```
  select index 
  ```

- 其他操作

  ```
  quit
  ping
  echo message
  ```

- 数据移动

  ```
  move key db
  ```

- 数据清除

  ```
  dbsize 
  flushdb 清除当前db数据
  flushall 清除所有数据
  ```

  