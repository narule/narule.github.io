---
title: SpringCloud-常用组件介绍
author: Narule
date: 2021-01-31 22:12:00 +0800
categories: [Technology^技术, Java, Spring]
tags: [writing, Java, Spring, SpringCloud]
---



分布式系统开发用于分布式环境（多个服务器不在同一个机房，同一个业务服务在多台服务器运行）

Spring Cloud 是基于Springboot的分布式云服务架构，SpringCloud的设计就是为了分布式的云环境设计

下面说一些SpringCloud项目在开发中常用的几个组件

说组件之前，将一些分布式相关的概念

> CAP定理 指分区容错性 服务可用性 数据一致性，分布式环境：
>
> ​	容错性，允许部分机器故障，但是系统人仍能正常运作
>
> ​	服务可用性   任何时候调用服务有响应
>
> ​	数据一致性   任何时候获取数据都一样(访问不同机器节点相同的业务数据)



CAP定理因为分布式而存在，离开具体业务相对来说有些抽象，CAP只是说设计系统时的参考思想；但很明显的意思就是为了提高系统稳定性，让原来的一台服务器变成多台，提供相同服务，部分服务器坏了没关系，其他正常服务器能提供服务，从而让使用系统的一方不受影响，觉得这个系统是稳定的，但随之而来的是不同服务器之间数据如何同步，协作，就成了分布式系统要面对的问题。

简单的商城分布式系统，一个服务会运行在多台机器上，买商品下单这一个件事，需要多台服务器中的一个去完成，如果有两台服务器宕机，就去访问没有宕机的服务器（容错性）。下单是调用服务，调用服务这个过程是需要畅通的，不能等很久，下单服务在最多1秒左右就完成。（服务可用性）。下完单，被调用的服务器需要修改商品库存数据，将库存减一(x-1)。在这之后其他服务器访问显示商品库存也是x-1，而不是x（数据一致性）。



## SpringCloud-常用组件

SpringCloud分布式组件提高了系统的开发效率，稳定性，可维护性

SpringCloud-Config 服务配置，提供统一配置功能。多个服务，或者服务器启动后，配置文件都在Config中，方便管理（分布式统一配置相关）

SpringCloud-Eureka 服务注册，提供注册服务。服务启动后，可以提供自己的服务地址注册到Eureka，暴露出来提供给其他人调用（服务可用性相关）

SpringCloud-OpenFeign 服务代理，提供代理调用服务。A服务要调用B服务，可以通过feign，feign使服务调用更方便（远程服务调用相关，有断路器，防止服务雪崩）

SpringCloud-Gateway 网关路由服务，提供代理访问转发。 访问narule.net/api 实际访问narule.github.io/api 可以通过Gateway来实现。（访问安全相关）



还有很多其他组件，比如redis相关组件用于数据一致性访问，kafka用于高吞吐消息队列等，本文主要讲上面提到的四个，通过代码简单说明如何使用。

### 参考代码

