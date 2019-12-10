title: Bitbucket pipeline传递文件内容变量
author: 大丈夫没问题
tags:
  - bitbucket
  - pipeline
categories:
  - linux
date: 2019-12-10 21:15:00
---
Bitbucket pipeline可以通过`Repository variables`来传递变量，但是如果变量包含一些特殊字符比如换行符，bitbucket就不能很好的处理，对于这种情况我们可以将变量用base64编码一下，在pipeline中再解码就可以解决这问题了。

```
cat file.txt | base64

// in pipeline file
echo ${YOUR_ENV} | base64 -d > file.txt
```