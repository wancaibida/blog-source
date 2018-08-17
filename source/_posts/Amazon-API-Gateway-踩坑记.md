title: Amazon API Gateway 踩坑记
author: 大丈夫没问题
date: 2018-07-19 20:23:52
tags:
---
最近项目中用到了Amazon API Gateway，当中遇到了不少的坑。本文主要罗列了在使用amazon api gateway中遇的问题，希望能帮助到遇到相同问题的开发者。

## 不支持parmeters array
```
GET /user/1/addresses?addressId=1&addressId=2
```
服务端接收到的只有一个值，解决办法[passing-array-query-parameters-with-api-gateway-to-lambda](https://stackoverflow.com/questions/43401777/passing-array-query-parameters-with-api-gateway-to-lambda)

## POST/PUT 返回 406
当使用json body发送put/post请求时，api gateway有一定机率返回406，查了半天也找出什么原国，后来在stackoverflow上找到了答案，需要在Integration Request中添加Accept头，并且设置其值为空字符串。具体什么原因估计只有AWS api gateway的开发者才知道了。
[aws-api-gateway-returns-http-406](https://stackoverflow.com/questions/42106605/aws-api-gateway-returns-http-406)

## URL Query String Parameter、header需要显示设定
如果不在Method Request里显示声名URL Query String Parameter的话，请求参数是不会转发到后台的，request header同理。

## Grails参数绑定不支持`application/x-amz-json-1.0`/`application/x-amz-json-1.0` content type.
请求转发到后台时，API gateway会将`application/json`转换成`application/x-amz-json-1.0`/`application/x-amz-json-1.0`，需要修改代码，添加对这两个header的支持，不然会无法绑定参数。


