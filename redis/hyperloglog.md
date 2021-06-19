# 基数统计

## 简介

Redis2.8.9更新了Hyperloglog数据结构

Redis Hyperloglog 基础统计的算法

优点：占用的内存固定，2^64不同元素的基数，只需要占用12kb内存，如果从内存角度来比较的话Hyperloglog首选。

**网页的UV(一个人访问一个网站多次，但是还是算作一个人)**

传统的方式，set中保存用户id,然后就可以统计set中的元素数量作为标准判断。

这个方式如果保存大量的用户id，就会比较麻烦，我们的目的地是为了计数而不是保存用户ID.

0.81%的错误率！统计UV任务可以忽略不计的。



## DEMO

```bash
ali:0>pfadd mykey a b c d e f g h i j # 创建第一组元素
"1"
ali:0>pfcount mykey # 统计第一组元素数量
"10"
ali:0>pfadd mykey1 i j z x c v b n m # 创建第二组元素
"1"
ali:0>pfcount mykey1
"9"
ali:0>pfmerge mykey mykey1 # 合并两组 到mykey
"OK"
ali:0>pfcount mykey # 查看并集数量
"15"
```



