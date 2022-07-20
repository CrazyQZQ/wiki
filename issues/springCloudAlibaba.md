---
title: spring cloud alibaba issues
description: 
published: true
date: 2022-07-20T02:54:13.179Z
tags: 
editor: markdown
dateCreated: 2022-05-02T14:56:06.302Z
---

## 通过gateway调用服务异常，直接调用成功

原因分析：GlobalAuthenticationFilter.java 中打印日志时消费了requestBody，导致到达服务时requestBody为空

打印gateway请求日志方法：

```java
private ServerHttpResponseDecorator printLog(ServerWebExchange exchange){
        long start = System.currentTimeMillis();

        RequestLogInfo logInfo = new RequestLogInfo();
        // 获取请求信息
        ServerHttpRequest request = exchange.getRequest();
        InetSocketAddress address = request.getRemoteAddress();
        String method = request.getMethodValue();
        URI uri = request.getURI();

        logInfo.setRequestId(request.getId());
        logInfo.setRequestUrl(uri.getPath());
        logInfo.setRequestTime(DateUtils.dateTimeNow(DateUtils.YYYY_MM_DD_HH_MM_SS));
        // 获取请求body
        if("POST".equals(method)){
            //获取请求体，消费了请求体
            Flux<DataBuffer> body = request.getBody();

            AtomicReference<String> bodyRef = new AtomicReference<>();
            body.subscribe(buffer -> {
                CharBuffer charBuffer = StandardCharsets.UTF_8.decode(buffer.asByteBuffer());
                DataBufferUtils.release(buffer);
                bodyRef.set(charBuffer.toString());
            });
            //获取request body
            logInfo.setRequestBody(bodyRef.get());
        }
        // 获取请求query
        logInfo.setRequestParam(JSON.toJSONString(request.getQueryParams()));
        logInfo.setRemoteAddr(address.getHostName()+address.getPort());
        logInfo.setRequestMethod(method);

        ServerHttpResponse response = exchange.getResponse();
        DataBufferFactory bufferFactory = response.bufferFactory();
        ServerHttpResponseDecorator decoratedResponse = new ServerHttpResponseDecorator(response) {
            @Override
            public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
                if (body instanceof Flux) {
                    Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;
                    return super.writeWith(fluxBody.map(dataBuffer -> {
                        byte[] content = new byte[dataBuffer.readableByteCount()];
                        dataBuffer.read(content);
                        String responseResult = new String(content, Charset.forName("UTF-8"));
                        logInfo.setResponseCode(this.getStatusCode().toString());
                        logInfo.setResponseBody(responseResult);

                        // 计算请求时间
                        long end = System.currentTimeMillis();
                        logInfo.setResponseTime(end - start);
                        log.info(JSON.toJSONString(logInfo));
                        return bufferFactory.wrap(content);
                    }));
                }
                return super.writeWith(body);
            }
        };
        return decoratedResponse;
    }
```