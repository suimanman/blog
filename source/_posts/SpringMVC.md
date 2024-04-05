---
​---
title: SpringMVC
date: 2024-02-15 09:25:00
author: 岁漫
img: /source/images/8.jpg
top: true
hide: false
cover: true
coverImg: /images/8.jpg
toc: false
mathjax: false
summary: 重定向、请求转发、RestFul、拦截器、JSON数据格式与Axios请求
categories: Markdown
tags:
  - Typora
  - Markdown
​---
---

# 重定向

将请求路径重定向到另外一个路径，加一个前缀**redierct** 

```java
@RequestMapping("/index")
public String index(){
    return "redirect:home";
}
@RequestMapping("/home")
public String home(){
    return "home";
}
```

# 请求转发

将请求转发给另一个请求路径去处理，但不会改变路径本身，加一个前缀**forward**

```java
@RequestMapping("/index")
public String index(){
    return "forward:home";
}
@RequestMapping("/home")
public String home(){
    return "home";
}
```



# Bean的Web作用域

在要注册为Bean的类上加上注解**@Requestscope** 或者**@SessionScope** 

* request：对于每次HTTP请求，使用request作用域定义的Bean都将产生一个新实例，请求结束Bean也消失。
* session：对于每一个会话，使用session作用域定义的Bean都将产生一个新实例，会话过期Bean也消失。

# RestFul风格

中文名“表现层状态转换”，将参数通过URL拼接传到服务端，目的是让URL看起来更加简洁实用。

直接从请求路径中读取参数：

```java
http://localhost:8080/user/1
```

使用**@PathVariable** 注解直接获取请求参数

```java
@RequestMapping("user/{id}")
public String login(@PathVariable String id){
    return "login";
}
```

# 拦截器

拦截器是整个SpringMVC的一个重要内容，拦截器与过滤器类似，都是用于拦截一些非法请求，但是我们之前讲解的过滤器是作用于Servlet之前，只有经过层层的拦截器才可以成功到达Servlet，而拦截器并不是在Servlet之前，它在Servlet与RequestMapping之间，相当于DispatcherServlet在将请求交给对应Controller中的方法之前进行拦截处理，它只会拦截所有Controller中定义的请求映射对应的请求（不会拦截静态资源），这里一定要区分两者的不同。

![image-20240202164726999](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240202164726999.png)

## 创建拦截器

创建一个拦截器我们需要实现一个`HandlerInterceptor`接口：

```java
public class MainInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("我是处理之前！");
        return true;   //只有返回true才会继续，否则直接结束
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("我是处理之后！");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("我是完成之后！");
    }
}
```

接着我们需要在配置类中进行注册：

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new MainInterceptor())
      .addPathPatterns("/**")    //添加拦截器的匹配路径，只要匹配一律拦截
      .excludePathPatterns("/home");   //拦截器不进行拦截的路径
}
```

得到整理拦截器的执行顺序：

>
>
>我是处理之前！
>
>我是处理！
>
>我是处理之后！
>
>我是完成之后！

如果处理之前的返回值为false，那么就不会进入Controller的方法。

如果执行方法过程中发生了异常，结果：

>
>
>我是处理之前！
>我是处理！
>我是完成之后！

我们发现如果处理过程中抛出异常，那么久不会执行处理后`postHandle`方法，但是会执行`afterCompletion`方法，我们可以在此方法中获取到抛出的异常。

## 多级拦截器

拦截顺序就是注册的顺序，因此拦截器会根据注册顺序依次执行。如果前一个拦截器返回了false，那么后面的拦截器也不会继续执行。

# JSON数据格式与Axios请求

将JSON格式转换成js对象：

```javascript
let obj = JSON.parse('{"studentList": [{"name": "杰哥", "age": 18}, {"name": "阿伟", "age": 18}], "count": 2}')
//将JSON格式字符串转换为JS对象
obj.studentList[0].name   //直接访问第一个学生的名称
```

将JS对象转换为JSON字符串：

```java
JSON.stringify(obj)
```

后端通过JSON格式传给前端。

## 如何创建JSON格式数据

### 导入依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.78</version>
</dependency>
```

```java
@RequestMapping(value = "/index")
public String index(){
    JSONObject object = new JSONObject();
    object.put("name", "杰哥");
    object.put("age", 18);
    System.out.println(object.toJSONString());   //以JSON格式输出JSONObject字符串
    return "index";
}
```

对于实体类

```java
@RequestMapping(value = "/index", produces = "application/json")
@ResponseBody
public String data(){
    Student student = new Student();
    student.setName("杰哥");
    student.setAge(18);
    return JSON.toJSONString(student);
}
```

如果想直接返回student，不用JSON.toJSONString:

 注意需要在配置类中添加一下FastJSON转换器（默认只支持JackSon）：

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters.add(new FastJsonHttpMessageConverter());
}
```

