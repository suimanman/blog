---
title: SpringBoot
date: 2024-02-13 09:25:00
author: 岁漫
img: /source/images/6.jpg
top: true
hide: false
cover: true
coverImg: /images/6.jpg
toc: false
mathjax: false
summary: 自定义运行器、配置文件、多环境配置、常用框架、接口文档生成
categories: Markdown
tags:
  - Typora
  - Markdown
---

# 自定义运行器

```java
@Component
public class TestRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("我是自定义执行！");
    }
}
```

# 配置文件

yaml格式：

```yaml
一级目录:
    二级目录:
      三级目录1: 值
      三级目录2: 值
      三级目录List: 
      - 元素1
      - 元素2
      - 元素3
```

我们可以看到，每一级目录都是通过缩进（不能使用Tab，只能使用空格）区分，并且键和值之间需要添加冒号+空格来表示。

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

SpringSecurity和SpringBootMvc配置：

```yaml
spring:  
  #  Spring Mvc相关配置
  mvc:
    static-path-pattern: /static/**   #静态资源解析地址
  # Spring Security 相关配置
  security:
    filter:
      order: -100 #Spring Security 过滤器优先级
    user:
      name: 'admin'   #默认登录用户名
      password: '123456'   #默认登录密码
      roles:    #默认用户的角色
        - admin
        - user
```

## 配置Logback日志

