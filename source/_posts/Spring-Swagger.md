---
title: SpringMVC整合Springfox-Swagger
tags:
  - JAVA
  - Spring
originContent: ''
categories:
  - 技术
  - JAVA
toc: true
date: 2019-07-02 23:26:12
---

```
关于Swagger的简介就不占篇幅了...
本文使用的Springfox-Swagger版本为2.8.0
```
---
要整合Springfox-Swagger,只需要在Maven导入两个包即可,没有Maven下载导入也行...
```
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.8.0</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.8.0</version>
        </dependency>
```
选择版本时,最好保持两个包的版本一致,以免出现不可预知的问题~
以上两个是使用Swagger的基本包,如果需要接口自动完成对象和JSON串的转换的话,需要再导入Jackson支持
```
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.6.5</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.6.5</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.6.5</version>
        </dependency>
```
导入包以后,创建一个简单的Swagger配置类
```
package net.sealin.config;

import io.swagger.annotations.ApiOperation;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

// 仅在没有Spring-boot的项目中需要开启此配置
@EnableWebMvc
// 启用Swagger2
@EnableSwagger2
// 让Spring来加载该类配置
@Configuration
/**
 * 也可在Spring配置文件中配置 
 * <context:component-scan base-package="net.sealin.controller"/>
 */
@ComponentScan(basePackages = "net.sealin.controller")
/**
 * @author Sealin
 * Created by Sealin on 2018-03-28.
 */
public class SwaggerConfig {
    @Bean
    public Docket buildDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(buildApiInf())
                .select()
                //controller匹配规则
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo buildApiInf() {
        return new ApiInfoBuilder()
                .title("开放接口API")
                .termsOfServiceUrl("http://localhost:8099/v2/api-docs")
                .description("项目名称等描述性词语")
                .contact(new Contact("Sealin", "http://www.sealin.net/", "admin@sealin.net"))
                .version("1.0")
                .build();

    }
}

```
要让此配置类生效,需要Spring上下文配置中存在如下选项:
```
    <!-- 让Swagger可以访问Controller -->
    <mvc:annotation-driven />
    <!-- 开启注解管理 -->
    <context:annotation-config />
    <!-- 将我们建立的配置类加入Spring容器 -->
    <bean class="net.sealin.config.SwaggerConfig" />
```
---
```
    <!-- 官方说明 -->
    <!-- Required so springfox can access spring's RequestMappingHandlerMapping  -->
    <mvc:annotation-driven/>

    <!-- Required to enable Spring post processing on @Configuration classes. -->
    <context:annotation-config/>

    <bean class="com.yourapp.configuration.MySwaggerConfig"/>
```
此外,因为我们用Spring实现的Servlet取代了默认的,在处理Swagger-UI的静态资源时,Spring-Servlet并不会帮我们映射这些资源文件,会导致不能访问swagger-ui.html的情况,两种方式可以解决这个问题,任选一种即可:
一.将没有@Controller解析的请求交给默认Servlet处理
```
    <mvc:default-servlet-handler />
```
二.给Spring-servlet指定我们需要映射的资源文件路径
```
    <mvc:resources mapping="swagger-ui.html" location="classpath:/META-INF/resources/"/>
    <mvc:resources mapping="/webjars/**" location="classpath:/META-INF/resources/webjars/"/>
```
至此,Spring和Swagger的整合过程就告一段落了,运行试试:
```
API文档视图及操作界面:
http://127.0.0.1:8080/swagger/swagger-ui.html
所有API的汇总信息(JSON)
http://127.0.0.1:8080/swagger/v2/api-docs
```
遇到其他问题可以留言噢,祝君好运~