# ES 常用设置

## 字段映射设置

```json{.line-numbers}
PUT expert/_mapping/expert
{
    "properties":{
        "administrativeLevel":{
            "type":"keyword"
        },
        "age":{
            "type":"long"
        },
        "asjcs":{
            "type":"keyword",
            "fields":{
                "keyword":{
                    "type":"keyword",
                    "ignore_above":256
                },
                "pinyin":{
                    "type":"text",
                    "analyzer":"pinyin_analyzer"
                },
                "ik":{
                "type":"text",
                "analyzer":"ik_max_word"
            },
            "suggest" : { 
                    "type" : "completion",                            "analyzer": "completion_analyzer",
                    "search_analyzer": "hanzi_pinyin_analyzer"
                }
            }
            
        },
      "honors":{
        "type": "keyword"
      },
        "bankIsSameCity":{
			"type":"keyword"
        },
        "birthCountry":{
            "type":"keyword"
        },
        "birthday":{
            "type":"keyword"
        },
        "createTime": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss"
         
        },
         "updateTime": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss"
         
        },
        "dataIntegrity":{
            "type":"long"
        },
        "dataSource":{
           "type":"keyword"
        },
        "department":{
            "type":"keyword"
        },
        "duty":{
           "type":"keyword"
        },
        "educationBackground":{
           "type":"keyword"
        },
        "email":{
           "type":"keyword"
        },
        "faxNumber":{
            "type":"keyword"
        },
        "id":{
           "type":"keyword"
        },
        "idNumber":{
            "type":"keyword"
        },
        "idType":{
            "type":"keyword"
        },
        "introduction":{
           "type":"keyword",
           "fields": {
              "ik":{
                "type":"text",
                "analyzer":"ik_max_word"
            }
           }
        },
        "isAcademician":{
           "type":"keyword"
        },
        "isDelete":{
            "type":"keyword"
        },
        "isDoctoralSupervisor":{
           "type":"keyword"
        },
        "jobTittle":{
            "type":"keyword"
        },
        "jobTittleLevel": {
          "type": "keyword"
        },
        "nameCh":{
            "type":"text",
            "fields":{
                "keyword":{
                    "type":"keyword",
                    "ignore_above":256
                },
                "pinyin":{
                    "type":"text",
                    "analyzer":"pinyin_analyzer",
                    "fielddata": true
                },
                 "suggest" : { 
                    "type" : "completion",                            "analyzer": "completion_analyzer",
                    "search_analyzer": "hanzi_pinyin_analyzer"
                }
            }
        },
        "nameEn":{
            "type":"text",
            "fields":{
                "keyword":{
                    "type":"keyword",
                    "ignore_above":256
                }
            }
        },
        "nationality":{
            "type":"keyword"
        },
        "organization":{
           "type":"text",
            "fields":{
                "keyword":{
                    "type":"keyword",
                    "ignore_above":256
                },
                "pinyin":{
                    "type":"text",
                    "analyzer":"pinyin_analyzer"
                },
                "ik":{
                "type":"text",
                "analyzer":"ik_max_word"
            },
            "suggest" : { 
                    "type" : "completion",        "analyzer": "completion_analyzer",
                    "search_analyzer": "hanzi_pinyin_analyzer"
                }
            }
        },
        "organizationAddr":{
            "type":"keyword"
        },
        "organizationPostcode":{
           "type":"keyword"
        },
        "remark":{
           "type":"keyword"
        },
        "sex":{
           "type":"keyword"
        },
        "tel":{
           "type":"keyword"
        },
        "type":{
           "type":"keyword"
        },
        "workTel":{
           "type":"keyword"
        },
		"graduatedDate":{
			"type": "keyword"
		},
		"technologyReward": {
		  "type": "keyword"
    }
}
}
```

## 分词设置及常用属性设置

```java{.line-numbers}
PUT /expert/
{
	"settings":{
		"number_of_shards": 1,
		"number_of_replicas": 1,
		"max_result_window": 100000, ## 返数据最大条数
		"analysis": {
            "analyzer": {
                "pinyin_analyzer": {
                    "tokenizer": "my_pinyin"
                },
                "ik_max_word":{
                  "tokenizer": "ik_max_word"
                },
                 "completion_analyzer": {
                    "tokenizer": "completion_pinyin"
                },
                "hanzi_pinyin_analyzer": {
                  "tokenizer" : "hanzi_pinyin_analyzer"
                }
            },
            "tokenizer": {
                "my_pinyin" : {
                      "type" : "pinyin",
                      "keep_first_letter":true,
                      "keep_separate_first_letter" : false,
                      "keep_full_pinyin" : true,
                      "keep_original" : false,
                      "keep_joined_full_pinyin":true,
                      "keep_none_chinese_in_joined_full_pinyin" : true ,
                      "limit_first_letter_length" : 16,
                      "lowercase" : true
                  },
                "completion_pinyin" : {
                      "type" : "pinyin",
                      "keep_first_letter":true,
                      "keep_separate_first_letter" : false,
                      "keep_full_pinyin" : false,
                      "keep_original" : true,
                      "keep_joined_full_pinyin":true,
                       "keep_none_chinese_in_joined_full_pinyin" : true ,
                      "limit_first_letter_length" : 16,
                      "lowercase" : true
                  },
                  
                "hanzi_pinyin_analyzer" : {
                      "type" : "pinyin",
                      "keep_first_letter":false,
                      "keep_none_chinese" : false,
                      "keep_full_pinyin" : false,
                      "keep_original" : false,
                      "keep_none_chinese_in_first_letter" : false,
                      "keep_joined_full_pinyin":true,
                       "keep_none_chinese_in_joined_full_pinyin" : true ,
                      "limit_first_letter_length" : 16,
                      "lowercase" : true
                  }
            }
            }
        }
}
```