**example 代码地址：**[springcloud-example](https://github.com/narule/springcloud-example)

完整代码请参考[springcloud-example](https://github.com/narule/springcloud-example)，以及maven的完整依赖；如果要启动里面的服务，请务必先启动config-server，因为里面的项目都是通过config统一配置，包括服务用哪个端口访问等参数，；

## Config

Config作为配置中心，能够很方便配置每个分布式系统参数，配置文件可以通过git等仓库专门管理

参考代码：[config](https://github.com/narule/springcloud-example/tree/main/springcloud-config)

测试访问：http://localhost:8888/eureka-server.yml http://localhost:8888/eureka-client.yml

### example



关键配置是application.yml中配置文件的位置，一般由url指定



#### dependencies 依赖 

serverConfig-pom.xml

```xml
<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>
</dependencies>
```

#### application.yml

```yml
server:
  port: 8888 #服务端口
spring:
  application:
    name: server-config  #服务名
  cloud:
    config:
      server:
        git:
          uri: https://github.com/narule/spring-cloud-config   #配置文件地址
          #username: narule   
          #password: password
    refresh:
      enabled: true
```



#### ServerConfigApplication

启动程序，除了SpringBootApplication，还需要EnableConfigServer注解:

```java
package net.narule.spring.cloud.config.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}

}
```

### 配置读取规则

下文中所有应用程序的配置（包括端口）都是读取的 https://github.com/narule/spring-cloud-config 

如果启动程序 spring.application.name=server1-client，并且配置了远程配置，此程序会尝试从远程读取server1-client.yml 文件的配置参数，这是springcloud-config配置规则

客户端应用读取配置只需要依赖`spring-cloud-starter-config`  ，不需要在启动类中写什么注解



#### client-dependencies 

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```



#### client-application.yml

spring-cloud-starter-config 3.0 有新增配置

```yml
# client一般指定配置路径
spring:
  application:
    name: config-client-one #一定要在本地指定 spring.application.name 才能读取远程配置

  cloud:
    refresh:
      enabled: true
    config:
      uri:
      - http://localhost:8888 # config-server的访问地址


# 3.0之后新的方式 指定url
spring:
  application:
    name: config-client-one
  config:
    import:
     - optional:configserver:http://narule.net:8888 #config-server的访问地址
```

#### ConfigClientOneApplication

```java
package net.narule.spring.cloud.config.client;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConfigClientOneApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigClientOneApplication.class, args);
	}

}
```



## Eureka

Eureka提供服务注册，分为注册中心和客户端，注册中心使用@EnableEurekaServer

客户端有producer服务提供者，consumer 服务消费者，在使用时，都使用@EnableEurekaCilent注解，启动服务的时候，将自己的服务信息，ip和端口号等，注册到Eureka服务中心，让其他eurekaclient能通过Eureka服务注册中心获取服务的ip和端口号。

参考代码：[eureka](https://github.com/narule/springcloud-example/tree/main/springcloud-eureka)

测试访问：https://localhost:1999/eureka-server



### example

​	服务端，主要参数是服务的节点，注册节点地址，注册帐号密码（可选）

#### server-dependencies 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
<!--Spring Boot Actuator，感应服务端变化-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!-- security 此模块权限校验 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-security</artifactId>
</dependency>
```



#### server-application.yml

```yml
server:
  port: 1999
spring:
  application:
    name: eureka-server #注册中心访问path
eureka:
  dashboard:
    path: eureka-server
  instance:
    hostname: localhost
  client:
    registerWithEureka: false #将自己注册微eurekaclient false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #注册节点地址
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 10000 #服务刷新时间
```



#### EurekaServerApplication

配置文件配置好后，使用@EnableEurekaServer注解即可

```java
package com.wunanyu.cloud.eureka;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.WebApplicationType;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}
}
```



#### 客户端

#### client-dependencies 

客户端依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### client-application.yml

配置文件主要是指定注册中心地址，通过这个配置客户端可以把自己的服务注册到注册中心，或者从注册中心获取其他服务的信息，

```yml
spring:
  application:
    name: eureka-client
## 下面的配置是通过config-server读取，如果没有配置中心，则需要在本地配置文件中写好    
server:
  port: 18080
eureka:
#客户端
  client:
#注册中心地址
    service-url:
      defaultZone: http://localhost:1999/eureka/  #这里是
      
  instance:
    instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}# 实例id命名规则，这个用于区分同一服务在不同的机器，可应用于分布式锁
    prefer-ip-address: true #以IP地址注册到服务中心，相互注册使用IP地址
    hostname: localhost
```



#### EurekaClientApplication

客户端的使用是注解@EnableEurekaClient，有这个注解，启动时会把自己的信息注册到服务中心，当然要配置注册中心节点信息，让客户端知道注册地址在哪。

```java
package net.narule.spring.cloud.eureka.client;


import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@EnableEurekaClient
@RestController
public class EurekaClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaClientApplication.class, args);
	}

	@RequestMapping("/request-eureka-client/{id}")
	public ResponseResult requestEurekaClient(@PathVariable String id) {
		return ResponseResult.ok(id + "from-eureka-client");
	}
}

```



## OpenFeign

Feign和eureka一样是Nexflix的组件，可以用作接口调用Eureka服务，完成服务于服务之间的调用

