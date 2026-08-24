# Web后端实战-Tlias案例

## 准备工作

### 开发规范-开发模式

**前后端混合开发**

- 分工不明确
- 不便管理
- 难以维护

**前后端分离开发**

- 当前最为主流的开发模式：前后端分离

<img width="1152" height="621" alt="Image" src="https://github.com/user-attachments/assets/b6c2d44b-d972-4ec2-8036-c6b92c049b7f" />

### 开发规范-Restful风格

**REST**(REpresentational State Transfer)，表述性状态转换，它是一种软件架构风格。

| 传统风格url                                | 请求方式 | 含义            | 备注   |
| ------------------------------------------ | -------- | --------------- | ------ |
| http://localhost:8080/user/getById?id=1    | GET      | 查询id为1的用户 | 不规范 |
| http://localhost:8080/user/saveUser        | POST     | 新增用户        | 难维护 |
| http://localhost:8080/user/updateUser      | POST     | 修改用户        |        |
| http://localhost:8080/user/deleteUser?id=1 | GET      | 删除id为1的用户 |        |

| REST风格url                   | 请求方式 | 含义            | 备注             |
| ----------------------------- | -------- | --------------- | ---------------- |
| http://localhost:8080/users/1 | GET      | 查询id为1的用户 | URL定位资源      |
| http://localhost:8080/users/1 | DELETE   | 删除id为1的用户 | HTTP动词描述操作 |
| http://localhost:8080/users   | POST     | 新增用户        | 简洁、规范、优雅 |
| http://localhost:8080/users   | PUT      | 修改用户        |                  |

> [!CAUTION]
>
> 1. REST是风格，是约定方式，约定不是规定，可以打破。
> 2. 描述功能模块通常使用复数形式(加s)，表示此类资源，而非单个资源。如:users、books...

### 项目框架讲解

#### 整体架构：三层架构

```java
浏览器请求 → DeptController（接收请求、响应数据）
                ↓ 调用
            DeptService 接口 ← DeptServiceImpl（业务逻辑处理）
                ↓ 调用
            DeptMapper（数据库操作，MyBatis）
                ↓
              MySQL 数据库
```

| 层     | 包名                       | 职责                                           |
| ------ | -------------------------- | ---------------------------------------------- |
| 表现层 | `controller`               | 接收前端请求、参数校验、返回 `Result` 统一响应 |
| 业务层 | `service` + `service.impl` | 编写业务逻辑（如删除部门时同时删除员工）       |
| 数据层 | `mapper`                   | 只负责 CRUD，操作数据库                        |
| 实体类 | `pojo`                     | `Dept` 封装数据、`Result` 统一响应结果         |

#### 用到的注解

**Spring Boot 核心：**

- `@SpringBootApplication` —— 启动类注解，包含自动配置 + 组件扫描（扫描启动类所在包及子包）
- `@RestController` —— = `@Controller` + `@ResponseBody`，声明为 Bean 且方法返回值直接作为 JSON 响应
- `@Service` —— 声明业务层 Bean，交给 IOC 容器管理

**MyBatis：**

- `@Mapper` —— 声明数据层接口，运行时由 MyBatis 自动生成代理实现并注入容器

**Lombok：**

- `@Data` —— 自动生成 getter/setter/toString/equals/hashCode
- `@NoArgsConstructor` / `@AllArgsConstructor` —— 自动生成空参/全参构造方法

#### Service 为什么要有接口和 impl 实现，而 Mapper 只要接口？

核心区别在于：**谁来生成实现类**。

**Mapper 不需要写实现 —— MyBatis 用动态代理帮你生成了**

```java
@Mapper
public interface DeptMapper { ... }
```

只写接口，MyBatis 在运行时通过 **JDK 动态代理** 自动生成实现对象：

- 拦截方法调用 → 根据方法上的 SQL 注解或 XML 找到 SQL → 执行 JDBC → 把结果封装成对象返回

所以实现类是"框架造的"，自然不用写。

**Service 需要自己写实现 —— 因为业务逻辑是自己写的代码**

```java
public class DeptServiceImpl implements DeptService { }
```

业务逻辑没有框架能替你生成，必须自己写。拆成接口+实现的原因：

