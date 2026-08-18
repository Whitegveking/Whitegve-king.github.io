

# Web基础

**web程序架构以及基础知识**：

<img width="1139" height="342" alt="Image" src="https://github.com/user-attachments/assets/61935a2f-3e73-4f67-8924-d18ffae21d76" />

- 静态资源： 服务器上存储的不会改变的数据，通常不会根据用户的请求而变化。比如：HTML、CSS、JS、图片、视频等（负责页面展示）
- 动态资源： 服务器端根据用户请求和其他数据动态生成的，内容可能会在每次请求时都发生变化。比如：Servlet、JSP等（负责逻辑处理） Spring框架
- B/S 架构： Browser/Server，浏览器/服务器架构模式。客户端只需浏览器，应用程序的逻辑和数据都存在服务器端。（维护方便 体验一般）
- C/S 架构： Client/Server，客户端/服务器架构模式。需要单独开发维护客户端。（体验不错 开发维护麻烦）

## HTTP协议

**概念**：*Hyper Text Transfer Protocol*，**超文本**传输协议，规定了浏览器和服务器之间数据传输的规则。

<img width="1025" height="195" alt="Image" src="https://github.com/user-attachments/assets/88909605-e823-4511-bb68-7a5ec840b53f" />

**特点**：

1. 基于TCP协议：面向连接，安全
2. 基于请求-响应模型的：一次请求对应一次响应
3. HTTP协议是无状态的协议：对于事务处理没有记忆能力。每次请求-响应都是独立的
   - 缺点：多次请求之间不能共享数据
   - 优点：速度快

## HTTP-请求协议

### 请求数据格式

```http
GET /brand/findAll?name=OPPO&status=1 HTTP/1.1
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Host: localhost:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/...
```

```http
POST /brand HTTP/1.1
Accept: application/json, text/plain, */*
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Content-Length: 161
Content-Type: application/json;charset=UTF-8
Cookie: Idea-8296eb32=841b16f0-0cfe-495a-9cc9-d5aaa71501a6; JSESSIONID=0FDE4E430876BD9C5C955F061207386F
Host: localhost:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/...

{"status":1,"brandName":"黑马","companyName":"黑马程序员","id":"","description":"黑马程序员"}
```

**请求行**：请求数据第一行（请求方式、资源路径、协议）

```http
GET /brand/findAll?name=OPPO&status=1 HTTP/1.1
POST /brand HTTP/1.1
```



**请求头**：第二行开始，格式：`key:value`

```http
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Host: localhost:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/...
Accept: application/json, text/plain, */*
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Content-Length: 161
Content-Type: application/json;charset=UTF-8
Cookie: Idea-8296eb32=841b16f0-0cfe-495a-9cc9-d5aaa71501a6; JSESSIONID=0FDE4E430876BD9C5C955F061207386F
Host: localhost:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/...
```



**请求体**：与请求头之间用空行分隔，表示POST请求，存放请求数据

```http
{"status":1,"brandName":"黑马","companyName":"黑马程序员","id":"","description":"黑马程序员"}
```

**请求头说明**

| 头字段            | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `Host`            | 请求的主机名                                                 |
| `User-Agent`      | 浏览器版本，例如Chrome浏览器的标识类似Mozilla/5.0 ... Chrome/79，IE浏览器的标识类似Mozilla/5.0 (Windows NT ...) like Gecko |
| `Accept`          | 表示浏览器能接收的资源类型，如text/\*,image/\*或者*/\*表示所有 |
| `Accept-Language` | 表示浏览器偏好的语言，服务器可以据此返回不同语言的网页；     |
| `Accept-Encoding` | 表示浏览器可以支持的压缩类型，例如gzip，deflate等。          |
| `Content-Type`    | 请求主体的数据类型。                                         |
| `Content-Length`  | 请求主体的大小（单位：字节）。                               |

