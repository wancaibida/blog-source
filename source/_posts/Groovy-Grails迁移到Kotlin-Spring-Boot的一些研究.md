title: Groovy+Grails迁移到Kotlin+Spring Boot的一些研究
author: 大丈夫没问题
tags:
  - kotlin
  - java
  - groovy
  - grails
  - spring
categories:
  - java
date: 2018-08-16 13:35:00
---
Kotlin是静态语言，可以运行在JVM上且与java百分百可互操作。自从Google将Kotlin正式做为Android开发的语言后，Kotlin的一下子就火了起来。Kotlin不仅可以做为Android的首选开发语言，而且它也非常适合服务端的开发。

## 为什么选择Kotlin

### 与Java百分百可互操作groovy-grails-to-kotlin-springbootgroovy-grails-to-kotlin-springboot

Kotlin可以使用JVM平台上的任何库与框架，Kotlin编写的代码也可以供Java调用

### 简洁

Kotlin的代码非常简洁:

#### 变量

变量声明：

```
val a: Int = 1  // immediate assignment
val b = 2   // `Int` type is inferred
val c: Int  // Type required when no initializer is provided
c = 3       // deferred assignment
```

可变变量：

```
var x = 5 // `Int` type is inferred
x += 1
```

可空值与不可空值：

```
val s: String? = null //May be null
val s2: String = "" //May not be null
```

#### 方法

```
fun sum(a: Int, b: Int): Int {
    return a + b
}

//Function with an expression body and inferred return type:
fun sum(a: Int, b: Int) = a + b
```

扩展函数

```
fun String.lastChar(): Char = this.get(this.length - 1)

>>> println("Kotlin".lastChar())
n
```

infix函数

```
infix fun Any.to(other: Any) = Pair(this, other)

val (number, name) = 1 to "one"
```

#### 类

```
class Greeter(val name: String) {
   fun greet() { 
      println("Hello, $name")
   }
}

fun main(args: Array<String>) {
    val greeter = Greeter("John")
    greeter.greet()
    println(greeter.name)
}
```

数据类（data class）

自带equals hashcode tostring copy等方法

```
data class User(val name: String, val age: Int)
```

### DSL(Domain-specific language)

```
open class Tag(val name: String) {
    private val children = mutableListOf<Tag>()

    protected fun <T : Tag> doInit(child: T, init: T.() -> Unit) {
        child.init()
        children.add(child)
    }

    override fun toString() = "<$name>${children.joinToString("")}</$name>"
}

class TABLE : Tag("table") {
    fun tr(init: TR.() -> Unit) = doInit(TR(), init)
}

class TR : Tag("tr") {
    fun td(init: TD.() -> Unit) = doInit(TD(), init)
}

class TD : Tag("td")

fun table(init: TABLE.() -> Unit) = TABLE().apply(init)

fun createTable() = table {
    tr {
        td {

        }
    }
}

>>> println(createTable())
<table><tr><td></td></tr></table>

```

### 空安全

空安全(null safe)可以说是kotlin杀手级的特性了,它能排除一切空指针的可能，让你写出更安全的代码。

如下面这段代码会直接编译不通过：

```
var output: String
output = null   // Compilation error
```

```
val name: String? = null    // Nullable type
println(name.length())      // Compilation error
```

自动类型转换：

```
fun calculateTotal(obj: Any) {
    if (obj is Invoice)
        obj.calculateTotal()
}
```

### 静态类型(Statically Typed)

动态语言是一把双刃剑，groovy的代码非常简洁，弥补了java语言太啰嗦的缺点，同时groovy的闭包也非常好，正因为如此我们选择了groovy。但时做为动态语言，许多行为是在代码运行时才会确定，编译错误在写代码时无法检测到，除非加上`@TypeChecked`注解。

Kotlin的代码非常简洁，更重要的是它是静态类型语言，能检测到编译错误，减少了代码中潜在的问题。

## 为什么选择Spring Boot

- 从2015年初开始，Sring Boot的热度就超过了Grails,可参考:[Grails && Spring Boot Stack Overflow Trends](https://insights.stackoverflow.com/trends?tags=grails%2Cspring-boot)
- Grails 2.x版本有上千个插件，但升级到Grails 3.x之后，这些插件就不再被支持，更糟糕的是没有人将这些插件迁移到Grails 3.x，Grails 3.x现在只有244个插件。
- 尽管Grails 3是基于Spring Boot的，但其自身也带来了些奇怪的bug, 有时候解决框架bug的时间甚至超过了使用框架节省的时间。

## Kotlin+Spring Boot的一些不足

### 没有Gorm查询

Grails框架的最大亮点是Gorm：动态方法和DSL查询:

```
//will be translated to hql automatically
def person = Person.findByFirstName('xxx')
equals:
"FROM Person WHERE firstname = ?"

Person.withCretia {
 or {
      ilike('name','%j%')
      gte('age',22)
 }
}
equals:
"FROM person WHERE name ilike ? OR age >= ?"
```

在Spring Boot中你只能自己写查询语句或者使用类似[spring data jpa query dsl](https://docs.spring.io/spring-data/jpa/docs/2.0.8.RELEASE/reference/html/#core.extensions)插件:

```
internal interface TodoRepository : Repository<Todo, Long> {

    fun findFirst3ByTitleOrderByTitleAsc(title: String): List<Todo>

    fun findTop3ByTitleOrderByTitleAsc(title: String): List<Todo>
}

```

### 数据库迁移（Database Migration）

Grails 的[database migration](https://plugins.grails.org/plugin/grails/database-migration)在数据库表结构和数据迁移两方面都非常强，它可以对比项目中的实体和数据库表来自动生成表结构修改语句，同时你可以写sql来完成数据迁移。

```
changeSet(author: "xxx (generated)", id: "1521527665533-1") {
        addColumn(tableName: "staff") {
            column(name: "date_created", type: "TIMESTAMP WITHOUT TIME ZONE") {
                constraints(nullable: "true")
            }
        }
    }

changeSet(author: "xxxx", id: "1521528248") {
        grailsChange {
            change {
                sql.execute('''
                            UPDATE staff
                            SET date_created = now()
                         ''')
            }
        }
    }
```

而在Spring Boot方面，数据库迁移插件有flyway和liquibase,两者各有千秋，flyway在数据迁移上比较强，但缺少自动生成表结构修改语句功能。liquibase正好相反，它有自动生成表结构修改语句功能，但在写数据迁移sql方面比较弱。

### 表单校验

在自定义校验方法上Grails非常方便，只需要一个闭句就可以了：

```
class User {
    String name
    static constraints = {
        name unique: true, validator: { val, obj ->

            if (!val.matches('[A-Z0-9-]+')) {
                return 'acronym.format.error'
            }

            true
        }
    }
}
```

而在Spring Boot方面比如用hiberate validator你需要定义类，定义注解，写很多代码来实现自定义校验。

## 总结

在项目开发速度上，Groovy+Grails上占优，如果时间比较紧的话，Groovy+Grails会是一个不错的选择，它能在较短时间内做出项目。如果你需要一个bug少，更稳定的项目或者你的项目用不到Gorm的话，你可以选择Kotlin+Spring Boot.

简而言之，100分做为满分的话，Groovy+Grails能在最短的时间内达到80分，而要达到90甚至95分的话，选择Kotlin+Spring Boot无疑。

