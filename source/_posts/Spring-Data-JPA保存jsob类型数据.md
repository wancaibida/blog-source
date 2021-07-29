title: Spring Data JPA保存jsonb类型数据
author: 大丈夫没问题
tags:
  - spring
  - jpa
  - jsonb
categories: []
date: 2021-07-29 21:10:00
---

## 创建表
```
CREATE TABLE test
(
    id        integer NOT NULL
        CONSTRAINT test_pk
            PRIMARY KEY,
    test_data jsonb
);
```

## 定义方法

```
    @Modifying
    @Query(
        value = "INSERT INTO test (id, test_data) VALUES (:id, CAST(:testData AS JSONB))",
        nativeQuery = true
    )
    fun saveTest(
        @Param("id") id: Long,
        @Param("testData") testData: String
    ): Int
```

## 调用代码
```
        val id = 1L
        val testData = listOf("1", "2", "3")
        val testDataJsonStr = ObjectMapper().writeValueAsString(testData)

        transactionTemplate.execute {
            repository.saveTest(id, testDataJsonStr)
        }
```