**请求方式-GET**：请求参数在请求行中，没有请求体，如:`/brand/findAll?name=OPPO&status=1`。GET请求大小在浏览器中是有限制的。

**请求方式-POST**：请求参数在请求体中，POST请求大小是没有限制的。

### 请求数据获取

Web服务器(Tomcat)对HTTP协议的请求数据进行解析，并进行了封装(HttpServletRequest)，在调用Controller（请求处理类）方法的时候传递给了该方法。这样，就使得程序员不必直接对协议进行操作，让Web开发更加便捷。

**小结**：

1. HTTP请求数据需要程序员自己解析吗？

   不需要，web服务器负责对HTTP请求数据进行解析，并封装为了请求对象

2. 如何获取请求数据?

   `HttpServletRequest`对象里面封装了所有的请求信息



## HTTP-响应协议

### 响应数据格式

```http
HTTP/1.1 200 OK
Content-Type: application/json
Transfer-Encoding: chunked
Date: Tue, 10 May 2022 07:51:07 GMT
Keep-Alive: timeout=60
Connection: keep-alive

[{id: 1, brandName: "阿里巴巴", companyName: "腾讯计算机系统有限公司", description: "玩玩玩"}]

```

**响应行**：响应数据第二行（协议、状态码、描述）

**响应头**：第二行开始，格式：`key:value`

**响应体**：最后一部分，存放响应数据



**状态码**

| 状态码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 1xx    | 响应中-临时状态码，表示请求已经接收，告诉客户端应该继续请求或者如果它已经完成则忽略它。 |
| 2xx    | 成功-表示请求已经被成功接收，处理已完成。                    |
| 3xx    | 重定向-重定向到其他地方；让客户端再发起一次请求以完成整个处理。 |
| 4xx    | 客户端错误-处理发生错误，责任在客户端。如：请求了不存在的资源、客户端未被授权、禁止访问等。 |
| 5xx    | 服务器错误-处理发生错误，责任在服务端。如：程序抛出异常等。  |

**响应头**

| 响应头             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| `Content-Type`     | 表示该响应内容的类型，例如text/html，application/json。      |
| `Content-Length`   | 表示该响应内容的长度（字节数）。                             |
| `Content-Encoding` | 表示该响应压缩算法，例如gzip。                               |
| `Cache-Control`    | 指示客户端应如何缓存，例如max-age=300表示可以最多缓存300秒。 |
| `Set-Cookie`       | 告诉浏览器为当前页面所在的域设置cookie。                     |

**重定向**

<img width="969" height="450" alt="Image" src="https://github.com/user-attachments/assets/6114afec-996d-4bc2-aeca-975011ab4c4a" />

### 响应数据设置

Web服务器对HTTP协议的响应数据进行了封装(HttpServletResponse)，并在调用Controller方法的时候传递给了该方法。这样，就使得程序员不必直接对协议进行操作，让Web开发更加便捷。

### 实例服务端代码

```java
package Demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application
{
    public static void main(String[] args)
    {
        SpringApplication.run(Application.class, args);
    }
}
```

```java
package Demo;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;

@RestController
public class RequestController
{
    @RequestMapping("/request")
    public String request(HttpServletRequest request)
    {
        // 1、获取请求方式
        String method = request.getMethod();
        System.out.println("请求方式：" + method);

        // 2、获取请求url地址
        String url = request.getRequestURL().toString();
        System.out.println("请求url地址：" + url);

        String uri = request.getRequestURI().toString();
        System.out.println("请求uri地址：" + uri);

        // 3、获取请求协议
        String protocol = request.getProtocol();
        System.out.println("请求协议：" + protocol);

        // 4、获取请求参数 -name
        String name = request.getParameter("name");
        System.out.println("请求参数-name：" + name);

        String age = request.getParameter("age");
        System.out.println("请求参数-age：" + age);

        // 5、获取请求头 -Accept
        String accept = request.getHeader("Accept");
        System.out.println("请求头-Accept：" + accept);

        String cookie = request.getHeader("Cookie");
        System.out.println("请求头-Cookie：" + cookie);
        return "OK";
    }
}
```