Logback官网：[https://logback.qos.ch](https://logback.qos.ch/)

和JUL一样，Logback也能实现定制化，我们可以编写对应的配置文件，SpringBoot推荐将配置文件名称命名为`logback-spring.xml`表示这是SpringBoot下Logback专用的配置，可以使用SpringBoot 的高级Proﬁle功能，它的内容类似于这样：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 配置 -->
</configuration>
```

最外层由`configuration`包裹，一旦编写，那么就会替换默认的配置，所以如果内部什么都不写的话，那么会导致我们的SpringBoot项目没有配置任何日志输出方式，控制台也不会打印日志。

我们接着来看如何配置一个控制台日志打印，我们可以直接导入并使用SpringBoot为我们预设好的日志格式，在`org/springframework/boot/logging/logback/defaults.xml`中已经帮我们把日志的输出格式定义好了，我们只需要设置对应的`appender`即可：

```xml
<included>
   <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
   <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
   <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />

   <property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
   <property name="CONSOLE_LOG_CHARSET" value="${CONSOLE_LOG_CHARSET:-${file.encoding:-UTF-8}}"/>
   <property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
   <property name="FILE_LOG_CHARSET" value="${FILE_LOG_CHARSET:-${file.encoding:-UTF-8}}"/>

   <logger name="org.apache.catalina.startup.DigesterFactory" level="ERROR"/>
   <logger name="org.apache.catalina.util.LifecycleBase" level="ERROR"/>
   <logger name="org.apache.coyote.http11.Http11NioProtocol" level="WARN"/>
   <logger name="org.apache.sshd.common.util.SecurityUtils" level="WARN"/>
   <logger name="org.apache.tomcat.util.net.NioSelectorPool" level="WARN"/>
   <logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="ERROR"/>
   <logger name="org.hibernate.validator.internal.util.Version" level="WARN"/>
   <logger name="org.springframework.boot.actuate.endpoint.jmx" level="WARN"/>
</included>
```

导入后，我们利用预设的日志格式创建一个控制台日志打印：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!--  导入其他配置文件，作为预设  -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />

    <!--  Appender作为日志打印器配置，这里命名随意  -->
    <!--  ch.qos.logback.core.ConsoleAppender是专用于控制台的Appender  -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>${CONSOLE_LOG_CHARSET}</charset>
        </encoder>
    </appender>

    <!--  指定日志输出级别，以及启用的Appender，这里就使用了我们上面的ConsoleAppender  -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

配置完成后，我们发现控制台已经可以正常打印日志信息了。

### 文件打印

接着我们来看看如何开启文件打印，我们只需要配置一个对应的Appender即可：

```xml
<!--  ch.qos.logback.core.rolling.RollingFileAppender用于文件日志记录，它支持滚动  -->
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <encoder>
        <pattern>${FILE_LOG_PATTERN}</pattern>
        <charset>${FILE_LOG_CHARSET}</charset>
    </encoder>
    <!--  自定义滚动策略，防止日志文件无限变大，也就是日志文件写到什么时候为止，重新创建一个新的日志文件开始写  -->
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <!--  文件保存位置以及文件命名规则，这里用到了%d{yyyy-MM-dd}表示当前日期，%i表示这一天的第N个日志  -->
        <FileNamePattern>log/%d{yyyy-MM-dd}-spring-%i.log</FileNamePattern>
        <!--  到期自动清理日志文件  -->
        <cleanHistoryOnStart>true</cleanHistoryOnStart>
        <!--  最大日志保留时间  -->
        <maxHistory>7</maxHistory>
        <!--  最大单个日志文件大小  -->
        <maxFileSize>10MB</maxFileSize>
    </rollingPolicy>
</appender>

<!--  指定日志输出级别，以及启用的Appender，这里就使用了我们上面的ConsoleAppender  -->
<root level="INFO">
    <appender-ref ref="CONSOLE"/>
    <appender-ref ref="FILE"/>
</root>
```

### MDC机制

Logback内置的日志字段还是比较少，如果我们需要打印有关业务的更多的内容，包括自定义的一些数据，需要借助logback MDC机制，MDC为“Mapped Diagnostic Context”（映射诊断上下文），即将一些运行时的上下文数据通过logback打印出来；此时我们需要借助org.sl4j.MDC类。

比如我们现在需要记录是哪个用户访问我们网站的日志，只要是此用户访问我们网站，都会在日志中携带该用户的ID，我们希望每条日志中都携带这样一段信息文本，而官方提供的字段无法实现此功能，这时就需要使用MDC机制：

```java
@ResponseBody
@GetMapping("/test")
public User test(HttpServletRequest request){
   MDC.put("reqId", request.getSession().getId());
   log.info("用户访问了一次测试数据");
   return mapper.findUserById(1);
}
```

通过这种方式，我们就可以向日志中传入自定义参数了，我们日志中添加这样一个占位符`%X{键值}`，名字保持一致：

```java
%clr([%X{reqId}]){faint}
```

可以使用在线生成网站进行生成自己的个性Banner：https://www.bootschool.net/ascii

创建一个banner.txt文件复制进去就行。

```txt
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//             佛祖保佑          永无BUG         永不修改             //
```

![image-20240205152748975](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240205152748975.png)

我们甚至还可以使用颜色代码来为文本切换颜色(直接加到banner.txt文件前面就行)：

```xml
${AnsiColor.BRIGHT_GREEN}  //绿色
```

# 多环境配置

在日常开发中，我们项目会有多个环境。例如开发环境（develop）也就是我们研发过程中疯狂敲代码修BUG阶段，生产环境（production ）项目开发得差不多了，可以放在服务器上跑了。不同的环境下，可能我们的配置文件也存在不同，但是我们不可能切换环境的时候又去重新写一次配置文件，所以我们可以将多个环境的配置文件提前写好，进行自由切换。

由于SpringBoot只会读取`application.properties`或是`application.yml`文件，那么怎么才能实现自由切换呢？SpringBoot给我们提供了一种方式，我们可以通过配置文件指定：

![image-20240205155043505](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240205155043505.png)

```yaml
spring:
  profiles:
    active: dev
```

接着我们分别创建两个环境的配置文件，`application-dev.yml`和`application-prod.yml`分别表示开发环境和生产环境的配置文件，比如开发环境我们使用的服务器端口为8080，而生产环境下可能就需要设置为80或是443端口，那么这个时候就需要不同环境下的配置文件进行区分：

```yaml
server:
  port: 8080
```

```yaml
server:
  port: 80
```

这样我们就可以灵活切换生产环境和开发环境下的配置文件了。

SpringBoot自带的Logback日志系统也是支持多环境配置的，比如我们想在开发环境下输出日志到控制台，而生产环境下只需要输出到文件即可，这时就需要进行环境配置：

```xml
<springProfile name="dev">
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</springProfile>

<springProfile name="prod">
    <root level="INFO">
        <appender-ref ref="FILE"/>
    </root>
</springProfile>
```

![image-20240205161334868](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240205161334868.png)

那如果我们希望生产环境中不要打包开发环境下的配置文件呢，我们目前虽然可以切换开发环境，但是打包的时候依然是所有配置文件全部打包，这样总感觉还欠缺一点完美，因此，打包的问题就只能找Maven解决了，Maven也可以设置多环境:

放到pom.xml文件中

```xml
<!--分别设置开发，生产环境-->
<profiles>
    <!-- 开发环境 -->
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <environment>dev</environment>
        </properties>
    </profile>
    <!-- 生产环境 -->
    <profile>
        <id>prod</id>
        <activation>
            <activeByDefault>false</activeByDefault>
        </activation>
        <properties>
            <environment>prod</environment>
        </properties>
    </profile>
</profiles>
```

![image-20240205161421949](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240205161421949.png)

接着，我们需要根据环境的不同，排除其他环境的配置文件：

```xml
<resources>
<!--排除配置文件-->
    <resource>
        <directory>src/main/resources</directory>
        <!--先排除所有的配置文件-->
        <excludes>
            <!--使用通配符，当然可以定义多个exclude标签进行排除-->
            <exclude>application*.yml</exclude>
        </excludes>
    </resource>

    <!--根据激活条件引入打包所需的配置和文件-->
    <resource>
        <directory>src/main/resources</directory>
        <!--引入所需环境的配置文件-->
        <filtering>true</filtering>
        <includes>
            <include>application.yml</include>
            <!--根据maven选择环境导入配置文件-->
            <include>application-${environment}.yml</include>
        </includes>
    </resource>
</resources>
```

![image-20240205161441157](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240205161441157.png)

接着，我们可以直接将Maven中的`environment`属性，传递给SpringBoot的配置文件，在构建时替换为对应的值：

```yaml
spring:
  profiles:
    active: '@environment@'  #注意YAML配置文件需要加单引号，否则会报错
```

![image-20240205161509877](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240205161509877.png)

这样，根据我们Maven环境的切换，SpringBoot的配置文件也会进行对应的切换。

![image-20240205161606350](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240205161606350.png)

最后我们打开Maven栏目，就可以自由切换了，直接勾选即可，注意切换环境之后要重新加载一下Maven项目，不然不会生效！

![image-20240205162032913](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240205162032913.png)

# 常用框架介绍

## 邮件发送

![image-20230716172901937](https://image.itbaima.cn/markdown/2023/07/16/sL56YdmgGblfFjo.png)

比较常用的协议有两种：

1. SMTP协议（主要用于发送邮件 Simple Mail Transfer Protocol）
2. POP3协议（主要用于接收邮件 Post Office Protocol 3）

整个发送/接收流程大致如下：

![img](https://image.itbaima.cn/markdown/2023/07/16/sOyWQguFonJKXNw.jpg)

实际上每个邮箱服务器都有一个smtp发送服务器和pop3接收服务器，比如要从QQ邮箱发送邮件到163邮箱，那么我们只需要通过QQ邮箱客户端告知QQ邮箱的smtp服务器我们需要发送邮件，以及邮件的相关信息，然后QQ邮箱的smtp服务器就会帮助我们发送到126邮箱的pop3服务器上，126邮箱会通过126邮箱客户端告知对应用户收到一封新邮件。

而我们如果想要实现给别人发送邮件，那么就需要连接到对应电子邮箱的smtp服务器上，并告知其我们要发送邮件。而SpringBoot已经帮助我们将最基本的底层通信全部实现了，我们只需要关心smtp服务器的地址以及我们要发送的邮件长啥样即可。

这里以163邮箱 [https://mail.126.com](https://mail.126.com/) 为例，我们需要在配置文件中告诉SpringBootMail我们的smtp服务器的地址以及你的邮箱账号和密码，首先我们要去设置中开启smtp/pop3服务才可以，开启后会得到一个随机生成的密钥，这个就是我们的密码。

导入依赖：

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-mail -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
    <version>3.2.1</version>
</dependency>
```

进行yml配置：

![image-20240205165154391](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240205165154391.png)

```yaml
 spring:
     mail:
        host: smtp.126.com
        username: wmc19831983050@126.com
        # 注意密码是在开启smtp/pop3时自动生成的，记得保存一下，不然就找不到了
        password: OBHYSHYPWEUNCELV
```



xxxxxxxxxx <input type="checkbox" name="remember" class="ad-checkbox">html

配置完成后，接着我们来进行一下测试：

```java
@SpringBootTest
class SpringBootTestApplicationTests {

      //JavaMailSender是专门用于发送邮件的对象，自动配置类已经提供了Bean
    @Autowired
    JavaMailSender sender;

    @Test
    void contextLoads() {
          //SimpleMailMessage是一个比较简易的邮件封装，支持设置一些比较简单内容
        SimpleMailMessage message = new SimpleMailMessage();
          //设置邮件标题
        message.setSubject("【河北工业大学教务处】关于近期学校对您的处分决定");
          //设置邮件内容
        message.setText("XXX同学您好，经监控和教务巡查发现，您近期存在旷课、迟到、早退、上课刷抖音行为，" +
                "现已通知相关辅导员，请手写5000字书面检讨，并在2022年4月1日17点前交到辅导员办公室。");
          //设置邮件发送给谁，可以多个，这里就发给你的QQ邮箱
        message.setTo("@qq.com");
          //邮件发送者，这里要与配置文件中的保持一致
        message.setFrom("javastudy111@163.com");
          //OK，万事俱备只欠发送
        sender.send(message);
    }

}
```

## 实现邮件注册

利用session存储邮件地址以及发送的验证码，在用户接受到验证码输入后进行注册时，对其输入的邮件和验证码和session中存储的进行对比。

```java
 @Autowired
    JavaMailSender mailSender;
    @Autowired
    UserMapper mapper;
    @PostMapping("/code")
    public String getCode(@RequestParam String email,
            HttpSession session){
        Random random=new Random();
        int code= random.nextInt(900000)+100000;//保证验证码是6位
        session.setAttribute("code",code);
        session.setAttribute("email",email);
        SimpleMailMessage message=new SimpleMailMessage();
        message.setSubject("你的验证码");
        message.setText("验证码是："+code+"，有效时间3分钟，请妥善保存！");
        message.setTo(email);
        message.setFrom("wmc19831983050@126.com");
        mailSender.send(message);
        System.out.println(message);
        return "发送成功";
    }
    @PostMapping("/register")
    public String register(@RequestParam String username,
                           @RequestParam String password,
                           @RequestParam String email,
                           @RequestParam int code,
                           HttpSession session){
        String sessionEmail=(String) session.getAttribute("email");
        Integer sessionCode=(Integer) session.getAttribute("code");
        System.out.println(sessionCode);
        System.out.println(code);
        if(sessionCode==null) return "请先获取验证码！";
        if(!sessionEmail.equals(email)) return "请先获取验证码";
        if(sessionCode!=code) return "验证码错误！";
        mapper.createUser(username,password,email);//注册成功，存到数据库中
        return "注册成功！";

    }
```

# 接口规则校验

通常我们在使用SpringMvc框架编写接口时，很有可能用户发送的数据存在一些问题，比如下面这个接口：

```java
@ResponseBody
@PostMapping("/submit")
public String submit(String username,
                     String password){
    System.out.println(username.substring(3));
    System.out.println(password.substring(2, 10));
    return "请求成功!";
}
```

这个接口中，我们需要将用户名和密码分割然后打印，在正常情况下，因为用户名长度规定不小于5，如果用户发送的数据是没有问题的，那么就可以正常运行，这也是我们所希望的情况，但是如果用户发送的数据并不是按照规定的，那么就会直接报错：

![image-20230716215850225](https://image.itbaima.cn/markdown/2023/07/16/n1FMADOiQCRcGw6.png)

这个时候，我们就需要在请求进来之前进行校验了，最简单的办法就是判断一下：

```java
@ResponseBody
@PostMapping("/submit")
public String submit(String username,
                     String password){
    if(username.length() > 3 && password.length() > 10) {
        System.out.println(username.substring(3));
        System.out.println(password.substring(2, 10));
        return "请求成功!";
    } else {
        return "请求失败";
    }
}
```

虽然这样就能直接解决问题，但是如果我们的每一个接口都需要这样去进行配置，那么是不是太麻烦了一点？SpringBoot为我们提供了很方便的接口校验框架：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

现在，我们可以直接使用注解完成全部接口的校验：

```java
@Slf4j
@Validated   //首先在Controller上开启接口校验
@Controller
public class TestController {

    ...

    @ResponseBody
    @PostMapping("/submit")
    public String submit(@Length(min = 3) String username,  //使用@Length注解一步到位
                         @Length(min = 10) String password){
        System.out.println(username.substring(3));
        System.out.println(password.substring(2, 10));
        return "请求成功!";
    }
}
```

现在，我们的接口校验就可以快速进行配置了，一个接口就能搞定：

![image-20230716220839816](https://image.itbaima.cn/markdown/2023/07/16/EibCc4sHWflywek.png)

不过这样依然会抛出一个异常，对用户不太友好，我们可以稍微处理一下，这里我们可以直接使用之前在SSM阶段中学习的异常处理Controller来自行处理这类异常：

```java
@ControllerAdvice
public class ValidationController {

    @ResponseBody
    @ExceptionHandler(ConstraintViolationException.class)
    public String error(ValidationException e){
        return e.getMessage();   //出现异常直接返回消息
    }
}
```

![image-20230716221420324](https://image.itbaima.cn/markdown/2023/07/16/7JH6BzOhlUe9gkG.png)

除了@Length之外，我们也可以使用其他的接口来实现各种数据校验：

|   验证注解   |                        验证的数据类型                        |                           说明                           |
| :----------: | :----------------------------------------------------------: | :------------------------------------------------------: |
| @AssertFalse |                       Boolean,boolean                        |                      值必须是false                       |
| @AssertTrue  |                       Boolean,boolean                        |                       值必须是true                       |
|   @NotNull   |                           任意类型                           |                       值不能是null                       |
|    @Null     |                           任意类型                           |                       值必须是null                       |
|     @Min     | BigDecimal、BigInteger、byte、short、int、long、double 以及任何Number或CharSequence子类型 |                   大于等于@Min指定的值                   |
|     @Max     |                             同上                             |                   小于等于@Max指定的值                   |
| @DecimalMin  |                             同上                             |         大于等于@DecimalMin指定的值（超高精度）          |
| @DecimalMax  |                             同上                             |         小于等于@DecimalMax指定的值（超高精度）          |
|   @Digits    |                             同上                             |                限制整数位数和小数位数上限                |
|    @Size     |               字符串、Collection、Map、数组等                |       长度在指定区间之内，如字符串长度、集合大小等       |
|    @Past     |       如 java.util.Date, java.util.Calendar 等日期类型       |                    值必须比当前时间早                    |
|   @Future    |                             同上                             |                    值必须比当前时间晚                    |
|  @NotBlank   |                     CharSequence及其子类                     |         值不为空，在比较时会去除字符串的首位空格         |
|   @Length    |                     CharSequence及其子类                     |                  字符串长度在指定区间内                  |
|  @NotEmpty   |         CharSequence及其子类、Collection、Map、数组          | 值不为null且长度不为空（字符串长度不为0，集合大小不为0） |
|    @Range    | BigDecimal、BigInteger、CharSequence、byte、short、int、long 以及原子类型和包装类型 |                      值在指定区间内                      |
|    @Email    |                     CharSequence及其子类                     |                     值必须是邮件格式                     |
|   @Pattern   |                     CharSequence及其子类                     |               值需要与指定的正则表达式匹配               |
|    @Valid    |                        任何非原子类型                        |                     用于验证对象属性                     |

虽然这样已经很方便了，但是在遇到对象的时候，依然不太方便，比如：

```java
@Data
public class Account {
    String username;
    String password;
}
```

```java
@ResponseBody
@PostMapping("/submit")
public String submit(Account account){   //直接使用对象接收
    System.out.println(account.getUsername().substring(3));
    System.out.println(account.getPassword().substring(2, 10));
    return "请求成功!";
}
```

此时接口是以对象形式接收前端发送的表单数据的，这个时候就没办法向上面一样编写对应的校验规则了，那么现在又该怎么做呢？

对应对象类型，我们也可以进行验证，方法如下：

```java
@ResponseBody
@PostMapping("/submit")  //在参数上添加@Valid注解表示需要验证
public String submit(@Valid Account account){
    System.out.println(account.getUsername().substring(3));
    System.out.println(account.getPassword().substring(2, 10));
    return "请求成功!";
}
```

```java
@Data
public class Account {
    @Length(min = 3)   //只需要在对应的字段上添加校验的注解即可
    String username;
    @Length(min = 10)
    String password;
}
```

这样当受到请求时，就会对对象中的字段进行校验了，这里我们稍微修改一下ValidationController的错误处理，对于实体类接收参数的验证，会抛出MethodArgumentNotValidException异常，这里也进行一下处理：

```java
@ResponseBody
@ExceptionHandler({ConstraintViolationException.class, MethodArgumentNotValidException.class})
public String error(Exception e){
    if(e instanceof ConstraintViolationException exception) {
        return exception.getMessage();
    } else if(e instanceof MethodArgumentNotValidException exception){
        if (exception.getFieldError() == null) return "未知错误";
        return exception.getFieldError().getDefaultMessage();
    }
    return "未知错误";
}
```

这样就可以正确返回对应的错误信息了。

# 接口文档生成

在后续学习前后端分离开发中，前端现在由专业的人来做，而我们往往只需要关心后端提供什么接口给前端人员调用，我们的工作被进一步细分了，这个时候为前端开发人员提供一个可以参考的文档是很有必要的。

但是这样的一个文档，我们也不可能单独写一个项目去进行维护，并且随着我们的后端项目不断更新，文档也需要跟随更新，这显然是很麻烦的一件事情，那么有没有一种比较好的解决方案呢？

当然有，那就是丝袜哥：Swagger

Swagger的主要功能如下：

- 支持 API 自动生成同步的在线文档：使用 Swagger 后可以直接通过代码生成文档，不再需要自己手动编写接口文档了，对程序员来说非常方便，可以节约写文档的时间去学习新技术。
- 提供 Web 页面在线测试 API：光有文档还不够，Swagger 生成的文档还支持在线测试。参数和格式都定好了，直接在界面上输入参数对应的值即可在线测试接口。

结合Spring框架（Spring-doc，官网：https://springdoc.org/），Swagger可以很轻松地利用注解以及扫描机制，来快速生成在线文档，以实现当我们项目启动之后，前端开发人员就可以打开Swagger提供的前端页面，查看和测试接口。依赖如下：

```java
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.1.0</version>
</dependency>
```

项目启动之后，我们可以直接访问：http://localhost:8080/swagger-ui/index.html，就能看到我们的开发文档了：

![image-20240207101351255](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207101351255.png)

可以看到这个开发文档中自动包含了我们定义的接口，并且还有对应的实体类也放在了下面。这个页面不仅仅是展示接口，也可以直接在上面进行调试：

![image-20240207101541342](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207101541342.png)

这就非常方便了，不仅前端人员可以快速查询接口定义，我们自己也可以在线进行接口测试，直接抛弃PostMan之类的软件了。

虽然Swagger的UI界面已经可以很好地展示后端提供的接口信息了，但是非常的混乱，我们来看看如何配置接口的一些描述信息。首先我们的页面肯定要展示一下这个文档的一些信息，只需要一个Bean就能搞定：

```java
@Bean
public OpenAPI springDocOpenAPI() {
        return new OpenAPI().info(new Info()
                        .title("图书管理系统 - 在线API接口文档")   //设置API文档网站标题
                        .description("这是一个图书管理系统的后端API文档，欢迎前端人员查阅！") //网站介绍
                        .version("2.0")   //当前API版本
                        .license(new License().name("我的B站个人主页")  //遵循的协议，这里拿来写其他的也行
                                .url("https://space.bilibili.com/37737161")));
}
```

![image-20240207101905233](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207101905233.png)

![image-20240207101914994](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207101914994.png)

接着我们来看看如何为一个Controller编写API描述信息：

```java
//使用@Tag注解来添加Controller描述信息
@Tag(name = "账户验证相关", description = "包括用户登录、注册、验证码请求等操作。")
public class TestController {
	...
}
```

![image-20240207102156222](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207102156222.png)

我们可以直接在类名称上面添加`@Tag`注解，并填写相关信息，来为当前的Controller设置描述信息。接着我们可以为所有的请求映射配置描述信息：

```java
@RestController
//使用@Tag注解来添加Controller描述信息
@Tag(name = "账户验证相关", description = "包括用户登录、注册、验证码请求等操作。")
public class RegisterController {
    @Autowired
    JavaMailSender mailSender;
    @Autowired
    UserMapper mapper;
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "测试成功"),
            @ApiResponse(responseCode = "500", description = "测试失败")   //不同返回状态码描述
    })
    @Operation(summary = "获取验证码",description = "填取email地址，获取验证码")
    @PostMapping("/code")
    public String getCode(@Parameter(description = "邮件地址",example = "1847515809@qq.com") @RequestParam String email,
            HttpSession session){
        Random random=new Random();
        int code= random.nextInt(900000)+100000;
        session.setAttribute("code",code);
        session.setAttribute("email",email);
        SimpleMailMessage message=new SimpleMailMessage();
        message.setSubject("你的验证码");
        message.setText("验证码是："+code+"，有效时间3分钟，请妥善保存！");
        message.setTo(email);
        message.setFrom("wmc19831983050@126.com");
        mailSender.send(message);
        System.out.println(message);
        return "发送成功";
    }
}

```



![image-20240207102530532](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207102530532.png)

![image-20240207102459198](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207102459198.png)

![image-20240207103013755](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207103013755.png)

![image-20240207102917053](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207102917053.png)

![image-20240207103130202](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207103130202.png)

![image-20240207103114370](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207103114370.png)

对于那些不需要展示在文档中的接口，我们也可以将其忽略掉：

```java
@Hidden
@ResponseBody
@GetMapping("/hello")
public String hello() {
    return "Hello World";
}
```

对于实体类，我们也可以编写对应的API接口文档：

![image-20240207104037352](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207104037352.png)

```java
@Data
@AllArgsConstructor
@Schema(description = "用户实体类")
public class User {
    @Schema(description = "ID")
    public Integer id;
    @Schema(description = "用户名")
    public String username;
    @Schema(description = "邮箱")
    public String email;
    @Schema(description = "密码")
    public String password;
}
```

![image-20240207104217338](C:\Users\茉莉花茶\AppData\Roaming\Typora\typora-user-images\image-20240207104217338.png)

这样，我们就可以在文档中查看实体类简介以及各个属性的介绍了。

不过，这种文档只适合在开发环境下生成，如果是生产环境，我们需要在xml配置文件里面关闭文档：

```xml
springdoc:
  api-docs:
    enabled: false
```

# 分页

## 后端

mybatis-plus分页配置文件:

```java
@Configuration
@ComponentScan("com.daocao.*")
public class MyBatisPlusConfig {
    /**
     * 添加分页插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));//如果配置多个插件,切记分页最后添加
        //interceptor.addInnerInterceptor(new PaginationInnerInterceptor()); 如果有多数据源可以不配具体类型 否则都建议配上具体的DbType
        return interceptor;
    }
}
```