1. **解耦**：Controller 依赖的是 `DeptService` 接口而不是具体实现类，将来换实现（如本地存储换成 OSS 存储），Controller 一行代码都不用改。
2. **多态与可替换**：一个接口可以有多个实现类，按需切换。
3. **方便测试**：单元测试时可以 Mock 一个假的 Service 实现，不用跑真实业务。
4. **规范约定**："接口定义做什么（规范），实现类定义怎么做"，是企业开发的通用分层规范。

> [!NOTE]
>
> Spring 并不强制 Service 必须有接口，直接在类上标 `@Service` 也能运行。拆接口是工程规范和习惯做法；而 Mapper 是 MyBatis 的机制决定的"只能有接口"。

## 接口开发-部门管理

<img width="1248" height="366" alt="Image" src="https://github.com/user-attachments/assets/c588b2f2-5bf1-4b29-96da-ed4279bfe670" />

### 当前接口代码结构（部门查询为例）

**Controller：DeptController**

```java
@RestController
public class DeptController {
    @Autowired
    private DeptService deptService;

    @RequestMapping("/dept")
    public Result list() {
        System.out.println("查询全部部门数据");
        List<Dept> deptlist = deptService.findAll();
        return Result.success(deptlist);
    }
}
```

**Service 接口：DeptService**

```java
public interface DeptService {
    List<Dept> findAll();
}
```

**Service 实现：DeptServiceImpl**

```java
@Service
public class DeptServiceImpl implements DeptService {
    @Autowired // 依赖注入
    private DeptMapper deptMapper;

    @Override
    public List<Dept> findAll() {
        return deptMapper.findAll();
    }
}
```

**Mapper：DeptMapper**

```java
@Mapper
public interface DeptMapper {
    @Select("select id, name, create_time, update_time from dept order by update_time desc;")
    List<Dept> findAll();
}
```

**实体类：Dept、Result**

- `Dept`：使用 Lombok `@Data`、`@NoArgsConstructor`、`@AllArgsConstructor`，字段：id、name、createTime、updateTime
- `Result`：统一响应封装，code/msg/data，提供 `success()`、`success(Object)`、`error(String)` 静态方法

### 遇到的问题与解决

#### 问题 1：请求返回 500，NullPointerException

**现象**：调用 `/dept` 接口报错：

```java
java.lang.NullPointerException
    at com.cyy.tliaswebmanagement.service.impl.DeptServiceImpl.findAll(DeptServiceImpl.java:19)
```

**原因**：`DeptServiceImpl` 中 `deptMapper` 字段**未加 `@Autowired`**，Spring 没有注入 Mapper Bean，字段为 `null`，调用 `deptMapper.findAll()` 抛空指针。

**对比**：Controller 里 `deptService` 有 `@Autowired`，所以能正常注入；Service 里漏写了。

**修复**：给 `deptMapper` 加上 `@Autowired`，或改用构造器注入（推荐）。

#### 问题 2：`@Autowired` 字段注入会有 IDEA 警告“字段注入不推荐”

**现象**：IntelliJ IDEA 在 `@Autowired private DeptMapper deptMapper;` 处显示黄色波浪线/提示。

**本质**：这是 **IDEA 代码检查警告（Inspection）**，不是编译错误。程序照样能跑，Spring 也能正常注入。

**为什么不推荐字段注入**：

1. **无法用 `final`**，对象可变
2. **掩盖依赖爆炸**：类里塞十几个 `@Autowired` 字段看不出问题，构造器参数一长串立刻显眼
3. **不利于单元测试**：脱离容器只能靠反射注入 Mock
4. **强依赖 IOC 容器**：自己 `new DeptServiceImpl()` 字段全是 null —— 正是刚才 NPE 的根源

**推荐写法：构造器注入（配合 Lombok）**

```java
@Service
@RequiredArgsConstructor // 为所有 final 字段生成构造器
public class DeptServiceImpl implements DeptService {
    private final DeptMapper deptMapper;
    // 自动拥有 public DeptServiceImpl(DeptMapper deptMapper) { this.deptMapper = deptMapper; }
}
```

> [!NOTE]
>
> 学习阶段跟教程用字段注入完全没问题；企业项目里更常用构造器注入。

### 删除部门

#### 方式一：Controller接收简单参数

```java
@DeleteMapping("/depts")
    public Result delete(HttpServletRequest request) {
        String idStr = request.getParameter("id");
        int id = Integer.parseInt(idStr);
        System.out.println("根据ID删除部门数据，id=" + id);
        return Result.success();
    }
```

