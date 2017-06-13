title: Hibernate学习
author: Gang Chen
tags:
  - Hibernate
categories:
  - Java
date: 2017-06-13 14:25:00
---
### 占位符（named parameters）对order by无效
```
//对于最后两字段sortField和sortOrder设值时是没有效果的
SELECT * FROM t_table WHERE column=:param1 ORDER BY :sortField :sortOrder
```

### isDirty细节
```
def product = Product.get(xxx)
product.name = 'anotherName'
product.save()
product.isDirty() //true

def product = Product.get(xxx)
product.name = 'anotherName'
product.save(flush:true)
product.isDirty() //false
```
hibernate 中entity get时会有个loadstate 来记录entity的初始状态,调用isDirty方法将比较当前 entity和loadstate来判断entity是否发生改变,然而当flush:true时,将刷新loadstate状态,从时调用isDirty将反回true.

### Session Flush
* HQL查询会导致session flush
所以在调用isDirty前进行HQL查询会导致isDirty始终返回false

### flush与transaction的联系
flush后Hibernate会生成SQL语句并执行，但只有环绕的transaction提交后才会真正的同步到数据库