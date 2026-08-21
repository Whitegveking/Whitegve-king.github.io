# MyBatis

## 概述

MyBatis 是一款优秀的**持久层**框架，用于**简化 JDBC** 的开发。

------

## 入门程序

### 1. 核心思想

| 对比项     | 原生 JDBC                          | MyBatis            |
| ---------- | ---------------------------------- | ------------------ |
| 加载驱动   | 每次 `Class.forName`               | 自动               |
| 获取连接   | 每次 `DriverManager.getConnection` | 连接池自动管理     |
| SQL 预编译 | 手动 `prepareStatement`            | 自动               |
| 结果封装   | 手动 `getInt/getString` 逐列赋值   | **自动映射到对象** |

MyBatis 核心作用：**方法调用 → 生成 SQL → 执行 → 结果自动封装**，开发者只需关注**接口方法 + SQL 语句**。

------

### 2. 环境搭建（本项目）

#### 2.1 引入依赖（pom.xml）

```
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>

<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

> 版本组合：Spring Boot 2.6.13 + mybatis-spring-boot-starter 2.2.2

#### 2.2 配置数据源（src/main/resources/application.properties）

```properties
# 指定Mybatis的Mapper文件
mybatis.mapper-locations=classpath:mappers/*xml
# 指定Mybatis的实体目录
mybatis.type-aliases-package=com.cyy.springbootmybatis.mybatis.entity
# 配置数据库的链接信息
spring.datasource.url=jdbc:mysql://localhost:3306/db01?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=你的密码
```

> `db01` 是数据库名。驱动加载由 starter 自动完成，不再需要 `Class.forName`。

------

### 3. 实体类（pojo 包）

对应数据库表 `employee`，使用 Lombok 简化开发：

```
package com.cyy.springbootmybatis.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDate;
import java.time.LocalDateTime;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Employee
{
    private Integer id;
    private String username;
    private String password;
    private String name;
    private Integer gender;
    private String phone;
    private Integer job;
    private Integer salary;
    private LocalDate entryDate;
    private String image;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

> 数据库列 `entry_date`、`create_time` 会自动映射为 `entryDate`、`createTime`（下划线转驼峰）。

------

### 4. Mapper 接口（mapper 包）

```
package com.cyy.springbootmybatis.mapper;

import com.cyy.springbootmybatis.pojo.Employee;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

import java.util.List;

@Mapper
public interface EmployeeMapper
{
    // 查询所有员工信息
    @Select("select * from employee")
    public List<Employee> findAll();
}
```

要点：

- `@Mapper`：运行时 MyBatis 自动创建**代理实现类对象**并存入 IOC 容器，无需手写实现类
- `@Select`：**注解形式**直接写 SQL 语句
- 返回值 = 查询结果（多条用 `List<Employee>`；单条用 `Employee`）

------

### 5. 启动类

```
package com.cyy.springbootmybatis;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootMybatisApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(SpringbootMybatisApplication.class, args);
    }
}
```

> `@SpringBootApplication` 启动时会自动扫描 `@Mapper` 接口并注册为 Bean。

------

### 6. 使用（单元测试）

```
package com.cyy.springbootmybatis;

import com.cyy.springbootmybatis.mapper.EmployeeMapper;
import com.cyy.springbootmybatis.pojo.Employee;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class SpringbootMybatisApplicationTests
{
    @Autowired
    private EmployeeMapper employeeMapper;

    @Test
    public void testFindAll()
    {
        List<Employee> emp = employeeMapper.findAll();
        emp.forEach(System.out::println);
    }
}
```

调用流程：

```
@Autowired 注入代理对象
    → employeeMapper.findAll()      调用方法
        → @Select 读取 SQL 语句
            → 执行 select * from employee
                → 结果封装成 List<Employee>
                    → 打印每个 Employee
```

------

### 7. 参数传递

带条件查询时，SQL 中的变量用 `#{}` 占位（预编译、防 SQL 注入）：

```
@Select("select * from employee where id = #{id}")
public Employee findById(Integer id);

@Select("select * from employee where job = #{job} and salary > #{salary}")
public List<Employee> findByCondition(Integer job, Integer salary);
```

- 单个参数：`#{任意名}` 均可（如 `#{id}`）
- 多个参数：方法参数名要和 `#{名}` 对应（如 `#{job}`、`#{salary}`）

------

### 8. 常用注解速查

| 注解      | 用途 | 示例                                                         |
| --------- | ---- | ------------------------------------------------------------ |
| `@Select` | 查询 | `@Select("select * from employee")`                          |
| `@Insert` | 新增 | `@Insert("insert into employee(name, job) values(#{name}, #{job})")` |
| `@Update` | 修改 | `@Update("update employee set name=#{name} where id=#{id}")` |
| `@Delete` | 删除 | `@Delete("delete from employee where id=#{id}")`             |

------

### 9. 注解方式 vs XML 方式

| 方式 | 适用场景           | 特点                                      |
| ---- | ------------------ | ----------------------------------------- |
| 注解 | 简单 SQL（本项目） | 代码简洁，SQL 写在 Java 里                |
| XML  | 复杂 SQL、动态 SQL | 解耦，SQL 集中管理，支持 `<if>/<foreach>` |

XML 方式需搭配 `application.properties` 中的 `mybatis.mapper-locations=classpath:mappers/*xml`，并在 `src/main/resources/mappers/` 下编写对应 XML 文件。

------

### 10. 小结：JDBC vs MyBatis

| 步骤     | 原生 JDBC                         | MyBatis             |
| -------- | --------------------------------- | ------------------- |
| 连接获取 | `DriverManager.getConnection`     | IOC 容器/连接池自动 |
| SQL 执行 | `Statement` / `PreparedStatement` | Mapper 接口方法     |
| 参数设置 | `setInt/setString...`             | `#{参数}` 自动设置  |
| 结果封装 | 手动 `getInt/getString` 逐列赋值  | 自动映射成对象      |
| 资源释放 | 手动 `close()`                    | 自动管理            |

## 数据库连接池

### 概念

- **数据库连接池**是个容器，负责分配、管理数据库连接(Connection)
- 它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个。
- 释放空闲时间超过最大空闲时间的连接，来避免因为没有释放连接而引起的数据库连接遗漏。
- 优势
  1. 资源重用
  2. 提升系统响应速度
  3. 避免数据库连接遗漏



## 基于MyBatis的增删改查操作

### 删除用户-Delete

```java
@Delete("delete from employee where id = #{id}")
    public int deleteById(Integer id);
```

```java
 @Test
    public void testDeleteById()
    {
        // 返回值是受影响的行数
        int i = employeeMapper.deleteById(5);
        System.out.println("受影响的行数: " + i);
    }
```

| 符号     | 说明                                                     | 场景                       | 优缺点               |
| -------- | -------------------------------------------------------- | -------------------------- | -------------------- |
| `#{...}` | 占位符。执行时，会将 `#{...}` 替换为 `?`，生成预编译 SQL | 参数值传递                 | 安全、性能高（推荐） |
| `${...}` | 拼接符。直接将参数拼接在 SQL 语句中，存在 SQL 注入问题   | 表名、字段名动态设置时使用 | 不安全、性能低       |

### 新增用户-insert

```java
@Insert("insert into employee (id, username, password, name, gender, phone, job, salary, entry_date, image, create_time, update_time) " +
        "values (#{id}, #{username}, #{password}, #{name}, #{gender}, #{phone}, #{job}, #{salary}, #{entryDate}, #{image}, #{createTime}, #{updateTime})")
public void insert(Employee employee);
```

```java
@Test
    public void testInsert()
    {
        Employee employee = new Employee();
        employee.setId(5);
        employee.setUsername("zhangsan");
        employee.setPassword("abc123");
        employee.setName("张三");
        employee.setGender(1); // Integer
        employee.setPhone("13800138005");
        employee.setJob(2); // Integer
        employee.setSalary(8000); // Integer
        employee.setEntryDate(LocalDate.of(2022, 3, 1)); // LocalDate
        employee.setImage(null);
        employee.setCreateTime(LocalDateTime.now()); // LocalDateTime
        employee.setUpdateTime(LocalDateTime.now());


        employeeMapper.insert(employee);
    }
```

### 更新操作-update

```java
@Update("update employee set username = #{username},name = #{name} where id = #{id}")
    public void update(Employee employee);
```

```java
@Test
    public void testUpdate()
    {
        Employee employee = new Employee();
        employee.setId(5);
        employee.setUsername("zhouyu");
        employee.setPassword("def456");
        employee.setName("周瑜");
        employee.setGender(0); // Integer
        employee.setPhone("13800138006");
        employee.setJob(1); // Integer
        employee.setSalary(9000); // Integer
        employee.setEntryDate(LocalDate.of(2022, 4, 1)); // LocalDate
        employee.setImage(null);
        employee.setCreateTime(LocalDateTime.now()); // LocalDateTime
        employee.setUpdateTime(LocalDateTime.now());

        employeeMapper.update(employee);
    }
```



### 查询操作-select

```java
@Select("select * from employee where id = #{id}")
    public Employee findById(Integer id);

@Select("select * from employee where username = #{username} and password = #{password}")
    public Employee findByUsernameAndPassword(@Param("username") String username, @Param("password") String password);
```

```java
@Test
public void testFindById()
{
    Employee employee = employeeMapper.findById(5);
    System.out.println(employee);
}

@Test
public void testFindByUsernameAndPassword()
{
   	Employee employee = employeeMapper.findByUsernameAndPassword("zhouyu", "abc123");
    System.out.println(employee);
}
```

#### 对应注解

| 注解      | 用途           | SQL 位置         | 返回值                                  |
| --------- | -------------- | ---------------- | --------------------------------------- |
| `@Select` | 查询（DQL）    | 写在注解的参数里 | 单条 `Employee` / 多条 `List<Employee>` |
| `@Param`  | 给多个参数命名 | 标注在方法参数上 | —                                       |

#### 注意事项

1. **参数占位用 `#{...}`，不要用 `${...}`**

   - `#{id}` → 预编译占位符 `?`，安全、防 SQL 注入（推荐）
   - `${id}` → 字符串拼接，有 SQL 注入风险，仅用于表名/字段名等动态场景

2. **单个参数**：`#{任意名}` 均可，不需要 `@Param`

   ```java
   @Select("select * from employee where id = #{id}")
   public Employee findById(Integer id);
   ```

3. **多个参数必须加 `@Param`**，否则 MyBatis 认不出 `#{username}`

   ```java
   @Select("select * from employee where username = #{username} and password = #{password}")
   public Employee findByUsernameAndPassword(@Param("username") String username, @Param("password") String password);
   ```

   > [!NOTE]
   >
   > 基于官方骨架创建的 springboot 项目中，接口编译时会保留方法形参名，**@Param** 注解可以省略**（#{形参名}）**。

   

4. **返回值类型的选取**

   - 查询结果可能是多条 → 返回 `List<Employee>`
   - 查询结果最多一条 → 返回 `Employee`
   - 注意：返回 `Employee` 但查到多条会抛 `TooManyResultsException`

5. **下划线列名转驼峰属性**

   - `entry_date` → `entryDate`，`create_time` → `createTime`

   - 默认未开启转换，需要配置，否则日期字段查询出来是`null`：`mybatis.configuration.map-underscore-to-camel-case=true`

     

6. **null 字段的处理**

   - 查询结果中 `null`（如 `image`、`entry_date` 为空）会原样封装为 Java `null`，不会报错
   - 打印结果中日期为 `null` 时，优先检查上一条驼峰映射配置

7. **调用与测试**

   - 使用 `@Autowired` 注入 `EmployeeMapper`（IOC 容器中的代理对象）
   - 单元测试类加 `@SpringBootTest` 才会启动 Spring 容器
   - 获取单条：`Employee e = employeeMapper.findById(5); System.out.println(e);`
   - 获取多条：`emp.forEach(System.out::println);` 或 `emp.forEach(e -> System.out.println(e))`

### XML映射配置

- 在Mybatis中，既可以通过注解配置SQL语句，也可以通过XML配置文件配置SQL语句。
- 默认规则：
  1. XML映射文件的名称与Mapper接口名称一致，并且将XML映射文件和Mapper接口放置在相同包下（同包同名）。
  2. XML映射文件的namespace属性为Mapper接口全限定名一致。
  3. XML映射文件中sql语句的id与Mapper接口中的方法名一致，并保持返回类型一致。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- mapper：根标签，namespace 必须写成对应 Mapper 接口的全限定名（包名.类名），
     MyBatis 靠它把 XML 里的 SQL 和接口方法绑定起来 -->
<mapper namespace="com.cyy.springbootmybatis.mapper.EmployeeMapper">

    <!-- resultMap：定义"数据库列 -> Java实体类属性"的映射关系。
         因为开启了 map-underscore-to-camel-case（下划线转驼峰），
         其实大多数字段不用手动配，这里主要为了演示 <id> 和 <result> 用法。
         id     : resultMap 的唯一标识，被下面各个 select 通过 resultMap="employeeMap" 引用
         type   : 映射到的结果类型（实体类的全限定名） -->
    <resultMap id="employeeMap" type="com.cyy.springbootmybatis.pojo.Employee">
        <!-- id：主键列映射（在 resultMap 里必须放最前面，表示唯一标识） -->
        <id property="id" column="id"/>
        <!-- result：普通列映射。property=实体类属性名，column=数据库列名。
             例如 entry_date 列 -> entryDate 属性，create_time -> createTime -->
        <result property="username" column="username"/>
        <result property="password" column="password"/>
        <result property="name" column="name"/>
        <result property="gender" column="gender"/>
        <result property="phone" column="phone"/>
        <result property="job" column="job"/>
        <result property="salary" column="salary"/>
        <result property="entryDate" column="entry_date"/>
        <result property="image" column="image"/>
        <result property="createTime" column="create_time"/>
        <result property="updateTime" column="update_time"/>
    </resultMap>

    <!-- select：查询语句标签。id 必须和接口方法名一致（如 EmployeeMapper.findAll()）。
         resultMap 指定结果映射，返回 List<Employee>，每条记录封装成一个 Employee -->
    <select id="findAll" resultMap="employeeMap">
        select * from employee
    </select>

    <!-- delete：删除语句标签，对应接口 deleteById(Integer id)。
         #{id} 是参数占位符（预编译，防SQL注入），参数值来自方法入参 -->
    <delete id="deleteById">
        delete from employee where id = #{id}
    </delete>

    <!-- insert：插入语句标签，对应接口 insert(Employee employee)。
         方法入参是实体对象，#{属性名} 会自动取出对象里对应字段的值，
         比如 #{username} 取 employee.getUsername() -->
    <insert id="insert">
        insert into employee (id, username, password, name, gender, phone, job, salary, entry_date, image, create_time, update_time)
        values (#{id}, #{username}, #{password}, #{name}, #{gender}, #{phone}, #{job}, #{salary}, #{entryDate}, #{image}, #{createTime}, #{updateTime})
    </insert>

    <!-- update：修改语句标签，对应接口 update(Employee employee)。
         根据 id 更新 username 和 name 两个字段 -->
    <update id="update">
        update employee set username = #{username}, name = #{name} where id = #{id}
    </update>

    <!-- 按主键查单个员工，对应接口 findById(Integer id)，返回一个 Employee 对象 -->
    <select id="findById" resultMap="employeeMap">
        select * from employee where id = #{id}
    </select>

    <!-- 按用户名和密码查员工（登录校验），对应接口 findByUsernameAndPassword(username, password)。
         #{} 里的名字要和接口方法上的 @Param 注解一致 -->
    <select id="findByUsernameAndPassword" resultMap="employeeMap">
        select * from employee where username = #{username} and password = #{password}
    </select>

</mapper>

```

### XML映射文件-辅助配置

使用 XML 方式时，`application.properties`（或 `application.yml`）中需要配套的配置，让 MyBatis 能**找到 XML 文件**、**识别实体别名**、**开启自动映射和 SQL 日志**：

```properties
# 指定Mybatis的Mapper文件（XML映射文件存放位置，注意与本项目实际目录一致）
mybatis.mapper-locations=classpath:com/cyy/mapper/*.xml
# 指定Mybatis的实体目录（为实体类起别名，XML的resultType可简写为类名）
mybatis.type-aliases-package=com.cyy.springbootmybatis.pojo
# 配置日志输出（把执行的SQL和参数打印到控制台，方便调试）
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# 配置下划线转驼峰（entry_date -> entryDate，不开启则日期等字段查出来是null）
mybatis.configuration.map-underscore-to-camel-case=true
```

对应的 `application.yml` 写法：

```yaml
mybatis:
  mapper-locations: classpath:com/cyy/mapper/*.xml
  type-aliases-package: com.cyy.springbootmybatis.pojo
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true
```

各项说明：

| 配置项                         | 作用                                   | 注意事项                                                     |
| ------------------------------ | -------------------------------------- | ------------------------------------------------------------ |
| `mapper-locations`             | 指定 XML 映射文件的路径                | 路径必须和 XML 实际存放位置一致，否则报 `Invalid bound statement` |
| `type-aliases-package`         | 实体类包名，把类名注册为别名           | 配了之后 `resultType` 可写成 `Employee`，否则要写全限定名    |
| `log-impl`                     | 打印 SQL 及参数到控制台                | 调式利器，生产环境建议关闭                                   |
| `map-underscore-to-camel-case` | 自动把 `entry_date` 映射为 `entryDate` | 不开启时下划线字段映射不到属性，值为 `null`                  |

> [!IMPORTANT]
> 默认规则：XML 文件与 Mapper 接口**同名同包**，`namespace` 写接口全限定名，SQL 的 `id` 写方法名。

> [!WARNING]
>
> 一个方法**不能同时**用注解和 XML 定义 SQL。迁移到 XML 后，接口上要删掉 `@Select/@Insert/@Update/@Delete` 注解，否则启动报错： `Mapped Statements collection already contains value for ...EmployeeMapper.findAll`