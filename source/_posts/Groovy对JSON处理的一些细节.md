title: Groovy JSON处理的一些细节
author: 大丈夫没问题
tags:
  - java
  - groovy
categories:
  - java
date: 2018-10-30 09:19:00
---
最近遇到一项目，需要在手机端存储用户数据，实现离线访问。其中用户数据处理的逻辑如下图:

![User data flow](https://www.websequencediagrams.com/cgi-bin/cdraw?lz=dGl0bGUgVXNlciBkYXRhIHVwZGF0ZSBmbG93CgpTZXJ2ZXItPlMzOiBEb3dubG9hZCB1c2VyIGpzb24gZmlsZQAaCgAnBTogRGVzZXJpYWxpemUAHA8gdG8gbWFwACERVQBtBgBTBW1hcACBAwUARhFTAEcOACAIIHRvAHgUMzogVXAAgRsTIHRvIFMzCgo&s=modern-blue)

1. 服务端从亚马逊S3上下载用户JSON文本数据库

2. 反序列化用户数据

3. 更新用户数据

4. 将用户数据序列化为JSON文本

5. 保存到亚马逊S3上

由于项目设计缺陷，用户所有的数据都存储一个Map对象里，导致Map对象过大，在项目运行过程中出现了内存不足的异常。为了解决内存不足问题，服务端采用了[JacksonStreamingApi](https://github.com/FasterXML/jackson-docs/wiki/JacksonStreamingApi)优化了JSON序列及反序列化步骤，避免将整个用户文件载入到内存中，至此内存不足的异常就再也没有发生。

其实这里有个问题，如果是用户数据过大，内存不足异常会在步骤3结束后就会发生，为什么偏偏在步骤4序列化为JSON时抛出呢？这里就要说到 Groovy的[LazyMap](http://docs.groovy-lang.org/2.4.3/html/api/groovy/json/internal/LazyMap.html)了。

LazyMap代码：

```java
public class LazyMap extends AbstractMap<String, Object> {

    static final String JDK_MAP_ALTHASHING_SYSPROP = System.getProperty("jdk.map.althashing.threshold");

    /* Holds the actual map that will be lazily created. */
    private Map<String, Object> map;
    /* The size of the map. */
    private int size;
    /* The keys  stored in the map. */
    private String[] keys;
    /* The values stored in the map. */
    private Object[] values;

    public LazyMap() {
        keys = new String[5];
        values = new Object[5];
    }
    ...
    
    public Object put(String key, Object value) {
        if (map == null) {
            for (int i = 0; i < size; i++) {
                String curKey = keys[i];
                if ((key == null && curKey == null)
                     || (key != null && key.equals(curKey))) {
                    Object val = values[i];
                    keys[i] = key;
                    values[i] = value;
                    return val;
                }
            }
            keys[size] = key;
            values[size] = value;
            size++;
            if (size == keys.length) {
                keys = grow(keys);
                values = grow(values);
            }
            return null;
        } else {
            return map.put(key, value);
        }
    }

    public Object get(Object key) {
        buildIfNeeded();
        return map.get(key);
    }

    private void buildIfNeeded() {
        if (map == null) {
            // added to avoid hash collision attack
            if (Sys.is1_8OrLater() || (Sys.is1_7() && JDK_MAP_ALTHASHING_SYSPROP != null)) {
                map = new LinkedHashMap<String, Object>(size, 0.01f);
            } else {
                map = new TreeMap<String, Object>();
            }

            for (int index = 0; index < size; index++) {
                map.put(keys[index], values[index]);
            }
            this.keys = null;
            this.values = null;
        }
    }
  }
```

从代码中可以看出：对于未进行过读操作(get,containsKey等)的LazyMap对象，keys和values分别存在了两个数组中，一旦调用了读取方法，LazyMap会将数组转化成Map对象，就是这一步操作引起了内存占用变化。

拿大小约为35MB的用户文件测试，反序列化后，内存中LazyMap对象为73MB(line 1)，一旦对对象进行`toJson`操作(line 2)，内存占用上升到了559MB(line 3)。

```groovy
    public static void main(String[] args) {
        File jsonFile = new File('data-format.json')
        Object obj = new JsonSlurper().parse(jsonFile) // LazyMap here
        showSize(obj) // line 1
        JsonOutput.toJson(obj) // line 2
        showSize(obj) // line 3
    }

    static void showSize(obj) {
        def size = ObjectSizeCalculator.getObjectSize(obj) / 1024 / 1024
        println("Object memory size is $size MB")
    }

// output logs:
Object memory size is 73.51363372802734375 MB
Object memory size is 559.32244873046875 MB
```

由于用户数据文件较大且嵌套了多层Map，加之`JsonOutput.toJson`方法会遍历LazyMap对象所有节点，相当于对所有节点进行了读操作，导致节点中的数组转换成Map对象，最终引起内存不足异常。