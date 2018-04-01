title: Spring RestTemplate 打印Request及Response内容
author: Gang Chen
tags:
  - spring
  - rest
categories:
  - java
date: 2018-04-01 21:09:00
---
```groovy
import org.apache.commons.io.IOUtils
import org.slf4j.Logger
import org.slf4j.LoggerFactory
import org.springframework.http.HttpRequest
import org.springframework.http.client.ClientHttpRequestExecution
import org.springframework.http.client.ClientHttpRequestInterceptor
import org.springframework.http.client.ClientHttpResponse


class LoggingRequestInterceptor implements ClientHttpRequestInterceptor {

    static final String DEFAULT_ENCODING = 'UTF-8'
    static final Logger LOGGER = LoggerFactory.getLogger(LoggingRequestInterceptor)

    @Override
    ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        traceRequest(request, body)
        ClientHttpResponse response = execution.execute(request, body)
        traceResponse(response)
        return response
    }

    private void traceRequest(HttpRequest request, byte[] body) throws IOException {
        LOGGER.info('===========================request begin================================================')
        LOGGER.info('URI         : {}', request.URI)
        LOGGER.info('Method      : {}', request.method)
        LOGGER.info('Headers     : {}', request.headers)
        LOGGER.info('Request body: {}', new String(body, 'UTF-8'))
        LOGGER.info('==========================request end================================================')
    }

    private void traceResponse(ClientHttpResponse response) throws IOException {
        LOGGER.info('============================response begin==========================================')
        LOGGER.info('Status code  : {}', response.statusCode)
        LOGGER.info('Status text  : {}', response.statusText)
        LOGGER.info('Headers      : {}', response.headers)
        LOGGER.info('Response body: {}', IOUtils.toString(response.body, DEFAULT_ENCODING))
        LOGGER.info('=======================response end=================================================')
    }

}

```
* 配置RestTemplate

```groovy
        RestTemplate restTemplate = new RestTemplate()
        restTemplate.setRequestFactory(new BufferingClientHttpRequestFactory(new SimpleClientHttpRequestFactory()))
        restTemplate.setInterceptors([new LoggingRequestInterceptor()])
```
由于会多次读取request及response body，所以我们这里会使用`BufferingClientHttpRequestFactory`类来保证能够读取多次。