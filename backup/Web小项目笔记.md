# 三层架构小项目学习笔记（Spring Boot + user.txt + /list 表格）

## 一、需求

从一个名为 `user.txt` 的文本文件中读取数据，通过 `localhost:8080/list` 以**表格形式**展示。
数据按行分隔，每行从左到右依次是：`ID 姓名 年龄`，中间用空格分隔。

```
1 张三 20
2 李四 25
3 王五 30
```

要求以**三层架构**实现：Dao 层负责数据读取，Service 调用 Dao 并把数据封装成 `List` 返回，Controller 接收 `List` 并响应数据。

---

## 二、最终实现思路（三层架构设计）

| 层 | 类 / 位置 | 职责 | 关键技术点 |
|---|---|---|---|
| Entity（实体） | `Entity/User.java` | 表示一条用户数据（id/name/age） | 纯数据类，无注解 |
| Dao（数据访问层） | `Dao/UserDao.java` + `Dao/impl/UserDaoimpl.java` | 只负责读取 user.txt，返回**原始行** `List<String>` | `@Repository` |
| Service（业务层） | `Service/UserService.java` + `Service/impl/UserServiceimpl.java` | 调用 Dao，把每行解析成 User，**封装进 `List<User>`** 返回 | `@Service` |
| Controller（表现层） | `Controller/UserController.java` | 接收 `List<User>`，拼成 HTML 表格字符串响应 | `@RestController`、`@RequestMapping` |
| 启动类 | `Demo/Application.java` | 启动入口 + 组件扫描 | `@SpringBootApplication` |

数据流：

```
user.txt ──读→ Dao(readAllLines: List<String>)
             └─被调用→ Service(findAll: List<User>)
                          └─被调用→ Controller(list: 输出HTML表格)
                                     └─响应→ 浏览器 localhost:8080/list
```

---

## 三、最终实现过程

### 1. 创建实体类 `Entity/User.java`

```java
package Entity;

public class User
{
    private Integer id;
    private String name;
    private Integer age;

    public User() { }
    public User(Integer id, String name, Integer age)
    {
        this.id = id;
        this.name = name;
        this.age = age;
    }
    // getter / setter ...
}
```

> 用 `Integer` 而非 `int`：实体要放进 `List<User>`（泛型不支持基本类型），且 `Integer` 可为 `null` 表示"无值"。

### 2. Dao 层

**接口 `Dao/UserDao.java`**：

```java
package Dao;

import java.util.List;

public interface UserDao
{
    List<String> readAllLines();   // 只负责数据读取，返回原始行
}
```

**实现类 `Dao/impl/UserDaoimpl.java`**：

```java
package Dao.impl;

import Dao.UserDao;
import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Repository;
import java.io.*; import java.nio.charset.StandardCharsets; import java.util.*;

@Repository
public class UserDaoimpl implements UserDao
{
    @Override
    public List<String> readAllLines()
    {
        List<String> lines = new ArrayList<>();
        try (InputStream in = new ClassPathResource("user.txt").getInputStream();
             BufferedReader reader =
                 new BufferedReader(new InputStreamReader(in, StandardCharsets.UTF_8)))
        {
            String line;
            while ((line = reader.readLine()) != null)
            {
                line = line.trim();
                if (!line.isEmpty())
                {
                    lines.add(line);
                }
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        return lines;
    }
}
```

读取代码逐层理解（从右往左）：

```
文件 user.txt
   → InputStream           字节流（读原始字节）
   → InputStreamReader     字符流（用 UTF-8 把字节解码成文字，防止中文乱码）
   → BufferedReader        加缓冲，获得 readLine() 一次读一行
```

`try (…) {}` 是 **try-with-resources 自动资源管理**：括号内声明的流用完自动调用 `close()`，不用手写 finally。

### 3. Service 层

**接口 `Service/UserService.java`**：

```java
package Service;

import Entity.User;
import java.util.List;

public interface UserService
{
    List<User> findAll();   // 返回封装好的用户集合
}
```

**实现类 `Service/impl/UserServiceimpl.java`**：

