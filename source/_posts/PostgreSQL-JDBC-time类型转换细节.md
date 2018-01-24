title: PostgreSQL JDBC time类型存取细节
author: Gang Chen
tags:
  - postgresql
categories:
  - database
  - ''
date: 2017-06-14 22:58:00
---
* PostgreSQL中默认时区为UTC
* record表中有一类型为Time without timezone的字段event_time
* 本机的时区为UTC+8

现象:
* 在页面提交的时间为 15:00:00+0
* 到了数据库时间却显示为 23:00:00+0
* 查询出来后页面显示的时间是 15:00:00+0
* 控制台打印出来的SQL如下:
```
insert into record (id, event_time) values (nextval ('hibernate_sequence'), '23:00:00.000000 +08:00:00')
```

原因:
查看源代码,发现在生成insert语句时,JDBC会通过toString方法将Time类型转成String,而问题就出现在Time转String这里,cal对象是取的JVM的默认时区,而JVM默认时区取的应是本机的时区UTC+8
```
public synchronized String toString(Calendar cal, Time x) {
        if(cal == null) {
            cal = this.defaultCal;
        }

        cal.setTime(x);
        this.sbuf.setLength(0);
        appendTime(this.sbuf, cal, cal.get(14) * 1000000);
        if(this.min74) {
            this.appendTimeZone(this.sbuf, cal);
        }

        showString("time", cal, x, this.sbuf.toString());
        return this.sbuf.toString();
    }
```
所以15:00:00+0在调用toString方法后返回值是23:00:00.000000 +08:00:00,但是数据库中event_time类型时Time without timezone,所为23后面的时区信息将会被忽略,最终数据库的时间变成了UTC+0 的23:00:00点

而将event_time从数据库中查询出来时,又将其当成了UTC+8的23:00:00,换算成UTC+0时间正好是UTC+0 15:00:00

所以要避免页面时间与数据库时间不统一的办法就是设置JVM的时区
```
-Duser.timezone=UTC
```


