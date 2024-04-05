---
title: SpringCloud
date: 2024-02-14 09:25:00
author: 岁漫
img: /images/4.jpg
top: true
hide: false
cover: true
coverImg: /images/4.jpg
toc: false
mathjax: false
summary: Eureka、LoadBalancer、GateWay
categories: Markdown
tags:
  - java后端
  - SpringCloud
---

# Eureka

能够自动注册并发现微服务，对服务进行管理。用到其他服务时，直接想Eureka询问。

![image-20240301152011158](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240301152011158.png)

Eureka集群

配置多个Eureka，防止一个挂掉导致所有服务之间都通信不了。

# LoadBalancer

负责对一个服务的多个实例资源进行调动分配，默认分配方式是轮循，可以更改。

## OpenFeign实现负载均衡

Feign和RestTemplate一样，都是HTTP客户端请求工具，但是它的使用方式更方便。

在需要调用其他服务模块的服务中加入以下：

依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

在启动类中添加`@EnableFeignClients`注解

```java 
@SpringBootApplication
@EnableFeignClients
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class, args);
    }
}
```

创建需要调用其他微服务的对应接口类：

```java
@FeignClient("userservice")   //声明为userservice服务的HTTP请求客户端
public interface UserClient {
}
```

然后可以直接将对应服务接口所需方法拿过来

```java
@FeignClient("userservice")
public interface UserClient {

  	//路径保证和其他微服务提供的一致即可
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);  //参数和返回值也保持一致
}
```

然后就可以直接注入使用了：

```java
@Service
public class BorrowServiceImpl implements BorrowService {

    @Resource
    BorrowMapper mapper;

    @Resource
    UserClient userClient;
    
    @Resource
    BookClient bookClient;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);

        User user = userClient.getUserById(uid);
        List<Book> bookList = borrow
                .stream()
                .map(b -> bookClient.getBookById(b.getBid()))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user, bookList);
    }
}
```

## 服务降级

提供一个补救措施，将补救结果正常返回给请求者，而不会直接返回错误，相当于是服务依然可用，但是服务能力下降。

## 服务熔断

熔断机制是应对雪崩效应的一种微服务链路保护机制，当检测出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，返回错误信息，**每隔一段时间后再次进行检测**，而当检测到该节点微服务响应正常后恢复调用链路。恢复之前可直接调用服务降级中的补救措施。

## OpenFeign实现降级

# GateWay网关

路由机制，添加一层防护，让所有请求都通过路由来转发到各个微服务，并且转发给多个相同微服务也可以实现负载均衡。

![image-20230306230524100](https://image.itbaima.cn/markdown/2023/03/06/gMwst5OGfvPCTd8.png)