#### 方式二：通过Spring提供的@RequestParam注解，将请求参数绑定给方法形参

一旦声明了@RequestParam，该参数在请求时必须传递，如果不传递将会报错。（默认 required 为 true）

```java
@DeleteMapping("/depts")
    public Result delete(@RequestParam("id") Integer deptId) {

        System.out.println("根据ID删除部门数据，id=" + deptId);
        return Result.success();
    }
```

#### 方式三：如果请求参数名与形参变量名相同，直接定义方法形参即可接收。(省略RequestParam)(推荐)

```java
@DeleteMapping("/depts")
    public Result delete(@RequestParam(required = false) Integer id) {

        System.out.println("根据ID删除部门数据，id=" + id);
        return Result.success();
    }
```

### 新增部门

**Controller：DeptController**

```java
@PostMapping("/depts")
public Result add(@RequestBody Dept dept) {
    System.out.println("添加部门： " + dept);
    deptServiceimpl.add(dept);
    return Result.success();
}
```

**Service 实现：DeptServiceImpl**

```java
public void add(Dept dept) {
    // 创建时间、更新时间由后端统一维护，不信任前端传值
    dept.setCreateTime(LocalDateTime.now());
    dept.setUpdateTime(LocalDateTime.now());
    deptMapper.insert(dept);
}
```

**Mapper：DeptMapper**

```java
@Insert("insert into dept(name, create_time, update_time) values(#{name}, #{createTime}, #{updateTime})")
public void insert(Dept dept);
```

### 修改部门

#### 查询回显

路径参数：通过请求URL直接传递参数，使用{…}来标识该路径参数，需要使用 **`@PathVariable`**获取路径参数。

**Controller：DeptController**

```java
@GetMapping("/depts/{id}")
    public Result getInfo(@PathVariable Integer id) {
        System.out.println("根据ID查询部门数据： " + id);
        Dept dept = deptServiceImpl.getById(id);
        return Result.success(dept);
    }
```

**Service 实现：DeptServiceImpl**

```java
 public Dept getById(Integer id) {
        return deptMapper.getInfo(id);
    }
```

**Mapper：DeptMapper**

```java
@Select("select * from dept where id = #{id}")
    public Dept getInfo(Integer id);
```

#### 修改数据

**Controller：DeptController**

```java
@PutMapping
    public Result update(@RequestBody Dept dept) {
        System.out.println("修改部门： " + dept);
        deptServiceImpl.update(dept);
        return Result.success();
    }
```

**Service 实现：DeptServiceImpl**

```java
public void update(Dept dept) {
        dept.setUpdateTime(LocalDateTime.now());
        deptMapper.update(dept);
    }
```

**Mapper：DeptMapper**

```java
@Update("update dept set name = #{name}, update_time = #{updateTime} where id = #{id}")
    public void update(Dept dept);
```

### 日志技术

**采用sout的形式**

- 只能输出到控制台
- 不便于扩展、维护

程序中的**日志**，是用来记录应用程序的运行信息、状态信息、错误信息等

**日志的好处**

- 便于进行数据追踪
- 方便进行性能优化
- 通过所记录的错误信息进行问题排查
- 便于进行系统监控

**Logback**:基于**Log4j**升级而来，提供了更多的功能和配置选项，性能优于**Log4j**

#### LogBack快速入门

- 准备工作：引入logback的依赖**(springboot项目中该依赖已传递)**、配置文件logback.xml。
- 记录日志：定义日志记录对象Logger，记录日志。

```java
package com.cyy.tliaswebmanagement;

import org.junit.jupiter.api.Test;
import org.slf4j.LoggerFactory;

import org.slf4j.Logger;

public class LogTest {
    private static final Logger log = LoggerFactory.getLogger(LogTest.class);
    @Test
    public void testLog(){
        log.debug("开始计算...");
        int sum = 0;
        int[] nums = {1, 5, 3, 2, 1, 4, 5, 4, 6, 7, 4, 34, 2, 23};
        for (int num : nums) {
            sum += num;
        }
        log.info("计算结果为: "+sum);
        log.debug("结束计算...");
    }

}
```

```cmd
2026-08-24 16:40:46.510 [main] DEBUG com.cyy.tliaswebmanagement.LogTest-开始计算...
2026-08-24 16:40:46.512 [main] INFO  com.cyy.tliaswebmanagement.LogTest-计算结果为: 101
2026-08-24 16:40:46.512 [main] DEBUG com.cyy.tliaswebmanagement.LogTest-结束计算...
```