```java
package Service.impl;

import Dao.UserDao;
import Entity.User;
import Service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.*;

@Service
public class UserServiceimpl implements UserService
{
    @Autowired
    private UserDao userDao;   // 依赖注入：调用 Dao 层

    @Override
    public List<User> findAll()
    {
        List<User> users = new ArrayList<>();
        for (String line : userDao.readAllLines())   // 调用 Dao 拿原始数据
        {
            String[] parts = line.split(" ");        // "1 张三 20" → ["1","张三","20"]
            Integer id  = Integer.parseInt(parts[0]); // 字符串转数字
            String name = parts[1];
            Integer age = Integer.parseInt(parts[2]);
            users.add(new User(id, name, age));       // 封装进 List 并返回
        }
        return users;
    }
}
```

> `Integer.parseInt("20")`：把数字字符串转成 `int`（静态方法），字符串不含数字会抛 `NumberFormatException`。

### 4. Controller 层

```java
package Controller;

import Entity.User;
import Service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.*;

@RestController
public class UserController
{
    @Autowired
    private UserService userService;   // 依赖注入：调用 Service 层

    @RequestMapping(value = "/list", produces = "text/html;charset=UTF-8")
    public String list()
    {
        List<User> users = userService.findAll();   // 接收 Service 返回的 List
        // …… 遍历 users 拼成 HTML <table> 字符串并 return
    }
}
```

`@RequestMapping("/list")`：路由映射注解，把 `/list` 这个 URL 绑定到该方法；`produces` 指定响应类型为 UTF-8 的 HTML（保证中文不乱码）。

### 5. 修改启动类 `Demo/Application.java`

原 `@SpringBootApplication` 只会扫描 `Demo` 包，新增的 Controller/Service/Dao 资源到不到，必须指定扫描范围：

```java
@SpringBootApplication(scanBasePackages = {"Demo", "Controller", "Service", "Dao", "Entity"})
```

### 6. 补充 `pom.xml`

原 pom 缺 `spring-boot-maven-plugin`，导致 `mvn spring-boot:run` 无法启动，补上：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.7.18</version>
        </plugin>
    </plugins>
