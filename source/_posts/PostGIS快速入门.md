title: PostGIS快速入门
author: 大丈夫没问题
tags:
  - postgis
  - postgresql
categories:
  - database
date: 2020-07-23 20:30:00
---
## 安装

```
CREATE EXTENSION postgis
```

## 添加列
```
ALTER TABLE your_table ADD COLUMN geom geometry(Point, 4326);
UPDATE your_table SET geom = ST_SetSRID(ST_MakePoint(longitude, latitude), 4326);
```

## 半径查找

```
SELECT lat,
       lon,
       location,
       cnt
FROM ggg
WHERE 1 = 1
  AND cnt > 1
  AND ST_DWithin(
        ggg.geom,
        ST_MakePoint(42.9764, 47.5024)::GEOGRAPHY,
        200000
    );

```

## 参考:
https://live.osgeo.org/en/quickstart/postgis_quickstart.html
https://gis.stackexchange.com/questions/145007/creating-geometry-from-lat-lon-in-table-using-postgis