title: Groovy正则表达式用法
author: Gang Chen
tags:
  - groovy
categories:
  - java
date: 2018-03-28 21:17:00
---
Matcher example：

```
        //Matcher example
        String regexStr = /gr.*/
        String str = 'groovy'

        Matcher matcher0 = (str =~ regexStr)
        boolean result0 = (str ==~ regexStr)
        assert matcher0.matches() == result0

        Matcher matcher1 = (str =~ /$regexStr/)
        boolean result1 = (str ==~ /$regexStr/)
        assert matcher1.matches() == result1
```
Find example:

```
        def cool = /gr\w{4}/  // Start with gr followed by 4 characters.
        Matcher matcher2 = ('groovy, java and grails rock!' =~ /$cool/)
        assert 2 == matcher2.count
        assert 2 == matcher2.size()  // Groovy adds size() method.
        assert 'groovy' == matcher2[0]  // Array-like access to match results.
        assert 'grails' == matcher2.getAt(1)
```