# JDBC

## 概述

**JDBC**：(Java DataBase Connectivity)，就是使用Java语言操作关系型数据库的一套API。

**本质**：

- sun公司官方定义的一套操作所有关系型数据库的规范，即接口。
- 各个数据库厂商去实现这套接口，提供数据库驱动jar包。
- 我们可以使用这套接口(JDBC)编程，真正执行的代码是驱动jar包中的实现类。

## 查询数据

-  ResultSet（结果集对象）：ResultSet rs = statement.executeQuery() 

   -  next()：将光标从当前位置向前移动一行，并判断当前行是否为有效行，返回值为boolean。 

      - true：有效行，当前行有数据 

      - false：无效行，当前行没有数据 

   -  getXxx(...)：获取数据，可以根据列的编号获取，也可以根据列名获取（推荐）。

**Employee类**

```java
package JDBC;

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

**DML语句**

```java
import JDBC.Employee;
import org.junit.Test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.time.LocalDate;
import java.time.LocalDateTime;

public class JdbcTest
{
    @Test
    public void testUpdate() throws ClassNotFoundException, SQLException
    {
        // 1、注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

        // 2、获取数据库连接
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db01", "root", "Cyy20040211,.");

        // 3、获取SQL语句执行对象
        Statement statement = conn.createStatement();

        // 4、执行SQL语句
        int i = statement.executeUpdate("update employee set job = 1 where id = 1");// DML
        System.out.println("受影响的行数：" + i);
        // 5、释放资源
        statement.close();
        conn.close();

    }

    
```

**DQL语句**

```mysql
@Test
    public void testSelect() throws ClassNotFoundException, SQLException
    {
        // 1、注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

        // 2、获取数据库连接
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db01", "root", "Cyy20040211,.");

        // 3、获取SQL语句执行对象
        Statement statement = conn.createStatement();

        // 4、执行SQL语句
        ResultSet rset = statement.executeQuery("select * from employee where job = 2");// DQL

        // 5、处理结果集，封装到Employee对象
        Employee employee = new Employee();
        while (rset.next())
        {
            employee.setId(rset.getInt("id"));
            employee.setUsername(rset.getString("username"));
            employee.setPassword(rset.getString("password"));
            employee.setName(rset.getString("name"));
            employee.setGender(rset.getInt("gender"));
            employee.setPhone(rset.getString("phone"));
            employee.setJob(rset.getInt("job"));
            employee.setSalary(rset.getInt("salary"));
            employee.setEntryDate(rset.getObject("entry_date", LocalDate.class));
            employee.setImage(rset.getString("image"));
            employee.setCreateTime(rset.getObject("create_time", LocalDateTime.class));
            employee.setUpdateTime(rset.getObject("update_time", LocalDateTime.class));

            System.out.println(employee);
        }


        // 6、释放资源
        rset.close();
        statement.close();
        conn.close();
    }
}
```



## 预编译SQL

**优势**

- 可以防止SQL注入，更安全
  - SQL注入：通过控制输入来修改实现定义好的SQL语句，以达到执行代码对服务器进行**攻击**的方法

- 性能更高
  - <img width="1160" height="258" alt="Image" src="https://github.com/user-attachments/assets/2235ea1a-b352-4756-8c4e-a354df3ae5bd" />

```java
@Test
public void testSelect1() throws ClassNotFoundException, SQLException
{
    // 1、注册驱动
    Class.forName("com.mysql.cj.jdbc.Driver");

    // 2、获取数据库连接
    Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db01", "root", "Cyy20040211,.");

    // 3、预编译SQL，使用?占位符
    PreparedStatement pstmt = conn.prepareStatement("select * from employee where id = ? and job = ?");

    // 4、设置占位符参数
    pstmt.setInt(1, 2);
    pstmt.setInt(2, 1);

    // 5、执行SQL语句
    ResultSet rset = pstmt.executeQuery();

    // 6、处理结果集，封装到Employee对象
    while (rset.next())
    {
        Employee employee = new Employee();
        employee.setId(rset.getInt("id"));
        employee.setUsername(rset.getString("username"));
        employee.setPassword(rset.getString("password"));
        employee.setName(rset.getString("name"));
        employee.setGender(rset.getInt("gender"));
        employee.setPhone(rset.getString("phone"));
        employee.setJob(rset.getInt("job"));
        employee.setSalary(rset.getInt("salary"));
        employee.setEntryDate(rset.getObject("entry_date", LocalDate.class));
        employee.setImage(rset.getString("image"));
        employee.setCreateTime(rset.getObject("create_time", LocalDateTime.class));
        employee.setUpdateTime(rset.getObject("update_time", LocalDateTime.class));

        System.out.println(employee);
    }

    // 7、释放资源
    rset.close();
    pstmt.close();
    conn.close();
}
```