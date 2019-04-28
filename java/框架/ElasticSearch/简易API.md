# API

## 文档

文档相当于JSON的rootObject, 必须包含_type, _index, _id三个字段

## 生成文档

### ID

#### 使用自带ID

```{.line-numbers}
PUT /{index}/{type}/{id}
{
"field": "value",
...
}
```

**demo:**

```{.line-numbers}
PUT /website/blog/123
{
"title": "My first blog entry",
"text": "Just trying this out...",
"date": "2014/01/01"
}
```

**结果:**

```{.line-numbers}
{
"_index": "website",
"_type": "blog",
"_id": "123",
"_version": 1,
"created": true
}
```

#### ElasticSearch自动生成ID

<font style="color:red">请求方式为POST </font>

url中值包含_index和_type两个字段

```json{.line-numbers}
POST /website/blog/
{
"title": "My second blog entry",
"text": "Still trying this out...",
"date": "2014/01/01"
}
```

结果

```json {.line-numbers}
{
"_index": "website",
"_type": "blog",
"_id": "wM0OSFhDQXGZAWDf0-drSA",
"_version": 1,
"created": true
}
```

## 检索文档

### 根据ID检索

<font style="color:red">请求方式为GET </font>

`GET /_index/_type/_id?pretty`

**demo:**
`GET /website/blog/123?pretty`

结果：

```json {.line-numbers}
{
    "_index":"website",
    "_type":"blog",
    "_id":"123",
    "_version":1,
    "found":true,
    "_source":{
        "title":"My first blog entry",
        "text":"Just trying this out...",
        "date":"2014/01/01"
    }
}
```

找到文档founnd字段为treu,否则false
pretty属性美化返回结果，_source为推送的原始格式，不会被美化

### 获取某些字段

`GET /website/blog/123?_source=title,text`

结果：

```json {.line-numbers}
{
    "_index":"website",
    "_type":"blog",
    "_id":"123",
    "_version":1,
    "exists":true,
    "_source":{
        "title":"My first blog entry",
        "text":"Just trying this out..."
    }
}
```

### 只查询_source

`GET /website/blog/123/_source`

结果：

```json {.line-numbers}
{
    "title":"My first blog entry",
    "text":"Just trying this out...",
    "date":"2014/01/01"
}
```

### 检查文档是否存在

`XHEAD http://localhost:9200/website/blog/123`

存在：
>HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0

不存在:
>404 Not Found

### 根据条件检索多个文档

#### 不同_index

请求：

```json {.line-numbers}
POST /_mget
{
    "docs":[
        {
            "_index":"website",
            "_type":"blog",
            "_id":2
        },
        {
            "_index":"website",
            "_type":"pageviews",
            "_id":1,
            "_source":"views"
        }
    ]
}
```

结果：

```json {.line-numbers}
{
    "docs":[
        {
            "_index":"website",
            "_id":"2",
            "_type":"blog",
            "found":true,
            "_source":{
                "text":"This is a piece of cake...",
                "title":"My first external blog entry"
            },
            "_version":10
        },
        {
            "_index":"website",
            "_id":"1",
            "_type":"pageviews",
            "found":true,
            "_version":2,
            "_source":{
                "views":2
            }
        }
    ]
}
```

#### 同一个_index或_type

请求：

```json {.line-numbers}s
POST /website/blog/_mget
{
    "docs":[
        {
            "_id":2
        },
        {
            "_type":"pageviews",
            "_id":1
        }
    ]
}
```

#### 设置超时时间

`GET /_search?timeout=10ms`
Elasticsearch将返回在请求超时前收集到的结果。

#### 分页

`GET /_all/doc/_search?size=11&from=0`

>GET /_search
{
"from": 30,
"size": 10
}

## 更新文档

>PUT /website/blog/123
{
"title": "My first blog entry",
"text": "I am starting to get the hang of this...",
"date": "2014/01/02"
}

更新之后version会增加

```json {.line-numbers}
{
"_index" : "website",
"_type" : "blog",
"_id" : "123",
"_version" : 2,
"created": false <1>
}
```

<1> created 标识为 false 因为同索引、同类型下已经存在同ID的文档

## 删除文档

### 根据ID

`DELETE /website/blog/123`

如果找到该文档响应数据里_version会增加
>{
"found" : true,
"_index" : "website",
"_type" : "blog",
"_id" : "123",
"_version" : 3
}

未找到则分会404, _version依旧会增加
>{
"found" : false,
"_index" : "website",
"_type" : "blog",
"_id" : "123",
"_version" : 4
}

删除一个文档也不会立即从磁盘上移除，它只是被
标记成已删除。Elasticsearch将会在你之后添加更多索引的时候才会在后台进行删除内
容的清理。

### 查看分词效果

`GET /_analyze?analyzer=standard&text=Text to analyze`