参考代码：[openfeign](https://github.com/narule/springcloud-example/tree/main/springcloud-openfeign)

测试访问：http://localhost:1666/request-feign-client

### example

#### dependencies 依赖

因为feign通过eureka调用服务，并且在分布式环境考虑到负载均衡，所以依赖除了feign，还有ribbon和eureka

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>
        spring-cloud-starter-netflix-eureka-client
    </artifactId>
</dependency>
```



#### FeignClientApplication

要使用feign功能，启动类需要使用注解@EnableFeignClients @EnableDiscoveryClient

```java
package net.narule.spring.cloud.feign.client;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@EnableFeignClients
@RestController
public class FeignClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(FeignClientApplication.class, args);
	}
	@Autowired
	private FeignInterFace feignInterFace;
	
	@RequestMapping("/request-feign-client")
	public ResponseResult requestFeignClient() {
		String id = String.valueOf(System.currentTimeMillis());
		ResponseResult reuqestEurekaClient = feignInterFace.reuqestEurekaClient(id);
		return ResponseResult.ok(reuqestEurekaClient);
	}
	
	
	@FeignClient("eureka-client")
	interface FeignInterFace {
		
		@RequestMapping(value = "/request-eureka-client/{id}")
		public ResponseResult reuqestEurekaClient(@PathVariable("id") String id);
		
	}

}
```



#### FeignInterface

Feign使用时，只需要写接口，并在注解上配好url即可，不需要写代码实现方法，

@FeignClient("eureka-client")表示有服务名叫eureka-client，方法上的value = "/request-eureka-client/{id}" 说明eureka-client服务有/request-eureka-client/{id}的请求路径，这是要对应的，并且eureka-client服务注册到eureka注册中心，不然服务调用失败。

```java
@FeignClient("eureka-client")
interface FeignInterFace {

    @RequestMapping(value = "/request-eureka-client/{id}")
    public ResponseResult reuqestEurekaClient(@PathVariable("id") String id);

}
```

#### feign-client.yml

配置文件需要指定eureka的注册节点，因为feign最终通过eureka获取服务信息来完成接口访问

```yml
server:
  port: 1666
  
eureka:
#客户端
  client:
#注册中心地址
    service-url:
      defaultZone: http://localhost:1999/eureka/
```



## Gateway

gateway是网关组件，此组件可以用来转发请求，设置路由，比较灵活。

如果有服务访问路径是http://localhost:1666/request-feign-client 我们可以因为一些个性化需求改变路由

通过gateway可以让访问http://localhost:12000/api/feign/request-feign-client 等同于上面的访问路径



参考代码：[gateway](https://github.com/narule/springcloud-example/tree/main/springcloud-gateway)

测试访问：http://localhost:12000/api/feign/request-feign-client

### example 

#### gateway-dependencies 

gateway作为网关服务，可以搭配eureka使用

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>
        spring-cloud-starter-netflix-eureka-client
    </artifactId>
</dependency>
```

#### GatewayServerApplication

```java
@SpringBootApplication
public class GatewayServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(GatewayServerApplication.class, args);
	}
}
```



#### gateway-server.yml

```yml
server:
  port: 12000

spring:
  cloud:
    gateway:
      routes:
        # 路由ID（一个路由配置一个ID）
        - id: eureka-c
 
          # 通过注册中心来查找服务（lb代表从注册中心获取服务，并且负载均衡）
          uri: lb://eureka-client/
 
          # 匹配到的以/api/eureka/ 通过eureka访问 eureka-client/**
          predicates:
            - Path=/api/eureka/**
          # 去掉匹配到的路径的前2级 这里也就是 /api/eureka
          filters:
            - StripPrefix=2
            
        - id: feign-c
          
          uri: lb://feign-client/
 		  # 匹配到的以/api/feign/开头的路径都转发到feign-client的服务，相当于访问 lb://feign-client/**
          predicates:
            - Path=/api/feign/**
          # 去掉匹配到的路径的前2级 这里也就是 /api/feign
          filters:
            - StripPrefix=2
      discovery:
        locator:
          enabled: true
          lowerCaseServiceId: true
  
eureka:
#客户端
  client:
#注册中心地址
    service-url:
      defaultZone: http://localhost:1999/eureka/
```

