---
title: spring cloud集成swagger3
description: spring cloud集成swagger3
published: true
date: 2022-07-20T05:30:07.142Z
tags: 
editor: markdown
dateCreated: 2022-07-19T11:39:34.108Z
---

# spring cloud集成swagger3
## 服务端集成
1. 引入依赖
```xml 
<dependency>
   <groupId>io.springfox</groupId>
  <artifactId>springfox-boot-starter</artifactId>
  <version>3.0.0</version>
</dependency>
```
2. 代码配置
```java
import io.swagger.annotations.ApiOperation;
import org.springframework.context.EnvironmentAware;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.http.HttpMethod;
import springfox.documentation.builders.*;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.schema.ScalarType;
import springfox.documentation.service.*;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

import java.util.ArrayList;
import java.util.List;

/**
 * @Description: 配置Swagger
 * @Author QinQiang
 * @Date 2022/7/19
 **/
@Configuration
@EnableOpenApi
public class Swagger3Config implements EnvironmentAware {

    private String applicationName;

    private String applicationDescription;

    /**
     * 返回文档概要信息
     * @return
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.OAS_30)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build()
                // .globalRequestParameters(getGlobalRequestParameters())
                .globalResponses(HttpMethod.GET, getGlobalResponseMessage())
                .globalResponses(HttpMethod.POST, getGlobalResponseMessage());
    }

    /**
     * 生成接口信息，包括标题，联系人等
     * @return
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title(applicationName + "接口文档")
                .description(applicationDescription)
                .contact(new Contact("tel me", "http://www.baidu.com", "18335844494@163.com"))
                .version("1.0")
                .build();
    }


    /**
     * 封装全局通用参数
     * @return
     */
    private List<RequestParameter> getGlobalRequestParameters() {
        List<RequestParameter> parameters = new ArrayList<>();
        parameters.add(new RequestParameterBuilder()
                .name("uuid")
                .description("设备uuid")
                .required(true)
                .in(ParameterType.QUERY)
                .query(q -> q.model(m -> m.scalarModel((ScalarType.STRING))))
                .required(false)
                .build());
        return parameters;
    }

    /**
     * 封装通用相应信息
     *
     * @return
     */
    private List<Response> getGlobalResponseMessage() {
        List<Response> responseList = new ArrayList<>();
        responseList.add(new ResponseBuilder().code("404").description("未找到资源").build());
        return responseList;
    }

    @Override
    public void setEnvironment(Environment environment) {
        this.applicationDescription = environment.getProperty("spring.application.description");
        this.applicationName = environment.getProperty("spring.application.name");
    }
}
```
3. 访问<http://{ip}:{port}/swagger-ui/>

## gateWay集成
1. 同样引入依赖
2. 实现`SwaggerResourcesProvider`聚合资源
```java
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.gateway.config.GatewayProperties;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.support.NameUtils;
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;
import springfox.documentation.swagger.web.SwaggerResource;
import springfox.documentation.swagger.web.SwaggerResourcesProvider;

import java.util.ArrayList;
import java.util.List;

/**
 * @Description: swagger聚合系统接口
 * @Author QinQiang
 * @Date 2022/7/19
 **/
@Component
@Primary
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
public class SwaggerProvider implements SwaggerResourcesProvider {

    // SWAGGER3默认的URL后缀
    public static final String API_URI = "/v3/api-docs";
    // 网关路由
    private final RouteLocator routeLocator;

    private final GatewayProperties gatewayProperties;

    /**
     * 这个类是核心，这个类封装的是SwaggerResource，即在swagger-ui.html页面中顶部的选择框，选择服务的swagger页面内容。
     * RouteLocator：获取spring cloud gateway中注册的路由
     * RouteDefinitionLocator：获取spring cloud gateway路由的详细信息
     * RestTemplate：获取各个配置有swagger的服务的swagger-resources
     */
    @Override
    public List<SwaggerResource> get() {
        List<SwaggerResource> resources = new ArrayList<>();
        List<String> routes = new ArrayList<>();
        //取出gateway的route
        routeLocator.getRoutes().subscribe(route -> routes.add(route.getId()));
        //结合配置的route-路径(Path)，和route过滤，只获取有效的route节点
        gatewayProperties.getRoutes().stream().filter(routeDefinition -> routes.contains(routeDefinition.getId()))
                .forEach(routeDefinition -> routeDefinition.getPredicates().stream()
                        .filter(predicateDefinition -> ("Path").equalsIgnoreCase(predicateDefinition.getName()))
                        .forEach(predicateDefinition -> resources.add(swaggerResource(routeDefinition.getId(),
                                predicateDefinition.getArgs().get(NameUtils.GENERATED_NAME_PREFIX + "0")
                                        .replace("/**", API_URI)))));
        return resources;
    }

    private SwaggerResource swaggerResource(String name, String location) {
        SwaggerResource swaggerResource = new SwaggerResource();
        swaggerResource.setName(name);
        swaggerResource.setLocation(location);
        swaggerResource.setSwaggerVersion("3.0.0");
        return swaggerResource;
    }
}
```
3. 定义资源处理器，ui页面将会自动访问
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;
import springfox.documentation.swagger.web.*;