```java
package Demo;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;

@RestController
public class ResponseController
{
    @RequestMapping("/response")
    public void response(HttpServletResponse response)
    {
        // 1、设置响应状态码
        response.setStatus(HttpServletResponse.SC_OK);
        // response.setStatus(467);

        // 2、设置响应头
        response.setHeader("Content-Type", "text/html;charset=UTF-8");

        // 3、设置响应体
        try {
            response.getWriter().write("Hello, World!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @RequestMapping("/responseEntity")
    public ResponseEntity<String> responseEntity()
    {
        return ResponseEntity
                .status(401)
                .header("name", "ai")
                .body("<h1>hello response</h1>");

    }
}
```

## 分层解耦

### 三层架构

<img width="1230" height="348" alt="Image" src="https://github.com/user-attachments/assets/8d12d0df-25b1-4c92-b281-a794267fd676" />

**基于单一职责原则分为三层**

- controller：控制层，接收前端发送的请求，对请求进行处理，并响应数据。
- service：业务逻辑层，处理具体的业务逻辑。
- dao：数据访问层(Data Access Object)（持久层），负责数据访问操作，包括数据的增、删、改、查。

[项目学习笔记](<C:\Users\72982\Desktop\JAVA学习日记\三层架构小项目学习笔记.md>)

### 分层解耦思想

- 耦合：衡量软件中各个层/各个模块的依赖关联程度
- 内聚：软件中各个功能模块内部的功能联系

### IOC&DI入门

- **控制反转**:Inversion Of Control，简称IOC。**对象的创建控制权由程序自身转移到外部(容器)**，这种思想称为控制反转。
- **依赖注入**:Dependency Injection，简称DI。容器为应用程序提供运行时，所依赖的资源，称之为依赖注入。
- **Bean对象**:I0C容器中创建、管理的对象，称之为Bean。

**实现分层解耦的思路是什么?**

