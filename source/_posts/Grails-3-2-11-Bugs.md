title: Grails 3.2.11 Bugs
author: Gang Chen
tags:
  - grails
  - groovy
categories:
  - java
date: 2018-04-01 21:30:00
---
* Functional Testing setup无法回滚

```
class BaseFunctionalTest extends GebSpec {
    def setup() {
        Dog dog = new Dog(name:'xxxx')
    }

    //需要手动删除
    def cleanup() {
        Dog.findByName('xxxx')?.delete()
    }
}


```

* JSON Views循环渲染bug

如果多个GORM实体存在循环引用，则会产生stackover flow异常，如下代码：

```
Class A {
    static hasMany = [children: B]
}

Class B {
    A parent
}

def a = new A()
def b = new B(parent:a)
a.addToChildren(b)

json g.render(a, [deep: true])
```
