---
title: SpringBoot+SpringCloud+Nacos
tags:
  - JAVA
  - SpringBoot
  - Nacos
  - SpringCloud
categories:
  - 技术
  - JAVA
toc: true
date: 2019-08-15 21:37:14
---

Nacos是阿里巴巴开源的一个注册中心和配置中心组件, 并且配置中心和注册中心可以单独导入项目, 只用其中一个.结合Spring Cloud可以实现与项目的高度融合, 配置支持多种格式, 可以轻松将项目配置从本地转到服务器, 集群搭建也非常简单.
在线配置结合nacos的命名空间功能, 轻松实现多环境切换, 完美~
本文使用的SpringBoot版本为: 2.1.7.RELEASE
# Nacos 安装与启动
参考: [官方文档](https://nacos.io/zh-cn/docs/quick-start.html)
启动后默认端口为8848(钛合金手机?)
管理页面:
http://localhost:8848/nacos
# 添加依赖
无需单独引入Spring Cloud依赖
```xml
<!--        配置中心-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
            <version>0.9.0.RELEASE</version>
        </dependency>

<!--        注册中心-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.9.0.RELEASE</version>
        </dependency>
```
# 项目配置
	bootstrap.yml
	注意文件名是bootstrap*, 试过使用application.yml启动会报错.

```yml
spring:
  application:
    # 程序的name与配置中心生成DataId有关, 避免出现问题建议配置此项
    # 配置中心的DataId可以理解为配置文件名称
    # 默认生成规则: ${spring.application.name}-${spring.profile.active}.${file-extension}
    name: nacos-demo
  cloud:
    nacos:
      # 注册中心
      discovery:
        server-addr: 192.168.10.129:8848
      # 配置中心
      config:
        server-addr: 192.168.10.129:8848
        # 配置文件后缀. 默认为.properties
        file-extension: yml
        # 需要加载的其他配置dataId, 多个以英文逗号隔开
        shared-dataids: common.yml, redis.yml, mysql.yml
        # 线上配置修改后需要自动刷新的配置
        refreshable-dataids: common.yml, redis.yml, mysql.yml
```

# 使用在线配置
到nacos控制台添加一个配置文件, 上文已配置了加载common.yml, 以此名称为例:
![image.png](/images/2019/08/15/9a2f2c30-bf5c-11e9-ae6a-b52e335d1f68.png)
保存后添加一个Controller, 用于读取在线配置
```JAVA
package net.sealin.nacos.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author Sealin
 * @date 2019-08-14
 */
@RestController
// 使用Spring Cloud提供的注解开启配置刷新支持
@RefreshScope
public class TestController {
    @Value(value = "${age}")
    private String value;

    @RequestMapping("/")
    public String getValue() {
        return value;
    }
}
```
启动项目后访问http://localhost:8080, 发现已经读取到了在线配置

# 多环境配置
结合Spring读取配置文件的规则, 结合nacos配置中心的命名空间, 可以为不同环境的项目加载对应环境的配置文件.
到管理页面添加命名空间:
![image.png](/images/2019/08/15/efb37c30-bf5a-11e9-ae6a-b52e335d1f68.png)
框选的部分为产生的命名空间ID, 项目配置文件中用这个ID去指定命名空间, 写命名空间名称是行不通滴~

	bootstrap-test.yml
```yml
spring:
  cloud:
    nacos:
      config:
	# 命名空间的值要写nacos生成的ID
        namespace: 141c776d-9524-4d84-a70d-9df2ea547708
```
此时将spring的启动参数中指定启动环境为test, springboot则会在读取bootstrap.yml后, 读取bootstrap-test.yml文件, 此时nacos配置中心的命名空间就指向了test
![image.png](/images/2019/08/15/50919520-bf5e-11e9-ae6a-b52e335d1f68.png)
重新运行项目, 错误信息如下:

	Caused by: java.lang.IllegalArgumentException: Could not resolve placeholder 'age' in value "${age}"
因为test命名空间下我们没有配置age这个key, 要屏蔽错误也比较简单, 可以在@Value注解中添加一个默认值, 在没有读取到在线配置时, 将显示此属性, 注册中心存在指定配置key时将获取线上配置:

	@Value(value = "${age:10}")

# 注册中心
## 服务注册
由于上文我们将环境切换到了测试环境, 项目启动时会读取nacos/appname-test.yml配置, 根据这个特性, 到nacos控制台在test命名空间下添加一个配置文件, 用于指定服务注册的命名空间
![image.png](/images/2019/08/16/c1fcbaf0-bfcc-11e9-8866-5302bf177721.png)

	nacos-demo-test.yml
内容如下:
```yml
spring:
  cloud:
    nacos:
      discovery:
        namespace: 141c776d-9524-4d84-a70d-9df2ea547708
```

在应用启动类上加入

	@EnableDiscoveryClient

此时去nacos控制台上查看, 会发现服务已经被注册到test命名空间了.
![image.png](/images/2019/08/15/c18a5ea0-bf5f-11e9-ae6a-b52e335d1f68.png)
## 服务发现
### LoadBalance
注册一个RestTemplate实例
```JAVA
package net.sealin.nacos;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

/**
 * @author Sealin
 */
@SpringBootApplication
@EnableDiscoveryClient
public class NacosApplication {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(NacosApplication.class, args);
    }

}

```
添加消费方法:
```JAVA
    @Autowired
    private RestTemplate restTemplate;
    @RequestMapping("/lb")
    public String loadBalance() {
        return restTemplate.getForObject("http://nacos-demo/", String.class);
    }
```
访问http://localhost:8080/lb, 结果与http://localhost:8080/一致
### OpenFeign
添加依赖
```XML
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.1.2.RELEASE</version>
        </dependency>
```
在启动类上开启Feign功能
```JAVA
package net.sealin.nacos;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

/**
 * @author Sealin
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class NacosApplication {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(NacosApplication.class, args);
    }

}

```

添加映射接口
```Java
package net.sealin.nacos.controller.mapping;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 *
 * @author Sealin
 * @date 2019-08-14
 */
@FeignClient("nacos-demo")
public interface INacosDemoMapping {
    @RequestMapping("/")
    String get();
}

```

添加消费方法:
```Java
    @Autowired
    private INacosDemoMapping nacosDemoMapping;
    @RequestMapping("/feign")
    public String feignRequest() {
        return nacosDemoMapping.get();
    }
```
访问http://localhost:8080/feign, 结果与/lb一致