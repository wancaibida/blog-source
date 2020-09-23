title: SQL分页性能优化
author: 大丈夫没问题
tags:
  - jdbc
  - sql
  - pagination
categories:
  - database
date: 2020-09-23 11:17:00
---
JDBC的分页是通过offset+limit实现的，数据库是通过抓取所有数据并跳过前面n行来数据来实现offset功能的，越到后面跳过的数据越多offset越耗时:

```
CREATE TABLE players AS
SELECT generate_series(1, 10000000)                                     AS id,
       uuid_in(md5(random()::TEXT || clock_timestamp()::TEXT)::CSTRING) AS username,
       floor(random() * 100)                                            AS score;

CREATE UNIQUE INDEX players_id_uindex
    ON players (id);
```

OFFSET + LIMIT:

```
SELECT * FROM players ORDER BY id OFFSET 10 LIMIT 10; // Execution Time: 0.034 ms
SELECT * FROM players ORDER BY id OFFSET 500000 LIMIT 10; // Execution Time: 115.706 ms
SELECT * FROM players ORDER BY id OFFSET 1000000 LIMIT 10; // Execution Time: 235.941 ms
SELECT * FROM players ORDER BY id OFFSET 5000000 LIMIT 10; // Execution Time: 1212.642 ms
SELECT * FROM players ORDER BY id OFFSET 9000000 LIMIT 10; // Execution Time: 2181.703 ms
```


解决办法就是用`keyset pagination`，用表里面一些唯一的列用来分页：

```
SELECT * FROM players WHERE id > 10 ORDER BY id LIMIT 10; // Execution Time: 0.033 ms
SELECT * FROM players WHERE id > 500000 ORDER BY id LIMIT 10; // Execution Time: 0.043 ms
SELECT * FROM players WHERE id > 1000000 ORDER BY id LIMIT 10; // Execution Time: 0.041 ms
SELECT * FROM players WHERE id > 5000000 ORDER BY id LIMIT 10; // Execution Time: 0.037 ms
SELECT * FROM players WHERE id > 9000000 ORDER BY id LIMIT 10; // Execution Time: 0.069 ms
```

实际情况中可能会以score排序：

创建索引
```
CREATE INDEX players_score_username_index ON players (score DESC, username DESC);
```

OFFSET + LIMIT
```
SELECT * FROM players ORDER BY score DESC, username DESC OFFSET 500000 LIMIT 10; //Execution Time: 1508.534 ms
```

Keyset pagination

```
SELECT * FROM players WHERE (score, username) < (94, 'fdf62b1a-5b4e-c561-38ad-74ac4f099a46') ORDER BY score DESC, username DESC LIMIT 10; //Execution Time: 0.077 ms


(score, username) < (94, 'fdf62b1a-5b4e-c561-38ad-74ac4f099a46')

// 效果等同于
score < 94 OR (score = 94 AND username < 'fdf62b1a-5b4e-c561-38ad-74ac4f099a46')
```

## 进阶

找出分页边界，这里的关键点是你要找出一列或者多列组成的唯一的列，如果你只按score排序的话，score会有重复的值，会导致排序结果不一致。

```
SELECT p.id,
       p.username,
       p.score,
       CASE row_number()
            OVER (ORDER BY p.score DESC , p.username DESC ) % 10
           WHEN 0 THEN 1
           ELSE 0
           END page_boundary
FROM players AS p
ORDER BY p.score DESC, p.username DESC
```
查询结果：

| id | username | score | page\_boundary |
| :--- | :--- | :--- | :--- |
| 6603631 | ffffa267-7102-d2e9-0316-572c0044ca4f | 99 | 0 |
| 3647464 | ffff44d0-f54a-9139-afb0-f3d188c79a77 | 99 | 0 |
| 422584 | ffff378c-b897-2498-8133-9d88183e52ec | 99 | 0 |
| 6213135 | fffe9b37-c685-4d5f-7781-31489e3b04ea | 99 | 0 |
| 7930940 | fffbde40-2009-8faf-4423-d28116f8dfee | 99 | 0 |
| 1078914 | fffb2654-23d1-e7f4-cf83-d4298b3c7208 | 99 | 0 |
| 9984059 | fffb0de3-366c-5d42-ce61-9f7ee197707b | 99 | 0 |
| 798388 | fffac15d-a418-83eb-9597-487aa0a9136b | 99 | 0 |
| 9247069 | fff8cea2-378d-7bb8-1954-3bc6e6971eee | 99 | 0 |
| 1711860 | fff79537-86b8-5bd1-a474-717f151e0448 | 99 | 1 |
| 4319563 | fff786e7-75b4-5cd7-09c1-eb4dc690dbbc | 99 | 0 |
| 6021293 | fff774fb-cf17-d02e-96b0-15555cb36b39 | 99 | 0 |
| 987629 | fff756f7-e76b-8e8b-9941-c39061dac542 | 99 | 0 |
| 149766 | fff75386-890f-af96-1c4c-64df0f6f4546 | 99 | 0 |
| 5546133 | fff6ebcb-f3fb-d4b3-6b33-b56c718d44e4 | 99 | 0 |
| 5167740 | fff6ce5a-b4c1-5363-554b-a285b6917054 | 99 | 0 |
| 5931868 | fff5d7a6-1b0d-cf87-e0df-2440a53a7e05 | 99 | 0 |
| 4577984 | fff55361-9b07-dfc6-9d1a-5fc08cdec7a6 | 99 | 0 |
| 4013672 | fff4c37a-7ddf-6042-6b8f-eb9d7f8ea691 | 99 | 0 |
| 7863996 | fff47c1f-18d8-9980-72fb-e51213ff0f17 | 99 | 1 |


