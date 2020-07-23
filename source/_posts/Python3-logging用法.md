title: Python3 logging用法
author: 大丈夫没问题
tags:
  - python
categories:
  - linux
date: 2020-07-23 20:31:00
---
```
import logging


log = logging.getLogger()
log.setLevel(logging.DEBUG)

handler = logging.StreamHandler(sys.stdout)
handler.setLevel(logging.INFO)
formatter = logging.Formatter('[%(threadName)s] %(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
log.addHandler(handler)

log.info("Hello %s", "world)
```