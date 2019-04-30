# ElasticSearch

## 实体类

```java {.line-numbers}
package com.bootdo.modules.elasticsearch.entity;

import com.alibaba.fastjson.annotation.JSONField;
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
    //@JsonFormat(shape = JsonFormat.Shape.STRING, pattern ="yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    @JSONField(format="yyyy-MM-dd HH:mm:ss")
    private Date createTime;

    @Field(type = FieldType.Text, store = true)
    private String nameEn;

    @Field(type = FieldType.Text, store = true, analyzer = "ik_max_word")
    private String nameCh;

}

```

## 批量插入bulIndex

该方法比ElasticSearch-Repository速度快了将近10倍

代码如下：

```java {.line-numbers}
/**
     * EsTemplate批量插入测试
     */
    @Test
    public void test5() {

        List<StkosConceptGroup> list = conceptGroupDao.queryList(null);

        if (!esTemplate.indexExists(EsConstant.BOOTDO_INDEX)) {
            esTemplate.createIndex(EsConstant.BOOTDO_INDEX);
        }

        long count = 0;

        List<IndexQuery> indexQueries = new ArrayList<>(10);
        long start = System.currentTimeMillis();
        for (StkosConceptGroup t : list) {
            IndexQuery indexQuery = new IndexQuery();
            indexQuery.setId(t.getConceptGroupId());
            indexQuery.setIndexName(EsConstant.BOOTDO_INDEX);
            indexQuery.setType(EsConstant.BOOTDO_TYPE);
            indexQuery.setSource(JSON.toJSONString(t));
            indexQueries.add(indexQuery);
            if (indexQueries.size() == 1000) {
                esTemplate.bulkIndex(indexQueries);
                indexQueries.clear();
                log.info(String.format("index StkosConceptGrop %d", count));
            }
            count++;
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