#### LogBack配置文件详解

- 配置文件名：logback.xml
- 该配置文件是对Logback日志框架输出的日志进行控制的，可以来配置输出的格式、位置及日志开关等。
- 常用的两种输出日志的位置：控制台、系统文件

```xml
<!-- 控制台输出 -->
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">...</appender>
<!-- 系统文件输出 -->
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">...</appender>
```

- 开启日志（ALL），关闭日志（OFF）

- ```xml
  <root level="ALL">
   <appender-ref ref="STDOUT" />
   <appender-ref ref="FILE" />
  </root>
  ```

| 块              | 标签                  | 作用                                           | 常改什么                            |
| --------------- | --------------------- | ---------------------------------------------- | ----------------------------------- |
| Appender        | `<appender>`          | “日志往哪里写”（控制台、文件、网络、数据库…）  | 输出目标、编码、滚动策略            |
| Encoder/Pattern | `<encoder><pattern>`  | “日志长什么样”（时间、线程、级别、类名、消息） | 格式字符串                          |
| Logger/Root     | `<root>` / `<logger>` | “哪些日志、什么级别、挂哪个 Appender”          | 级别、additivity、引用哪个 appender |

| 符号                          | 含义                                 | 例                                   |
| ----------------------------- | ------------------------------------ | ------------------------------------ |
| `%d{yyyy-MM-dd HH:mm:ss.SSS}` | 时间（可自定义格式）                 | `2026-08-24 14:30:45.123`            |
| `%thread`                     | 线程名                               | `http-nio-8080-exec-1`               |
| `%-5level`                    | 级别左对齐 5 宽                      | `DEBUG`, `INFO `, `ERROR`            |
| `%logger{50}`                 | 类名，最长 50 字符（超出截断包名）   | `c.c.t.service.impl.DeptServiceImpl` |
| `%msg` / `%message`           | 日志消息                             | `查询全部部门数据`                   |
| `%n`                          | 换行符（跨平台）                     | `\n` 或 `\r\n`                       |
| `%file` / `%line` / `%method` | 源文件/行号/方法名（性能敏感，慎用） | `DeptServiceImpl.java:21`            |

#### LogBack日志级别

日志级别指的是日志信息的类型，日志都会分级别，常见的日志级别如下（级别由低到高）：

| 日志级别 | 说明                                                         | 记录方式           |
| -------- | ------------------------------------------------------------ | ------------------ |
| trace    | 追踪，记录程序运行轨迹 【使用很少】                          | `log.trace("...")` |
| debug    | 调试，记录程序调试过程中的信息，实际应用中一般将其视为最低级别 【使用较多】 | `log.debug("...")` |
| info     | 记录一般信息，描述程序运行的关键事件，如：网络连接、io操作 【使用较多】 | `log.info("...")`  |
| warn     | 警告信息，记录潜在有害的情况 【使用较多】                    | `log.warn("...")`  |
| error    | 错误信息 【使用较多】                                        | `log.error("...")` |

可以在配置文件中，灵活的控制输出哪些类型的日志。(大于等于配置的日志级别的日志才会输出)

## 结果封装

- **手动结果映射**：通过 `@Results` 及 `@Result` 进行手动结果映射。

 ```java
  // DeptMapper
  @Results({
   @Result(column = "create_time", property = "createTime"),
   @Result(column = "update_time", property = "updateTime")
  })
  @Select("select id, name, create_time, update_time from dept order by update_time desc")
  public List<Dept> findAll();
 ```

- **起别名**：在SQL语句中，对不一样的列名起别名，别名和实体类属性名一样。

```java
// DeptMapper
@Select("select id, name, create_time createTime, update_time updateTime from dept ...")
public List<Dept> findAll();
```

- **开启驼峰命名开关**（要求:xxx_abc  -> xxxAbc）

```yaml
map-underscore-to-camel-case: true
```

## 反向代理

反向代理是一种网络架构技术，通过反向代理服务器为后端服务器做代理（安全、灵活、负载均衡）

### Nginx中反向代理的配置

<img width="531" height="351" alt="Image" src="https://github.com/user-attachments/assets/06961555-2eae-4488-a296-68bae856a10a" />

## 