- 将项目中的类交给IOC容器管理(IOC，控制反转)
- 应用程序运行时需要什么对象，直接依赖容器为其提供(DI，依赖注入）

1.如何将一个类交给IOC容器管理?

- `@Component`(注意:是加在实现类上，而非接口上)

2.如何从I0C容器中找到该类型的bean，然后完成依赖注入?

- `@Autowired`

### IOC详解

| 注解        | 说明                 | 位置                                              |
| ----------- | -------------------- | ------------------------------------------------- |
| @Component  | 声明bean的基础注解   | 不属于以下三类时，用此注解                        |
| @Controller | @Component的衍生注解 | 标注在控制层类上                                  |
| @Service    | @Component的衍生注解 | 标注在业务层类上                                  |
| @Repository | @Component的衍生注解 | 标注在数据访问层类上（由于与mybatis整合，用的少） |

> [!CAUTION]
>
> 声明bean的时候，可以通过注解的value属性指定bean的名字，如果没有指定，默认为类名首字母小写

- 前面声明bean的四大注解，要想生效，还需要被组件扫描注解@ComponentScan扫描。
- 该注解虽然没有显式配置，但是实际上已经包含在了启动类声明注解 @SpringBootApplication中，默认扫描的范围是启动类所在包及其子包。

> - `@Controller`：返回视图（如 JSP/Thymeleaf 页面），返回值字符串当作视图名称去解析跳转。
> - `@RestController`：返回数据（如 JSON/XML），返回值字符串直接作为响应体输出给浏览器。
>
> `@RestController` = `@Controller` + `@ResponseBody`，是个组合注解。
>
> 所以：
>
> - 需要返回页面（跳转 HTML）→ 用 `@Controller`
> - 需要返回 JSON/字符串给前端（前后端分离）→ 用 `@RestController`

### DI详解

基于@Autowired进行依赖注入的常见方式有三种

- 属性注入

<img width="540" height="198" alt="Image" src="https://github.com/user-attachments/assets/d2afa188-626b-4f8a-941f-5e64d8d78ab5" />

- 构造函数注入

<img width="573" height="249" alt="Image" src="https://github.com/user-attachments/assets/f01b0190-44b8-4223-bc32-851245ca42d2" />

- setter注入

<img width="546" height="261" alt="Image" src="https://github.com/user-attachments/assets/83d9f88e-75a8-4f12-a257-9d9782ea02af" />

**三种注入方式总结**

#### 1. 属性注入（字段注入）
```java
@RestController
public class UserController {
    @Autowired
    private UserService userService;
}
```

- 优点：代码最简洁、方便快速开发
- 缺点：依赖无法用`final`修饰、不易测试（无法new后再注入）、依赖关系隐蔽，可能会破坏类的封装性

#### 2. 构造函数注入（推荐）
```java
@RestController
public class UserController {
    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

- 优点：依赖不可变（`final`）、对象创建即完整可用、易测试、能从根本**避免循环依赖**、清晰看出类的依赖关系，提高代码的安全性
- 缺点：构造参数过多时代码冗余

> 注：基于@Autowired，当类中只有一个构造方法时，@Autowired可以省略。

#### 3. setter注入
```java
@RestController
public class UserController {
    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```

- 优点：测试时可灵活更换依赖、保持类的封装性，依赖关系清晰
- 缺点：依赖在对象创建后可被修改，可能出现"半初始化"状态

**实现原理**：三种方式都是通过**反射**完成注入——

- 属性注入：`field.set(bean, 依赖)`
- 构造函数注入：`constructor.newInstance(构造参数)`
- setter注入：反射调用setter方法

整个过程由`AutowiredAnnotationBeanPostProcessor`在Bean实例化后、初始化前扫描`@Autowired`并完成赋值。

**关于循环依赖**：

| 注入方式 | 能否解决循环依赖 | 原因 |
| --- | --- | --- |
| 属性注入 | ✅ 能 | 借助三级缓存提前暴露半成品Bean引用 |
| setter注入 | ✅ 能 | 属性创建后再赋值 |
| 构造函数注入 | ❌ 不能 | 构造时必须先拿到完整依赖，两个Bean无法先创建，直接抛`BeanCurrentlyInCreationException` |

> 注：Spring Boot 2.6+ 默认禁止循环依赖（`spring.main.allow-circular-references=false`），即使属性注入也会报错。

**总结**：业务上优先使用**构造函数注入**，循环依赖属于设计坏味道，出现时应通过**重构拆分**解决。

#### @AutoWired详解

- @AutoWired注解，默认是按照类型进行注入的
- 如果存在多个相同类型的bean，将会报出如下错误

```cmd
Field ... in ... required a single bean,but ... were found
```

**解决方案**

| 方案   | 关键字       | 代码                                                         |
| ------ | ------------ | ------------------------------------------------------------ |
| 方案一 | `@Primary`   | ` @Primary @Service public class UserServiceImpl implements UserService { @Override public List<User> list(){ // 省略…… } }` |
| 方案二 | `@Qualifier` | ` @RestController public class UserController { @Autowired @Qualifier("userServiceImpl") private UserService userService; }` |
| 方案三 | `@Resource`  | ` @RestController public class UserController { @Resource(name = "userServiceImpl") private UserService userService; }` |

1、依赖注入的注解

- @Autowired：默认按照类型自动装配
- 如果同类型的bean存在多个：
- @Primary
- @Autowired + @Qualifier
- @Resource

2、@Resource 与 @Autowired区别？

- @Autowired是Spring框架提供的注解，而@Resource是JavaEE规范提供的
- @Autowired默认是按照类型注入，而@Resource默认是按照名称注入