</build>
```

### 7. 填充 `resources/user.txt` 示例数据

```
1 张三 20
2 李四 25
3 王五 30
```

### 8. 验证

```bash
mvn compile            # 编译，无报错
mvn spring-boot:run    # 启动
# 浏览器访问 localhost:8080/list，看到带边框 HTML 表格，中文正常显示
```

---

## 四、项目中注解的作用与联动

### 注解总表

| 注解 | 位置 | 作用 |
|---|---|---|
| `@SpringBootApplication` | Application | 组合注解 = 配置 + 自动配置 + 组件扫描 |
| `@RestController` | UserController | 注册 Bean + 声明"返回数据的控制器" |
| `@Service` | UserServiceimpl | 注册 Bean + 声明"业务层" |
| `@Repository` | UserDaoimpl | 注册 Bean + 声明"数据访问层" |
| `@Autowired` | 字段 | 依赖注入：按类型从容器找到匹配 Bean 装进来 |
| `@RequestMapping` | 方法 | 把 URL 路径绑定到处理方法 |
| `@Override` | 方法 | 校验确实是接口方法的实现 |

### 三个流程

**① 启动扫描（找 Bean）**：`@SpringBootApplication` → 按 `scanBasePackages` 扫描各包，把带 `@RestController/@Service/@Repository`（都是 `@Component` 衍生）的类实例化（默认单例），存进 IoC 容器 `ApplicationContext`。

**② 依赖注入（链接 Bean）**：容器看到 `@Autowired` 字段，按类型匹配注入：
`UserController.userService ← UserServiceimpl`，`UserServiceimpl.userDao ← UserDaoimpl`。各层就此串联。

**③ 一次请求**：浏览器 `/list` → 匹配 `list()` 方法 → `userService.findAll()` → `userDao.readAllLines()` → 读文件 → Service 封装 `List<User>` → Controller 渲染表格返回。

> 注意：去掉组件注解，容器里就没有对应 Bean，启动会报 `NoSuchBeanDefinitionException`。

---

## 五、学习过程中提出的问题汇总

1. **`try (InputStream ... readLine())` 各方法作用**
   `ClassPathResource("user.txt")` 定位类路径文件 → `.getInputStream()` 得到字节流 → `InputStreamReader(in, UTF-8)` 转字符流（解决中文乱码）→ `BufferedReader` 提供 `readLine()` 逐行读取；try-with-resources 自动关闭流。

2. **`@RequestMapping` 作用**
   把 URL 路径绑定到处理方法；可配 `method`（请求方式）、`produces`（返回类型）等。

3. **Spring 如何获取 Javabean 并链接**
   组件扫描发现注解类 → 实例化放入 IoC 容器 → `@Autowired` 按类型注入完成链接。

4. **`@Repository` 作用**
   注册 Bean（`@Component` 衍生）+ 标识数据访问层 + 数据访问异常统一翻译。

5. **`@Autowired` 作用 / 项目联动**
   依赖注入，按类型把匹配的 Bean 注入字段；配合扫描与路由形成 Controller→Service→Dao 链路。

6. **`Integer` 与 `int` 区别**
   `int` 基本类型/默认 0/不可为 null/值运算；`Integer` 引用类型/默认 null/可装箱拆箱/有 `parseInt` 等方法/比较值用 `equals()`。泛型容器、可空数据用 `Integer`，纯运算用 `int`。

7. **`Integer.parseInt()` 作用**
   字符串 → int 的静态方法；参数必须为数字字符串，否则抛 `NumberFormatException`。

8. **`@Autowired private UserDao userDao` 与 `private UserDao userDao = new UserDaoimpl()` 区别**
   `@Autowired`：依赖由 **IoC 容器创建并注入**，应用内共享单例，只依赖接口 `UserDao`，换实现类不用改业务代码（解耦、易测试）；`new`：自己手动创建实例并**写死实现类**，每次 `new` 一个对象，换实现必须改代码（高耦合）。分层项目中应采用 `@Autowired`。

9. **单例 Bean 为什么不直接用静态方法**
   因为这些类不是纯工具类，而是**依赖关系网里的组件**：它们需要被注入其他依赖（如 `UserServiceimpl` 依赖 `UserDao`）；用静态方法只能内部 `new` 依赖、回到硬编码。且 Bean 支持接口多态替换、mock 测试、AOP 代理（`@Transactional` 等）、可配置与生命周期管理。真正的纯功能工具类（如 `Integer.parseInt`、`StringUtils`）才适合静态方法——判断标准：**带依赖、参与业务协作 → Bean；纯函数、无依赖 → 静态方法**。

10. **依赖注入（DI）是什么意思**
    让对象**不自己 new 依赖，而是声明"我需要什么"，由外部（Spring 容器）把依赖注入给它**。核心叫控制反转（IoC）：创建谁、给谁的权力从类本身反转到容器。好处是解耦、可替换、易测试——Service 层只依赖接口 `UserDao`，数据源从 txt 换成数据库只需新增实现类，Service 代码不用改。一句话："我不找依赖，等着依赖被送上门。"

11. **Markdown 链接引用语法**
    基本写法 `[文字](地址 "标题")`，`]` 与 `(` 之间不能有空格；标题用英文双引号包围（可省略）；本地路径含空格/中文时用 `<...>` 包裹。参考式：`[文字][id]` + 文末定义 `[id]: 地址 "标题"`，适合同一链接多处引用。

---

## 六、文件结构（忽略 Demo 示例控制器）

```
src/main/java
├── Controller/UserController.java     表现层：接收 List，渲染 HTML 表格
├── Service/UserService.java           业务层接口
├── Service/impl/UserServiceimpl.java  业务层实现：解析数据、封装 List
├── Dao/UserDao.java                   数据访问层接口
├── Dao/impl/UserDaoimpl.java          数据访问层实现：读取 user.txt
├── Entity/User.java                   实体类
└── Demo/Application.java              启动类（已调整扫描范围）

src/main/resources
└── user.txt                           数据文件：ID 姓名 年龄
```