找出每一页的最后一条数据
```
SELECT p.id,
       p.username,
       p.score,
       page_boundary,
       row_number()
       OVER (ORDER BY p.score DESC , p.username DESC) + 1 page_number
FROM (
         SELECT p.id,
                p.username,
                p.score,
                CASE row_number()
                     OVER (ORDER BY p.score DESC , p.username DESC ) % 10
                    WHEN 0 THEN 1
                    ELSE 0
                    END page_boundary
         FROM players AS p
         ORDER BY p.score DESC, p.username DESC
     ) AS p
WHERE p.page_boundary = 1;
```

查询结果：

| id | username | score | page\_boundary | page\_number |
| :--- | :--- | :--- | :--- | :--- |
| 1711860 | fff79537-86b8-5bd1-a474-717f151e0448 | 99 | 1 | 2 |
| 7863996 | fff47c1f-18d8-9980-72fb-e51213ff0f17 | 99 | 1 | 3 |
| 7016430 | ffe9a2c3-f257-24f2-e6f1-f7219c3581ea | 99 | 1 | 4 |
| 994965 | ffe61629-a620-7fcb-af3f-4c2c15821b77 | 99 | 1 | 5 |
| 9267095 | ffe1b897-24ac-61d0-a5c2-9103a43b4e77 | 99 | 1 | 6 |
| 936851 | ffda271d-5c69-1a4b-061f-077d75b77f49 | 99 | 1 | 7 |
| 9772528 | ffd521da-c0bf-9909-77a1-9bce8e2c39a6 | 99 | 1 | 8 |
| 3913178 | ffccfdeb-5608-c02e-6dd8-fe372245ee2b | 99 | 1 | 9 |
| 3757222 | ffc54f77-1f21-d63c-d660-a35910625bfe | 99 | 1 | 10 |
| 9150019 | ffbe0ffe-db67-44e2-2e84-41d0b54467b6 | 99 | 1 | 11 |


所以如果我们要查询第二页的数据，执行下面的sql即可：

```
SELECT * FROM players WHERE (score, username) < (99, 'fff79537-86b8-5bd1-a474-717f151e0448') ORDER BY score DESC, username DESC;
```

查询结果：

| id | username | score |
| :--- | :--- | :--- |
| 4319563 | fff786e7-75b4-5cd7-09c1-eb4dc690dbbc | 99 |
| 6021293 | fff774fb-cf17-d02e-96b0-15555cb36b39 | 99 |
| 987629 | fff756f7-e76b-8e8b-9941-c39061dac542 | 99 |
| 149766 | fff75386-890f-af96-1c4c-64df0f6f4546 | 99 |
| 5546133 | fff6ebcb-f3fb-d4b3-6b33-b56c718d44e4 | 99 |
| 5167740 | fff6ce5a-b4c1-5363-554b-a285b6917054 | 99 |
| 5931868 | fff5d7a6-1b0d-cf87-e0df-2440a53a7e05 | 99 |
| 4577984 | fff55361-9b07-dfc6-9d1a-5fc08cdec7a6 | 99 |
| 4013672 | fff4c37a-7ddf-6042-6b8f-eb9d7f8ea691 | 99 |
| 7863996 | fff47c1f-18d8-9980-72fb-e51213ff0f17 | 99 |

OFFSET + LIMIT:

```
SELECT * FROM players ORDER BY score DESC, username DESC OFFSET 10 LIMIT 10;
```

查询结果：

| id | username | score |
| :--- | :--- | :--- |
| 4319563 | fff786e7-75b4-5cd7-09c1-eb4dc690dbbc | 99 |
| 6021293 | fff774fb-cf17-d02e-96b0-15555cb36b39 | 99 |
| 987629 | fff756f7-e76b-8e8b-9941-c39061dac542 | 99 |
| 149766 | fff75386-890f-af96-1c4c-64df0f6f4546 | 99 |
| 5546133 | fff6ebcb-f3fb-d4b3-6b33-b56c718d44e4 | 99 |
| 5167740 | fff6ce5a-b4c1-5363-554b-a285b6917054 | 99 |
| 5931868 | fff5d7a6-1b0d-cf87-e0df-2440a53a7e05 | 99 |
| 4577984 | fff55361-9b07-dfc6-9d1a-5fc08cdec7a6 | 99 |
| 4013672 | fff4c37a-7ddf-6042-6b8f-eb9d7f8ea691 | 99 |
| 7863996 | fff47c1f-18d8-9980-72fb-e51213ff0f17 | 99 |







# 参考
* https://vladmihalcea.com/resultset-statement-fetching-with-jdbc-and-hibernate/
* https://use-the-index-luke.com/no-offset
* https://uxdesign.cc/why-facebook-says-cursor-pagination-is-the-greatest-d6b98d86b6c0
* https://medium.com/swlh/how-to-implement-cursor-pagination-like-a-pro-513140b65f32
* https://www.postgresql.org/docs/10/tutorial-window.html
* https://blog.jooq.org/2013/10/26/faster-sql-paging-with-jooq-using-the-seek-method
* https://www.citusdata.com/blog/2016/03/30/five-ways-to-paginate/
* https://blog.jooq.org/2013/11/18/faster-sql-pagination-with-keysets-continued/