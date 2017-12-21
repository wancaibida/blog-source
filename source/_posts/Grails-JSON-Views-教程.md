title: Grails JSON Views 教程
author: Gang Chen
tags:
  - grails
  - json
  - view
  - json-views
categories:
  - Java
  - ''
date: 2017-12-21 13:21:00
---
Grails 3.2版本中的rest-api profile加入了JSON View插件，JSON View插件主要用于渲染JSON返回内容，类似于GSP，其好处就是将JSON渲染从控制器层移到了视图层，同时JSON View还能定义模板，继承等。

## 开始使用
### 创建视图
JSON View视图文件以`.gson`为扩展名，且文件要放在`grails-app/views`目录下。
例：person.gson

```
json.person {
    name "bob"
}
```
返回的JSON值为：

```
{"person":{"name":"bob"}}
```
### 创建模版
模版文件需要以`_`开头，比如你有一个类名叫QueryResult,则其模版文件完整路径为`grails-app/views/queryResult/_queryResult.gson`
例：`Author.groovy`

```
class Author {
    String name
}
```
模版：`grails-app/views/author/_author.gson`

```
model {
    Author author
}
json {
    name author.name
}
```
也可以简写为：

```

@Field Author author
json {
    name author.name
}
```
如果Author类是Domain Object的话，则模版文件可以写成：


```
@Field Author author
json g.render(author)
```

### 高级用法
#### 自定义字段

```
model {
    Book book
}

json g.render(book, [deep: false, renderNulls: true]) {
    authorName book.author?.name
    publishDate new Date().time
}

```
#### 集合渲染
例1：

```
model {
    Author author
}

json {
    name author.name
    books g.render(template: '/book/book', collection: author.books, var: 'book')
}
```
`/book/book`引用的是`grails-app/views/book`目录下名为`_book.gson`的模版文件

例2：
如果返回结果是个集合，

```
class AuthorController {
    static responseFormats = ['json', 'xml']

    def index() {
        /**
         * 这里需要注意res的类型，
         * 如果是List类型，则json view中的model名称则为authorList
         * 如果是Set类型，则json view中的model名称则为authorSet
         */
        def res = Author.list()
        respond(res, [view: 'authorList'])
    }
}

```
* **这里要注意下`res`的类型**
    * 如果是List类型，则json view中的model名称则为authorList
    * 如果是Set类型，则json view中的model名称则为authorSet

* Grails默认会在`grails-app/views/author/`路径下查找`authorList.gson`视图文件
* 如果需要在其他Controller下引用该视图的话，需要写绝对路径`/author/authorList`

`_author.gson`

```
model {
    Author author
}

json {
    name author.name
    books g.render(template: '/book/book', collection: author.books, var: 'book')
}

```
`authorList.gson`

```
model {
    Collection<Author> authorList = []
}

json tmpl.author(authorList)

```
**authorList要指定下默认值`[]`，如果不指定的话会返回`[null]`这种JSON**

#### 渲染参数
你可以通过`includes`和`excludes`参数，来包含或排除一些字段，如

```
@Field Author author
json g.render(author，[includes:['title'])
```
你也可以自定义额外输出的内容：

```
json g.render(author) {
    age 30
}
```
在默认情况下，JSON View不会返回值为`null`的字段，如：

```
class Author {
    String name
    Integer age
}

@Field Author author
json g.render(author)

def a = new Author(name: 'charles',age: 12) => {name:'chalres',age: 12}
def b = new Author(name: null,age: 29)=> {age: 29}
```
这时候你可以设直`renderNulls`来返回空值字段：

```
@Field Author author
json g.render(author,[renderNulls: true])
```
则b的返回中就变成：

```
def b = new Author(name: null,age: 29)=> {name: null,age: 29}

```

### 完整的例子
[https://github.com/wancaibida/grails-json-views-example](https://github.com/wancaibida/grails-json-views-example)

### 总结
JSON View用下来还是遇到了不少的问题，主要原因是其约定太多且官方的文档也说的不是很清楚，有 些地方还是要一步步调试来研究源代码，但总的来说比传统的在代码中指定JSON格式方便了不少，用好了能节约不少时间与工作量。

#### 参考链接
https://github.com/skyboy101/widget-store-rest-api
http://views.grails.org/1.1.x/