import java.util.Optional;

/**
 * @Description:
 * @Author QinQiang
 * @Date 2022/7/19
 **/
@RestController
@RequestMapping("/swagger-resources")
public class SwaggerHandler {
    @Autowired(required = false)
    private SecurityConfiguration securityConfiguration;
    @Autowired(required = false)
    private UiConfiguration uiConfiguration;
    private final SwaggerResourcesProvider swaggerResources;

    @Autowired
    public SwaggerHandler(SwaggerResourcesProvider swaggerResources) {
        this.swaggerResources = swaggerResources;
    }


    @GetMapping("/configuration/security")
    public Mono<ResponseEntity<SecurityConfiguration>> securityConfiguration() {
        return Mono.just(new ResponseEntity<>(
                Optional.ofNullable(securityConfiguration).orElse(SecurityConfigurationBuilder.builder().build()), HttpStatus.OK));
    }

    @GetMapping("/configuration/ui")
    public Mono<ResponseEntity<UiConfiguration>> uiConfiguration() {
        return Mono.just(new ResponseEntity<>(
                Optional.ofNullable(uiConfiguration).orElse(UiConfigurationBuilder.builder().build()), HttpStatus.OK));
    }

    @GetMapping("")
    public Mono<ResponseEntity> swaggerResources() {
        return Mono.just((new ResponseEntity<>(swaggerResources.get(), HttpStatus.OK)));
    }
}
```
4. 修改gateWay配置
```yml
spring:
    application:
        name: gateway-server
    main:
        allow-bean-definition-overriding: true
    cloud:
        gateway:
            discovery:
                locator:
                    enabled: true
            routes:
                -   id: system_server
                    uri: lb://system-server
                    predicates:
                        - Path=/system/**
                    filters:
                        - RewritePath=/system/v3/api-docs, /v3/api-docs
```
5. 访问<http://{gateWay-ip}:{port}/swagger-ui/>

## 使用SpringDoc UI界面
1. gateWay 引入依赖
```xml
<dependency>
  <groupId>com.github.xiaoymin</groupId>
  <artifactId>knife4j-spring-boot-starter</artifactId>
  <version>3.0.3</version>
</dependency>
```
2. 放行相关路径
```yml
oauth2:
    cloud:
        sys:
            parameter:
                ignoreUrls:
                    - /oauth/**
                    - /system/login
                    - /system/client_login
                    - /system/*.html
                    - /system/*.js
                    - /system/*.css
                    - /system/favicon.ico
                    - /system/v3/api-docs
                    - /account/v3/api-docs
                    - /order/v3/api-docs
                    - /product/v3/api-docs
                    - /search/v3/api-docs
                    - /swagger-ui/**
                    - /swagger-resources/**
                    - /doc.html
                    - /webjars/**
                    - /favicon.ico
```
3. 访问<http://{gateWay-ip}:{port}/doc.html>