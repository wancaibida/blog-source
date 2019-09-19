title: Google Cloud Function Http 认证配置
author: 大丈夫没问题
tags:
  - google
  - cloud
  - function
  - linux
  - http
  - authentication
categories:
  - linux
date: 2019-09-19 22:00:00
---
最近在做的项目用到了google cloud，其中有个功能模块的google cloud function需要调用另一个通过http请求触发的function，在研究配置http function认证信息期间花了不少时间，在这记录一下。

默认情况下创建http请求触发的function会勾选`Allow unauthenticated invocations`选项，这样无需通过认证就可以调用function，但这样做显然是不安全的，一旦接口地址泄漏就可能会被恶意调用。

查看了google cloud文档，google建议用[Cloud Endpoint](https://cloud.google.com/endpoints/)来做认证，但是在cloud console始终无法创建endpoint，遂放弃。

一开始想了个临时的解决方案：在勾选`Allow unauthenticated invocations`的情况下，在function代码里加上token验证，如果token不匹配就返回并提示`forbidden`，虽然可以减少function在被恶意调用的情况下的执行时间，但还是会产生一定的费用。

今天抽时间重新研究了下这个问题，终于找到了[解决方案](https://cloud.google.com/functions/docs/securing/authenticating#function-to-function)：

假设调用者function为 `xxx-master`，被调用的function为`xxx-slave`,

* 通过下面命令给 `xxx-slave` function加上`roles/cloudfunctions.invoker` 角色，允许 `roles/cloudfunctions.invoker`这个角色来调用`xxx-slave` function

```
gcloud beta functions add-iam-policy-binding RECEIVING_FUNCTION \
  --member='serviceAccount:CALLING_FUNCTION_IDENTITY' \
  --role='roles/cloudfunctions.invoker'
```

`RECEIVING_FUNCTION` -> 被调用的function名字
`CALLING_FUNCTION_IDENTITY` -> 一般为 `PROJECT_ID@appspot.gserviceaccount.com`

比如：

```
gcloud beta functions add-iam-policy-binding xxx-slave \
  --member='serviceAccount:YOUR_PROJECT_ID@appspot.gserviceaccount.com' \
  --role='roles/cloudfunctions.invoker'
```

* 在 `xxx-master` function中加上获取token代码，请求时将token放在`Authorization` header里，以python为例：

```
# Requests is already installed, no need to add it to requirements.txt
import requests

def calling_function(request):
  # Make sure to replace variables with appropriate values
  receiving_function_url = 'https://us-central1-graphical-bus-248617.cloudfunctions.net/xxx-slave
'

  # Set up metadata server request
  # See https://cloud.google.com/compute/docs/instances/verifying-instance-identity#request_signature
  metadata_server_token_url = 'http://metadata/computeMetadata/v1/instance/service-accounts/default/identity?audience='

  token_request_url = metadata_server_token_url + receiving_function_url
  token_request_headers = {'Metadata-Flavor': 'Google'}

  # Fetch the token
  token_response = requests.get(token_request_url, headers=token_request_headers)
  jwt = token_response.content.decode("utf-8")

  # Provide the token in the request to the receiving function
  receiving_function_headers = {'Authorization': f'bearer {jwt}'}
  function_response = requests.get(receiving_function_url, headers=receiving_function_headers)

  return function_response.content
```

完成上面两部后，`xxx-slave` function就可以在被认证的情况下调用了。

这里还需要注意的是要将`Cloud Functions Invoker` 中的 `all user`移除，不然`xxx-slave`方法还是公开的，操作步骤：

* Google Cloud Console -> Cloud Function
* 勾选`xxx-slave` function
* 点击左侧的 `PERMISSIONS` tab
* 点开 `Cloud Functions Invoker`
* 将 `all user`移除


参考：
* [https://cloud.google.com/functions/docs/securing/authenticating#function-to-function](https://cloud.google.com/functions/docs/securing/authenticating#function-to-function)

