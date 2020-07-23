title: Google Cloud BigQuery Create Partition Table From Existing Table
author: 大丈夫没问题
tags:
  - google cloud
categories:
  - linux
date: 2020-07-23 20:35:00
---
```
create table [schema-name].copy
partition by date([timestamp-column])
as select * from [schema-name].[table-name];
drop table [schema-name].[table-name];
```