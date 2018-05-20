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

* isDirty方法实际上时通过检测类属性指向的引用是否改变来实现的，假如是属性内部的属性发生改变，isDirty方法实际是检测不到的。如：


```
class Person{
    String firstName
    String lastName
    Address address
}

class Address{
    String city
    String street
}

Address address = new Address('shanghai','street')
Person person = new Person('fname','lname',address)

person.save()
person.firstName = 'xxxx'
person.isDirty() == true
person.isDirty('firstName') == true

person.save()
address.city = 'beijing'
person.isDirty('address') == false
person.isDirty() == false

person.address.isDirty() == true
person.address.isDirty('city') == true
```

* build test data plugin 不支持自定义约束的属性，就算在`TestDataConfig.groovy`中设置了也没用，如：

```
testDataConfig {
    sampleData {
        Dog {
            name = { ->
                UUID.randomUUID().toString().toUpperCase().replace('-', '')
            }
        }
    }
}

Class Dog {
    String owner
    String name
    
    static constraints = {
            name validator: { val, obj ->
            ...
        }
    } 
}

Dog dog = Dog.build()
dog.name == 'name'
```
* one-to-many关联中，用left join的方式从many的一方查找出来one，one关联的many对象不正确，如：

```
class Person {
    static hasMany = [dogs:Dog]
}

class Dog {
    String name
}

Dog a = new Dog('A')
Dog b = new Dog('B')

Person p = new Person('P')
person.addToDogs(a)
person.addToDogs(b)
person.save()

Collection<Person> result = Person.withCriteria {
    createAlias('dogs', 'd', org.hibernate.sql.JoinType.LEFT_OUTER_JOIN)
    eq('d.name', 'A')
}

result[0].dogs.size() == 1

result = Person.withCriteria {
    dogs {
        eq('name','A')
    }
}

result[0].dogs.size() == 2
```

