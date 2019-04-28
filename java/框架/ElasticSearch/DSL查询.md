# DSL查询语法

## query参数

```json {.line-numbers}
GET /_search
{
    "query": "YOUR_QUERY_HERE"
}
```

### match-all 空查询

匹配所有文档

```json {.line-numbers}
GET /_search
{
    "query":{
        "match_all":{

        }
    }
}
```

### 查询子句

```json {.line-numbers}
GET /_search
{
    "query":{
        "match":{
            "tweet":"elasticsearch"
        }
    }
}
```

### 合并子句

```json {.line-numbers}
{
    "bool":{
        "must":{"match":{"tweet":"elasticsearch"}},
        "must_not":{"match":{"name":"mary"}},
        "should":{"match":{"tweet":"full text"}}
    }
}
```

太多了，详情见文档结构化查询
