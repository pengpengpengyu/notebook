# ElasticSearchTemplate

## 实体类

```java {.line-numbers}
package com.bootdo.modules.elasticsearch.entity;

import com.bootdo.common.Constant.EsConstant;
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.DateFormat;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;
import java.io.Serializable;
import java.util.Date;

/**
 * @ClassName stkos_concept_group
 * @Description ESConceptGroupEntity
 * @Author pywang
 * @Date 2019-04-29 11:01:13
 * @Version 0.0.1-SNAPSHOT
 */
@Data
@ToString
@AllArgsConstructor
@NoArgsConstructor
@Document(indexName = EsConstant.BOOTDO_INDEX, type = EsConstant.BOOTDO_TYPE)
public class StkosConceptGroup implements Serializable {
    private static final long serialVersionUID = 4378257088766250398L;

    @Id
    @Field(type = FieldType.Keyword, store = true)
    private String conceptGroupId;

    @Field(type = FieldType.Keyword, store = true)
    private String notation;

    /**
     * N or Y
     */
    @Field(type = FieldType.Keyword, store = true)
    private String topConceptGroup;

    /**
     * ispartof
     */
    @Field(type = FieldType.Keyword, index = false, store = true)
    private String kosid;

    @Field(type = FieldType.Keyword, store = true)
    private String hasConceptGroupLabelEn;

    @Field(type = FieldType.Keyword, store = true)
    private String hasConceptGroupLabelCh;

    @Field(type = FieldType.Date, format = DateFormat.custom, pattern = "yyy-MM-dd HH:mm:ss", store = true)
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern ="yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createTime;

    @Field(type = FieldType.Text, store = true)
    private String nameEn;

    @Field(type = FieldType.Text, store = true, analyzer = "ik_max_word")
    private String nameCh;

}

```

## 批量插入bulkIndex

该方法比ElasticSearch-Repository速度快了将近10倍

代码如下：

```java {.line-numbers}
 /**
     * EsTemplate批量插入测试
     */
    @Test
    public void test5() throws Exception {

        List<StkosConceptGroup> list = conceptGroupDao.queryList(null);

        if (!esTemplate.indexExists(EsConstant.BOOTDO_INDEX)) {
            esTemplate.createIndex(EsConstant.BOOTDO_INDEX);
        }

        long count = 0;

        List<IndexQuery> indexQueries = new ArrayList<>(10);
        long start = System.currentTimeMillis();
        for (StkosConceptGroup t : list) {
            count++;
            // 通过new的方式构建IndexQuery
            /*IndexQuery indexQuery = new IndexQuery();
            indexQuery.setId(t.getConceptGroupId());
            indexQuery.setIndexName(EsConstant.BOOTDO_INDEX);
            indexQuery.setType(EsConstant.BOOTDO_TYPE);
            indexQuery.setSource(new ObjectMapper().writeValueAsString(t));*/

            // 直接构建IndexQuery(建议使用)
            IndexQuery indexQuery = new IndexQueryBuilder()
                    .withIndexName(EsConstant.BOOTDO_INDEX)
                    .withType(EsConstant.BOOTDO_TYPE)
                    .withId(t.getConceptGroupId())
                    .withObject(t)
                    .build();
            indexQueries.add(indexQuery);


            if (indexQueries.size() == 10000) {
                esTemplate.bulkIndex(indexQueries);
                indexQueries.clear();
                log.info(String.format("index StkosConceptGrop %d", count));
            }

        }

        if (!indexQueries.isEmpty()) {
            esTemplate.bulkIndex(indexQueries);
            indexQueries.clear();
            log.info(String.format("bulkIndex StkosConceptGrop %d", count));
        }
        esTemplate.refresh(EsConstant.BOOTDO_INDEX);
        log.info("bulkIndex StkosCncept Completed. total time {}", System.currentTimeMillis() - start);
    }
```

结果如下：

``` {.line-numbers, .highlight=14}
10:52:06 [main] INFO  com.bootdo.testDemo.TestDemo - index StkosConceptGrop 10000
10:52:06 [main] INFO  com.bootdo.testDemo.TestDemo - bulkIndex StkosConceptGrop 12221
10:52:07 [main] INFO  com.bootdo.testDemo.TestDemo - bulkIndex StkosCncept Completed. total time 1553
```

## 删除

```java {.line-numbers}
 /**
     * 删除
     */
    @Test
    public void test6() {

        // 删除某条记录（按id）
        esTemplate.delete(EsConstant.BOOTDO_INDEX, EsConstant.BOOTDO_TYPE, "47115410");

        // 删除某个Index某Type下的所有数据
        DeleteQuery deleteQuery = new DeleteQuery();
        deleteQuery.setIndex("user");
        deleteQuery.setType("doc");
        esTemplate.delete(deleteQuery);

        // 删除Index下的所有数据
        esTemplate.deleteIndex("user");

        // 删除查询出来的说有数据
        deleteQuery = new DeleteQuery();
        deleteQuery.setQuery(QueryBuilders.matchQuery("id", "100"));
        esTemplate.delete(deleteQuery);
    }
```

## 更新

```java {.line-numbers}
/**
     * 更新
     */
    @Test
    public void test7(){
        StkosConceptGroup stkosConceptGroup = conceptGroupDao.queryById("47115421");
        UpdateQuery updateQuery = new UpdateQueryBuilder()
                .withIndexName(EsConstant.BOOTDO_INDEX)
                .withType(EsConstant.BOOTDO_TYPE)
                .withClass(stkosConceptGroup.getClass())
                .withId(stkosConceptGroup.getConceptGroupId())
                // 要修改的字段
                .withIndexRequest(new IndexRequest().source("nameCh", stkosConceptGroup.getNameCh() + "1111"))  
                .build();
        esTemplate.update(updateQuery);
    }
```

更多资料请点击该播客地址[ElasticSearchTemplate技术播客](https://www.cnblogs.com/yueshutong/p/9381543.html)