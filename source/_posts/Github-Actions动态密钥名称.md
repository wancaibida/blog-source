title: Github Actions动态密钥名称
author: 大丈夫没问题
tags:
  - github
  - actions
categories:
  - linux
date: 2021-09-14 10:05:00
---

`test.yml`
```
name: LS

...

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:

    - name: Update and restart server
      uses: appleboy/ssh-action@v0.1.4
      with:
        host: ${{ secrets[format('{0}_HOST', github.workflow)] }}
        username: ${{ secrets[format('{0}_USER', github.workflow)] }}
```

比如workflow的名称为`LS`, 密钥名称就为:

```
      with:
        host: ${{ secrets['LS_HOST'] }}
        username: ${{ secrets['LS_USER'] }}
```