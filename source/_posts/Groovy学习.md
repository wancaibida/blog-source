title: Groovy学习
author: Gang Chen
tags:
  - hibernate
  - groovy
categories:
  - java
  - ''
date: 2017-06-13 14:03:00
---
### sort方法作用于Set和List时的区别
```
        def a = new HashSet()
        a << [3, 1, 2]

        def b = new ArrayList()
        b << [2, 1, 3]

        assert b.sort().is(b) //true
        assert a.sort().is(a) //false
```

### sum and flatten 作用于Collection时的区别
```
[].sum() returns null
[].flatten() returns []
//原因在于调用sum时默认initvalue为null,当list为空时直接返回initvalue
private static Object sum(Iterable self, Object initialValue, boolean first) {
    Object result = initialValue;
    Object[] param = new Object[1];
    for (Object next : self) {
        param[0] = next;
        if (first) {
            result = param[0];
            first = false;
            continue;
        }
        MetaClass metaClass = InvokerHelper.getMetaClass(result);
        result = metaClass.invokeMethod(result, "plus", param);
    }
    return result;
}
```

### @EqualsAndHashCode注解
#### 当exclude类所有field及property时，会导制所有实例都相等
```
    @EqualsAndHashCode(excludes = ['x', 'y'])
    static class Point {
        int x
        int y
    }

    public static void main(String[] args) {
        def point0 = new Point(x: 1, y: 1)
        def point1 = new Point(x: 2, y: 2)
        assert point0 == point1
    }
```
#### 对ORM类慎用该注解，可能会导制关联丢失，当mutable字段变化时，会导制hash值改变，导制无法定位到槽位，
```
    @EqualsAndHashCode
    static class Point {
        int x
        int y
    }

    public static void main(String[] args) {
        def point0 = new Point(x: 1, y: 1)
        def point1 = new Point(x: 2, y: 2)
        def set = new HashSet()
        set << point0
        set << point1

        assert set.contains(point0)

        point0.x = 6
        assert set.contains(point0)  //false !!!
    }
```