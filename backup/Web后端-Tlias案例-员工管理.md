# Web后端-Tlias案例-员工管理

## 多表关系

### 一对多（多对一）

**场景**：部门与员工的关系（一个部门下有多少个员工）

<img width="1194" height="369" alt="Image" src="https://github.com/user-attachments/assets/d45f3f01-ef55-4f1f-bdce-92145daafdc9" />

**实现**：在数据库表中多的一方，添加字段，来关联一的一方的**主键**

#### 多表问题分析

- **现象**:部门数据可以直接删除，然而还有部分员工归属于该部门下，此时就出现了数据的不完整、不一致问题
- **原因**:目前上述的两张表，在数据库层面，并未建立关联，所以是无法保证数据的一致性和完整性的
- **解决方案**：外键约束

```mysql
-- 创建表时指定
create table 表名(
 字段名 数据类型,
 ...
 [constraint] [外键名称] foreign key (外键字段名) references 主表 (字段名)
);
-- 建完表后，添加外键
alter table 表名 add constraint 外键名称 foreign key (外键字段名) references 主表(字段名);
```



#### 外键约束

**物理外键**

**概念**：使用`foreign key`定义外键关联另一张表

**缺点**：

1. 影响增、删、改的效率(需要检查外键关系)
2. 仅用于单节点数据库，不适用与分布式、集群场景
3. 容易引发数据库的死锁问题，消耗性能

```mysql
alter table emp add constraint fk_emo_dept_id foreign key (dept_id) references dept(id);
```



**逻辑外键**

**概念**：在业务层逻辑中，解决外键关联

通过逻辑外键就可以方便地解决物理外键的问题

### 一对一

**案例**:用户 与身份证信息的关系

**关系**:一对一关系，多用于单表拆分，将一张表的基础字段放在一张表中，其他字段放在另一张表中，以提升操作效率

**实现**:在任意一方加入外键，关联另外一方的主键，并且设置外键为唯一的(UNIQUE)

```mysql
create table tb_user_card(
                             id int unsigned  primary key auto_increment comment 'ID',
                             nationality varchar(10) not null comment '民族',
                             birthday date not null comment '生日',
                             idcard char(18) not null comment '身份证号',
                             issued varchar(20) not null comment '签发机关',
                             expire_begin date not null comment '有效期限-开始',
                             expire_end date comment '有效期限-结束',
                             user_id int unsigned not null unique comment '用户ID',
                             constraint fk_user_id foreign key (user_id) references tb_user(id)
```



### 多对多

**案例**:学生与课程的关系

**关系**:一个学生可以选修多门课程，一门课程也可以供多个学生选择

**实现**:建立第三张中间表，中间表至少包含两个外键，分别关联两方主键

### 案例

**需求**

- 根据参考资料中提供的《Talis智能学习辅助系统》页面原型，设计员工管理模块涉及到的表结构

**步骤**

- 阅读页面原型及需求文档，分析各个模块涉及到的表结构，及表结构之间的关系
- 根据页面原型及需求文档，分析各个表结构中具体的字段及约束

**分析**

涉及到的模块有：

- **部门管理**

![img](./Web%E5%90%8E%E7%AB%AF-Tlias%E6%A1%88%E4%BE%8B-%E5%91%98%E5%B7%A5%E7%AE%A1%E7%90%86.assets/1787624647353-3.png)

部门管理涉及到一张部门表，这个前面我们都已经设计过了 无需再进行设计了

- **员工管理**

![img](./Web%E5%90%8E%E7%AB%AF-Tlias%E6%A1%88%E4%BE%8B-%E5%91%98%E5%B7%A5%E7%AE%A1%E7%90%86.assets/1787624647353-1.png)

上述在员工列表查询的页面原型，当我们点击 "新增员工" 按钮时，会弹出一个新增员工的表单，表单展示形式如下：

![img](./Web%E5%90%8E%E7%AB%AF-Tlias%E6%A1%88%E4%BE%8B-%E5%91%98%E5%B7%A5%E7%AE%A1%E7%90%86.assets/1787624647353-2.png)

在上述的页面原型中，我们可以看到，每一个员工是归属于某一个部门的，而一个部门下可以有多个员工，所以部门与员工之间的关系是一对多的关系

从页面员工中，我们可以看到，员工还有工作经历的信息而每一个员工，是可以添加多个工作经历的 所以，工作经历我们可以再设计一张表，而员工与员工的工作经历之间的关系，是一对多的关系

那么，这里总共会涉及三张表，分别是：部门表、员工表、员工工作经历表

```mysql
-- 部门表
create table dept (
  id int unsigned PRIMARY KEY AUTO_INCREMENT COMMENT 'ID, 主键',
  name varchar(10) NOT NULL UNIQUE COMMENT '部门名称',
  create_time datetime COMMENT '创建时间',
  update_time datetime COMMENT '修改时间'
) COMMENT '部门表';

-- 员工表
create table emp(
    id int unsigned primary key auto_increment comment 'ID,主键',
    username varchar(20) not null unique comment '用户名',
    password varchar(50) default '123456' comment '密码',
    name varchar(10) not null comment '姓名',
    gender tinyint unsigned not null comment '性别, 1:男, 2:女',
    phone char(11) not null unique comment '手机号',
    job tinyint unsigned comment '职位, 1 班主任, 2 讲师 , 3 学工主管, 4 教研主管, 5 咨询师',
    salary int unsigned comment '薪资',
    image varchar(300) comment '头像',
    entry_date date comment '入职日期',
    dept_id int unsigned comment '部门ID',  -- 关联的是dept部门表的ID
    create_time datetime comment '创建时间',
    update_time datetime comment '修改时间'
) comment '员工表';

-- 员工工作经历表
create table emp_expr(
    id int unsigned primary key auto_increment comment 'ID, 主键',
    emp_id  int unsigned null comment '员工ID', -- 关联的是emp员工表的ID
    begin  date null comment '开始时间',
    end date null comment '结束时间',
    company varchar(50) null comment '公司名称',
    job varchar(50) null comment '职位'
) comment '工作经历';
```

## 多表查询

查询时从多张表中获取所需数据

单表查询的SQL语句：select  字段列表  from  表名;

那么要执行多表查询，只需要使用逗号分隔多张表即可，如： select   字段列表  from  表1, 表2;而这会导致表1和表2的所有字段进行组合，会出现表一记录数×表二记录数条记录，这种现象称之为**笛卡尔积**

### 内连接

内连接查询的是两张表交集部分的数据

```mysql
-- 1.隐式内连接
select 字段列表from 表1，表2 where 连接条件...;
-- 2.显式内连接
select 字段列表from 表1 [inner] join 表2 on 连接条件...
```

> [!CAUTION]
>
> 一旦为表指定了别名，就要通过别名来指定字段名，而不能再使用表名了

### 左外连接

```mysql
-- 1.左外连接
select 字段列表 from 表1 left [outer] join 表2 on 连接条件 ...;
```

左外连接相当于查询表1(左表)的所有数据，当然也包含表1和表2交集部分的数据

### 右外连接

```mysql
-- 2.右外连接
select 字段列表 from 表1 right [outer] join 表2 on 连接条件 ...;
```

右外连接相当于查询表2(右表)的所有数据，当然也包含表1和表2交集部分的数据

> [!CAUTION]
>
> 左外连接和右外连接是可以相互替换的，只需要调整连接查询时SQL语句中表的先后顺序就可以了。而日常开发使用时，更偏向于左外连接。

### 子查询

- 介绍：SQL语句中嵌套select语句，称为嵌套查询，又称子查询

- 形式：`select * from t1 where column1 = (select column1 from t2 ...);`

- 说明：子查询外部的语句可以是 insert / update / delete / select 的任何一个，最常见的是 select

- 分类：

  1. 标量子查询：子查询返回的结果为单个值
  2. 列子查询：子查询返回的结果为一列
  3. 行子查询：子查询返回的结果为一行
  4. 表子查询：子查询返回的结果为多行多列

  > [!NOTE]
  >
  > 子查询要先对需求做拆分，明确具体的步骤，再逐步编写SQL语句

  

#### 标量子查询

子查询返回的结果是单个值(数字、字符串、日期等)，最简单的形式，这种子查询称为**标量子查询**。

常用的操作符： =   <>   >    >=    <   <=   

- 案例1：查询 最早入职 的员工信息

```SQL
-- 1. 查询最早的入职时间
select min(entry_date) from emp;  -- 结果: 2000-01-01

-- 2. 查询入职时间 = 最早入职时间的员工信息
select * from emp where entry_date = '2000-01-01';

-- 3. 合并为一条SQL
select * from emp where entry_date = (select min(entry_date) from emp);
```

- 案例2：查询在 阮小五 入职之后入职的员工信息

```SQL
-- 1. 查询 "阮小五" 的入职日期
select entry_date from emp where name = '阮小五'; -- 结果: 2015-01-01

-- 2. 根据上述查询到的这个入职日期, 查询在该日期之后入职的员工信息
select * from emp where entry_date > '2015-01-01';

-- 3. 合并SQL为一条SQL
select * from emp where entry_date > (select entry_date from emp where name = '阮小五');
```

#### 列子查询

子查询返回的结果是一列(可以是多行)，这种子查询称为列子查询。

常用的操作符：

| 操作符 | 描述                         |
| ------ | ---------------------------- |
| in     | 在指定的集合范围之内，多选一 |
| not in | 不在指定的集合范围之内       |

- 案例1：查询 "教研部" 和 "咨询部" 的所有员工信息

```SQL
-- 1. 查询 "教研部" 和 "咨询部" 的部门ID
select id from dept where name = '教研部' or name = '咨询部'; -- 结果: 3,2

-- 2. 根据上面查询出来的部门ID, 查询员工信息
select * from emp where dept_id in(3,2);

-- 3. 合并SQL为一条SQL语句
select * from emp where dept_id in (select id from dept where name = '教研部' or name = '咨询部');
```

#### 行子查询

子查询返回的结果是一行(可以是多列)，这种子查询称为行子查询。

常用的操作符：= 、<> 、IN 、NOT IN

- 案例1：查询与 "李忠" 的薪资 及 职位都相同的员工信息

```SQL
-- 1. 查询 "李忠" 的薪资和职位
select salary , job from emp where name = '李忠'; -- 结果: 5000, 5

-- 2. 根据上述查询到的薪资和职位 , 查询对应员工的信息
select * from emp where (salary, job) = (5000,5);

-- 3. 将两条SQL合并为一条SQL
select * from emp where (salary, job) = (select salary , job from emp where name = '李忠');
```

#### 表子查询

子查询返回的结果是多行多列，常作为临时表，这种子查询称为**表子查询**。

- 案例：*获取每个部门中薪资最高的员工信息*

```SQL
-- a. 获取每个部门的最高薪资
select dept_id, max(salary) from emp group by dept_id;

-- b. 查询每个部门中薪资最高的员工信息
select * from emp e , (select dept_id, max(salary) max_sal from emp group by dept_id) a
    where e.dept_id = a.dept_id and e.salary = a.max_sal;
```

#### 案例

根据需求，完成多表查询的SQL语句的编写。

- 查询 "教研部" 性别为 男，且在 "2011-05-01" 之后入职的员工信息 。

```SQL
select e.* from emp as e , dept as d where e.dept_id = d.id and d.name = '教研部' and e.gender = 1 and e.entry_date > '2011-05-01';
```

- 查询工资 低于公司平均工资的 且 性别为男 的员工信息 。

```SQL
select e.* from emp as e , dept as d where e.dept_id = d.id and e.salary < (select avg(salary) from emp) and e.gender = 1;
```

- 查询部门人数超过 10 人的部门名称 。

```SQL
select d.name , count(*) from emp as e , dept as d where e.dept_id = d.id group by d.name having count(*) > 10;
```

- 查询在 "2010-05-01" 后入职，且薪资高于 10000 的 "教研部" 员工信息，并根据薪资倒序排序。

```SQL
select * from emp e , dept d where e.dept_id = d.id and e.entry_date > '2010-05-01' and e.salary > 10000 and d.name = '教研部' order by e.salary desc;
```

- 查询工资 低于本部门平均工资的员工信息 。【难】

```SQL
-- 5.1 查询每个部门的平均工资
select dept_id, avg(salary) avg_sal from emp group by dept_id;

-- 5.2 查询工资 低于本部门平均工资 的员工信息 。
select e.* from emp e , (select dept_id, avg(salary) avg_sal from emp group by dept_id) as a
          where e.dept_id = a.dept_id and e.salary < a.avg_sal;
```

## 分页查询

### 一、后端代码结构（员工管理）

员工管理模块采用 **Spring Boot + MyBatis + MySQL** 分层架构：

| 层级   | 包             | 职责                                          |
| ------ | -------------- | --------------------------------------------- |
| 表现层 | `controller`   | 接收请求、解析参数、返回统一结果              |
| 业务层 | `service/impl` | 业务逻辑处理（分页计算、时间赋值等）          |
| 数据层 | `mapper`       | 数据库 CRUD，MyBatis 注解 SQL                 |
| 实体类 | `pojo`         | 数据封装 + 统一返回结果 `Result`/`PageResult` |

> [!NOTE] 本项目用**逻辑外键**（业务层维护关联），不建物理外键，避免物理外键拖慢增删改、单机单库、易死锁等弊端。

### 二、表设计（多表关系）

员工管理涉及三张表，均为一对多关系：

- **dept 部门表**：一个部门下有多个员工（多的一方是 emp）
- **emp 员工表**：通过 `dept_id` 关联 dept 主键
- **emp_expr 员工工作经历表**：一个员工可有多条工作经历

```mysql
-- 员工表
create table emp (
  id int unsigned primary key auto_increment comment 'ID,主键',
  username varchar(20) not null unique comment '用户名',
  password varchar(32) default '123456' comment '密码',
  name varchar(10) not null comment '姓名',
  gender tinyint unsigned not null comment '性别, 1:男, 2:女',
  phone varchar(11) comment '手机号',
  job tinyint unsigned comment '职位, 1:班主任,2:讲师,3:学工主管,4:教研主管,5:咨询师',
  salary int unsigned comment '薪资',
  image varchar(255) comment '头像',
  entry_date date comment '入职日期',
  dept_id int unsigned comment '部门ID（逻辑外键）',
  create_time datetime comment '创建时间',
  update_time datetime comment '修改时间'
);

-- 员工工作经历表
create table emp_expr (
  id int unsigned primary key auto_increment comment 'ID,主键',
  emp_id int unsigned comment '员工ID（逻辑外键）',
  begin_date date comment '开始时间',
  end_date date comment '结束时间',
  company varchar(50) comment '公司名称',
  job varchar(50) comment '职位'
);
```

### 三、已实现的分页查询接口

当前员工模块只实现了分页查询，其余待开发。

**`EmpController`（表现层）**

```java
@Slf4j
@RequestMapping("/emps")
@RestController
public class EmpController {
    @Autowired
    private EmpServiceImpl empServiceImpl;   // 直接注入实现类

    @GetMapping
    public PageResult<Emp> page(Integer page, Integer pageSize){
        log.info("分页查询： {},{}",page,pageSize);
        return empServiceImpl.page(page, pageSize);
    }
}
```

**`EmpServiceImpl`（业务层）**

```java
@Service
public class EmpServiceImpl {
    @Autowired
    private EmpMapper empMapper;

    public PageResult<Emp> page(Integer page, Integer pageSize) {
        Integer start = (page - 1) * pageSize;      // 分页起始下标
        List<Emp> rows = empMapper.page(start, pageSize);

        PageResult<Emp> result = new PageResult<>();
        result.setTotal(empMapper.count());
        result.setRows(rows);
        return result;
    }
}
```

**`EmpMapper`（数据层，MyBatis 注解 SQL）**

```java
@Mapper
public interface EmpMapper {
    @Select("select e.*, d.name as deptName from emp e left join dept d on e.dept_id = d.id")
    public List<Emp> list();

    @Select("select count(*) from emp e left join dept d on e.dept_id = d.id")
    public Long count();

    @Select("select e.*, d.name as deptName from emp e left join dept d on e.dept_id = d.id "
          + " order by e.update_time desc limit #{start}, #{size}")
    public List<Emp> page(@Param("start") Integer start, @Param("size") Integer size);
}
```

**`PageResult<T>`（分页统一返回）**

```java
@Data
public class PageResult<T> {
    private Long total;      // 总记录数
    private List<T> rows;    // 当前页数据列表
}
```

### 四、核心要点

1. **RESTful 风格**：URL 定位资源、HTTP 动词描述操作（GET 查询 / POST 新增 / PUT 修改 / DELETE 删除），用复数形式如 `/emps`。

2. **统一返回结果**：`Result { code, msg, data }`，`code = 1` 成功、`0` 失败；分页用 `PageResult { total, rows }`，前端用 `rows` 渲染表格、`total` 做分页条。

3. **MyBatis 注解 SQL + 下划线转驼峰**：单表 CRUD 直接注解，不写 XML；`application.yml` 开启 `map-underscore-to-camel-case: true`，数据库 `dept_id` 自动映射为 `deptId`；关联部门名用 `d.name as deptName` 手动别名。

4. **一对多关联查询**：`emp e LEFT JOIN dept d ON e.dept_id = d.id`，取 `e.*` 加部门名。

5. **注入约定（导师建议）**：**单一实现直接用实现类**，`@Autowired EmpServiceImpl` 不抽接口；只有出现“多种规则/多实现”时才抽接口按需切换，避免为单一实现过早抽象。

6. **逻辑外键 vs 物理外键**：物理外键 `FOREIGN KEY` 拖慢增删改、单机单库、易死锁；本项目在业务层用 `dept_id` 关联，由代码保证一致性，属逻辑外键。

### 五、当前进度与待办

| 模块                                    | 状态                                    |
| --------------------------------------- | --------------------------------------- |
| 员工-分页查询                           | ✅ 已实现（`GET /emps?page=&pageSize=`） |
| 员工-按条件查询                         | ❌ 待实现（姓名/性别/入职时间范围）      |
| 员工-新增/修改/删除                     | ❌ 待实现，删除需联动删除工作经历        |
| 工作经历 `emp_expr`                     | ❌ `EmpExprMapper` 仅建骨架，业务未做    |
| 登录 `/login`                           | ❌ 待实现（无鉴权、无拦截器）            |
| 报表 `/report/emp*`、图片 `/api/upload` | ❌ 待实现（前端已调用，后端未开发）      |

> [!WARNING] 
>
> `application.yml` 中 `mybatis.type-aliases-package` 指向 `com.cyy.springbootmybatis.mybatis.entity`，与实际包名 `com.cyy.tliaswebmanagement.pojo` 不一致。当前因全用注解 SQL（无 XML）暂不影响运行，但后续写 XML Mapper 会踩坑，需改对。

### PageHelper

#### 一、为什么要用 PageHelper

原来的手写分页需要**自己算 start、自己写 LIMIT、还要单独写一条 count 统计 SQL**,条件一变两条 SQL 都得改,容易漏。

PageHelper 的核心思路:**`startPage` 一行开启分页 + 一条普通 SQL + 自动统计总数**,把 `LIMIT` 拼装和 `count(*)` 统计交给拦截器自动完成。

#### 二、依赖与配置

##### pom.xml 新增依赖

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.6</version>
</dependency>
```

> 当前项目 Spring Boot 2.6,配的是 `pagehelper-spring-boot-starter 1.4.6`。starter 会自动注册 `PageInterceptor`,无需手动配置拦截器。

##### application.yml

```yaml
mybatis:
  type-aliases-package: com.cyy.tliaswebmanagement.pojo
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true
```

> 这里**没有**额外写 `pagehelper:` 配置块,用的是默认方言。MySQL 会话下 PageHelper 会自动识别方言并追加 `limit`。

#### 三、代码改动对比

##### 3.1 PageResult 增加全参构造

```java
@Data
public class PageResult<T> {
    private Long total;        // 总记录数
    private List<T> rows;      // 当前页数据列表

    public PageResult(long total, List<T> rows) {
        this.total = total;
        this.rows = rows;
    }
}
```

> 版本更晚加上 `@AllArgsConstructor` / `@NoArgsConstructor` 也可以,原理一样。目前手写了全参构造供 `new PageResult<>(total, rows)` 使用。

##### 3.2 EmpServiceImpl：手动分页 → PageHelper

**原始写法(已注释保留)**

```java
public PageResult<Emp> page(Integer page, Integer pageSize) {
    Integer start = (page - 1) * pageSize;      // 自己算起始下标
    List<Emp> rows = empMapper.page(start, pageSize);  // 传 LIMIT
    PageResult<Emp> result = new PageResult<>();
    result.setTotal(empMapper.count());          // 单独 count
    result.setRows(rows);
    return result;
}
```

**PageHelper 写法**

```java
@Service
public class EmpServiceImpl {
    @Autowired
    private EmpMapper empMapper;

    public PageResult<Emp> page(Integer page, Integer pageSize) {
        // 1. 设置分页参数
        PageHelper.startPage(page, pageSize);

        // 2. 执行查询(普通 SQL,不带 LIMIT)
        List<Emp> list = empMapper.list();
        Page<Emp> p = (Page<Emp>) list;          // list 实际就是 Page 对象

        // 3. 解析并封装返回
        return new PageResult<>(p.getTotal(), p.getResult());
    }
}
```

**要点**

- `PageHelper.startPage(page, pageSize)` 只作用于**紧跟其后的第一条查询**。
- `empMapper.list()` 返回的虽然声明成 `List<Emp>`,运行时实际是 `com.github.pagehelper.Page` 对象,所以可以强转 `(Page<Emp>) list`。
- `p.getTotal()` 是插件自动统计的总条数,`p.getResult()` 是当前页数据。

##### 3.3 EmpMapper：删掉 count 和 LIMIT

```java
@Mapper
public interface EmpMapper {
    // 普通查询,不再写 LIMIT;总数由 PageHelper 自动统计
    @Select("select e.*, d.name as deptName from emp e left join dept d on e.dept_id = d.id order by update_time desc")
    public List<Emp> list();
}
```

**原始写法(已注释保留)**

```java
// 总记录数
@Select("select count(*) from emp e left join dept d on e.dept_id = d.id")
public Long count();

// 分页查询
@Select("select e.*, d.name as deptName from emp e left join dept d on e.dept_id = d.id  order by e.update_time desc limit #{start}, #{size}")
public List<Emp> page(@Param("start") Integer start, @Param("size") Integer size);
```

> 注意:`EmpMapper` 里还保留着一个**未删除的 `page(@Param("start")...` 带 LIMIT 的方法**,和 `list()` 同时存在。如果后续不再用旧方法,建议删掉,否则两个方法语义重复、维护时容易混淆。

##### 3.4 EmpController：分页参数加默认值

```java
@Slf4j
@RequestMapping("/emps")
@RestController
public class EmpController {
    @Autowired
    private EmpServiceImpl empServiceImpl;

    @GetMapping
    public Result page(@RequestParam(defaultValue = "1") Integer page, @RequestParam(defaultValue = "10") Integer pageSize){
        log.info("分页查询： {},{}",page,pageSize);
        return Result.success(empServiceImpl.page(page, pageSize));
    }
}
```

> `defaultValue = "1"` / `"10"` 让前端不传参数也能正常分页,避免 `page` 或 `pageSize` 为 `null` 导致 `startPage` 报错。

#### 四、PageHelper 工作原理

1. **拦截**：`PageInterceptor` 拦截 `Executor` 的 `query` 方法。
2. **改写 SQL**：检测到 `ThreadLocal` 里有分页参数,就在 SQL 末尾追加 `limit ?, ?`。
3. **自动 count**：同时生成 `select count(*) ...` 统计总数,包在 `Page` 对象里。
4. **返回值**：原 List 被强转为 `Page` 对象,里面携带 `total`、`pageNum`、`pageSize` 等。

#### 五、核心要点

1. **物理分页**：PageHelper 在数据库层加 `LIMIT`,只查当前页数据,不是内存分页。
2. **一条 SQL 两用**：查询列表 + 统计总数都由同一条 `list()` 派生,不用维护两套 SQL。
3. **startPage 就近生效**：它基于 `ThreadLocal`,只影响紧随其后的第一条查询,中间不能插别的 MyBatis 查询。
4. **强转 Page**：`List<Emp> list = empMapper.list()` 运行时就是 `Page<Emp>`,取 `getTotal()` / `getResult()`。
5. **默认值防御**：Controller 用 `defaultValue` 兜底,防止前端漏传导致空指针。

#### 六、使用注意

- `list()` 里的 `order by update_time desc` 保留排序,PageHelper 只是追加 LIMIT,不影响排序。
- 旧的手动 `page(...)` / `count()` 方法**建议清理**,避免和新方法混淆。
- 分页参数必须是正整数,`pageSize` 过大会查出大量数据,可考虑加上限。
- 若将来要**条件查询**(姓名/性别/时间),在 `list()` 的 SQL 里拼 `where` 即可,总数仍由 PageHelper 自动统计,无需再维护 count。

#### 七、小结

PageHelper 把「手算 start + 手写 LIMIT + 单独 count」三件事,压缩成「一行 `startPage` + 一条普通 `list()` + `Page` 对象取数」。代码更简洁、更易扩展,尤其适合后续加多条件分页查询。

> [!CAUTION]
>
> 1. PageHelper的查询语句后不能加";"，PageHelper会对SQL语句进行处理，在原本的语句后会加上`limit`相关语句
> 2. PageHelper只会对紧跟在其后面的第一个查询语句进行分页处理

## 条件分页查询

<img width="1281" height="396" alt="Image" src="https://github.com/user-attachments/assets/b954cd48-d09a-4e08-ac1a-262209dd7ce2" />

### 一、需求

员工列表支持按**姓名、性别、入职日期区间**筛选，同时分页显示。前端请求：

```
GET /emps?name=&gender=&begin=&end=&page=1&pageSize=10
```

要求：条件为空时不筛选（查出全部）；有值时按条件过滤；分页用 PageHelper。

### 二、涉及的四个文件

| 文件                | 职责                                                         |
| ------------------- | ------------------------------------------------------------ |
| `EmpController`     | 接收前端参数、默认值兜底、返回 `Result`                      |
| `EmpServiceImpl`    | `PageHelper.startPage` 开启分页、调 Mapper、封装 `PageResult` |
| `EmpMapper`（接口） | 定义 `list` 方法，参数用 `@Param` 命名                       |
| `EmpMapper.xml`     | 写 SQL，`namespace` 绑定接口，`id="list"` 对应方法           |

### 三、代码实现

#### 3.1 Controller —— 接收参数 + 默认值 + Result 包装

```java
@Slf4j
@RequestMapping("/emps")
@RestController
public class EmpController {

    @Autowired
    private EmpServiceImpl empServiceImpl;

    @GetMapping
    public Result page(
            @RequestParam(defaultValue = "1") Integer page,
            @RequestParam(defaultValue = "10") Integer pageSize,
            @RequestParam(required = false) String name,
            @RequestParam(required = false) Integer gender,
            @DateTimeFormat(pattern = "yyyy-MM-dd") @RequestParam(required = false) LocalDate begin,
            @DateTimeFormat(pattern = "yyyy-MM-dd") @RequestParam(required = false) LocalDate end) {

        log.info("分页查询： {},{},{},{},{},{}", page, pageSize, name, gender, begin, end);
        return Result.success(empServiceImpl.page(page, pageSize, name, gender, begin, end));
    }
}
```

**要点**

- `@RequestParam(defaultValue = "1")`：页码缺省为 1（`defaultValue` 隐含 `required = false`）。
- `@RequestParam(defaultValue = "10")`：每页条数缺省为 10。
- `@RequestParam(required = false)`：查询条件可缺省，值为 `null`，交给 SQL 判断，不报 400。
- `@DateTimeFormat(pattern = "yyyy-MM-dd")`：把 URL 里的字符串日期转成 `LocalDate`，**只在 Controller 入参生效**。
- `Result.success(...)`：外层包 `Result`，前端按 `code` + `data.total/rows` 解包。

#### 3.2 Service —— PageHelper 分页 + 封装 PageResult

```java
@Service
public class EmpServiceImpl {
    @Autowired
    private EmpMapper empMapper;

    public PageResult<Emp> page(Integer page, Integer pageSize, String name, Integer gender,
                               LocalDate begin, LocalDate end) {

        // 1. 设置分页参数
        PageHelper.startPage(page, pageSize);

        // 2. 执行查询（SQL 在 XML 里，不带 LIMIT）
        List<Emp> list = empMapper.list(name, gender, begin, end);
        Page<Emp> p = (Page<Emp>) list;          // list 运行时是 Page 对象

        // 3. 解析并封装返回
        return new PageResult<>(p.getTotal(), p.getResult());
    }
}
```

**要点**

- `PageHelper.startPage(page, pageSize)` 只作用于紧随其后的第一条查询，中间不能插别的 MyBatis 查询。
- `empMapper.list(...)` 返回的 `List` 运行时是 `com.github.pagehelper.Page`，可强转取 `getTotal()` / `getResult()`。
- `total` 由 PageHelper 自动统计，无需手写 count。

#### 3.3 Mapper 接口 —— @Param 命名参数

```java
@Mapper
public interface EmpMapper {
    List<Emp> list(@Param("name") String name, @Param("gender") Integer gender,
                   @Param("begin") LocalDate begin,
                   @Param("end") LocalDate end);
}
```

**要点**

- 多个参数必须用 `@Param("...")` 命名，XML 里才能用 `#{name}`、`#{gender}`、`#{begin}`、`#{end}` 引用。
- `@DateTimeFormat` **不要**放在 Mapper 的 `@Param` 上，它只对 Spring MVC 的 Controller 入参起作用。

#### 3.4 Mapper XML —— namespace + id 绑定

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.cyy.tliaswebmanagement.mapper.EmpMapper">

    <select id="list" resultType="com.cyy.tliaswebmanagement.pojo.Emp">
        select e.*, d.name deptName
        from emp e
        left join dept d on e.dept_id = d.id
        where
        e.name like concat('%', #{name}, '%')
        and e.gender = #{gender}
        and e.entry_date between #{begin} and #{end}
        order by e.update_time desc
    </select>

</mapper>
```

**要点**

- `namespace` 必须是接口的全限定名：`com.cyy.tliaswebmanagement.mapper.EmpMapper`。
- `id="list"` 必须和接口方法名 `list` 完全一致。
- `resultType` 指定返回实体 `com.cyy.tliaswebmanagement.pojo.Emp`。
- `like concat('%', #{name}, '%')`：实现模糊查询，`#{name}` 会作为独立参数占位符，**不能**写成 `'%#{name}%'`（会生成错误的 `'%?%'` 并导致占位符数量对不上）。

### 四、application.yml 配置

```yaml
mybatis:
  mapper-locations: classpath:com/cyy/tliaswebmanagement/mapper/*.xml
  type-aliases-package: com.cyy.tliaswebmanagement.pojo
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true
```

**要点**

- `mapper-locations` 指向 XML 所在目录，MyBatis 才能扫描到映射文件。
- XML 路径要和接口包路径对齐：接口包 `com.cyy.tliaswebmanagement.mapper` → XML 放 `resources/com/cyy/tliaswebmanagement/mapper/`。
- `map-underscore-to-camel-case: true`：数据库 `dept_id` 自动映射为 `deptId`。

### 五、实现要点速记

1. **Controller**：接收 `page/pageSize/name/gender/begin/end`，分页参数给 `defaultValue`，查询条件给 `required = false`，返回 `Result.success(...)`。
2. **Service**：`PageHelper.startPage(page, pageSize)` + `empMapper.list(...)` + `new PageResult<>(p.getTotal(), p.getResult())`。
3. **Mapper 接口**：多参数用 `@Param` 命名。
4. **XML**：`namespace` = 接口全名，`id` = 方法名，SQL 里用 `#{name}` 引用参数。
5. **前端期望结构**：`{ code: 1, data: { total, rows } }`，所以 Controller 用 `Result.success(PageResult)`。

### 六、当前代码的一个待优化点

现在 XML 里三个条件是**固定条件**（`where name like ... and gender = ... and between ...`）。若前端不传条件（`name=null`、`gender=null`、`begin/end=null`），SQL 会变成 `name = null` 之类，**查不到任何数据**，页面显示"暂无数据"。

要支持"无条件下查出全部"，应改成 **MyBatis 动态 SQL**（`<where> + <if>`），条件有值才拼：

```xml
<select id="list" resultType="com.cyy.tliaswebmanagement.pojo.Emp">
    select e.*, d.name deptName
    from emp e
    left join dept d on e.dept_id = d.id
    <where>
        <if test="name != null and name != ''">
            and e.name like concat('%', #{name}, '%')
        </if>
        <if test="gender != null">
            and e.gender = #{gender}
        </if>
        <if test="begin != null">
            and e.entry_date &gt;= #{begin}
        </if>
        <if test="end != null">
            and e.entry_date &lt;= #{end}
        </if>
    </where>
    order by e.update_time desc
</select>
```

> `<where>` 会自动去掉第一个多余的 `and`；若所有 `<if>` 都不成立，则**不生成** `where` 子句，从而查出全部数据。

### 七、小结

条件是分页查询在 PageHelper 基础上扩展了**多条件筛选**：

- 分页：靠 `PageHelper.startPage` 自动完成，SQL 不带 LIMIT。
- 筛选：靠 XML 里按需拼 `where`（当前为固定条件，建议改动态 SQL）。
- 串联：Controller 收参 → Service 开分页 → Mapper 查数据 → `Result.success(PageResult)` 返回前端。

### 请求参数接收优化

**EmpQueryParam数据类**

```java
@Data
public class EmpQueryParam {
    Integer page;
    Integer pageSize;
    String name;
    Integer gender;
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    LocalDate begin;
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    LocalDate end;
}
```

**Controller层**

```java
@GetMapping
    public Result page(EmpQueryParam param) {

        log.info("分页查询： {},{},{},{},{},{}", param.getPage(), param.getPageSize(), param.getName(), param.getGender(), param.getBegin(), param.getEnd());
        return Result.success(empServiceImpl.page(param));
    }
```

**Service层**

```java
public PageResult<Emp> page(EmpQueryParam param) {

        Integer page = param.getPage();
        Integer pageSize = param.getPageSize();

        // 设置分页参数
        PageHelper.startPage(page, pageSize);

        // 执行查询
        List<Emp> list = empMapper.list(param);
        Page<Emp> p = (Page<Emp>) list;
        // 解析查询结果并封装
        return  new PageResult<>(p.getTotal(),p.getResult());

    }
```

**Mapper层**

```java
public List<Emp> list(EmpQueryParam param);
```

### 程序优化-动态sql

```xml
<select id="list" resultType="com.cyy.tliaswebmanagement.pojo.Emp">
    select e.*, d.name deptName
    from emp e
    left join dept d on e.dept_id = d.id
    <where>
        <if test="name != null and name != ''">
            and e.name like concat('%', #{name}, '%')
        </if>
        <if test="gender != null">
            and e.gender = #{gender}
        </if>
        <if test="begin != null">
            and e.entry_date &gt;= #{begin}
        </if>
        <if test="end != null">
            and e.entry_date &lt;= #{end}
        </if>
    </where>
    order by e.update_time desc
</select>
```



- **<if>标签**：条件判断，如果条件成立，则拼接对应的sql片段
- **<where标签>**：根据查询条件生成where关键字，并自动去除条件前多余的and或or

## 新增员工

#### 一、需求概述

新增员工时，前端提交一个员工对象，该对象中不仅包含员工基本信息，还包含多条工作经历（`exprList`）。

因此新增员工需要分两步完成：

1. 向员工表 `emp` 插入员工基本信息；
2. 向工作经历表 `emp_expr` 批量插入该员工的多条工作经历。

由于工作经历表中的 `emp_id` 要关联刚插入的员工，所以第 1 步插入后必须**拿到数据库生成的主键**。

#### 二、三层架构调用流程

```text
前端 POST /emps
    ↓
Controller.save(@RequestBody Emp emp)
    ↓
Service.save(Emp emp)
    ├── empMapper.insert(emp)          插入员工，主键回填到 emp.id
    └── empExprMapper.insertBatch(list) 批量插入工作经历
    ↓
Mapper（注解 + XML）
    ↓
MySQL emp / emp_expr
```

#### 三、Controller 层

```java
// 添加员工
@PostMapping
public Result save(@RequestBody Emp emp) {
    log.info("添加员工：{}", emp);
    empServiceImpl.save(emp);
    return Result.success();
}
```

**要点**

- `@PostMapping`：接收新增请求。
- `@RequestBody`：把前端 JSON 数据转换成 `Emp` 对象。
- `Emp` 中的 `exprList` 会一起被反序列化，无需分别接收工作经历。

#### 四、Service 层

```java
@Service
public class EmpServiceImpl {
    @Autowired
    private EmpMapper empMapper;
    @Autowired
    private EmpExprMapper empExprMapper;

    public void save(Emp emp) {
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());

        // 1. 插入员工基本信息，插入后主键回填到 emp.getId()
        empMapper.insert(emp);

        // 2. 保存员工工作经历
        List<EmpExpr> list = emp.getExprList();
        if (!list.isEmpty()) {
            // 工作经历需要关联员工主键
            for (EmpExpr empExpr : list) {
                empExpr.setEmpId(emp.getId());
            }
            empExprMapper.insertBatch(list);
        }
    }
}
```

**要点**

- 先设置 `createTime`、`updateTime`。
- `empMapper.insert(emp)` 执行后，emp 的主键已经被回填，因此可以直接调用 `emp.getId()`。
- 遍历工作经历，为每条记录设置 `empId`。
- 只有工作经历不为空时才执行批量插入，避免不必要的 SQL。

#### 五、Mapper 层

##### 5.1 员工表 Mapper —— 主键返回

`EmpMapper.java`：

```java
// 添加员工
@Options(useGeneratedKeys = true, keyProperty = "id") // 获取数据库生成的主键
@Insert("insert into emp(username, name, gender, phone, job, salary, image, entry_date, dept_id, create_time, update_time) " +
        "values (#{username},#{name},#{gender},#{phone},#{job},#{salary},#{image},#{entryDate},#{deptId},#{createTime},#{updateTime})")
public void insert(Emp emp);
```

**数据库新方法：主键返回**

`@Options` 的常用属性：

| 属性                      | 作用                                        |
| ------------------------- | ------------------------------------------- |
| `useGeneratedKeys = true` | 告诉 MyBatis 使用数据库自动生成的主键       |
| `keyProperty = "id"`      | 将生成的主键回填到 `Emp` 对象的 `id` 属性中 |

因为 `EmpMapper.insert()` 的返回值是 `void`，插入成功与否不能通过返回值判断主键，而是靠 `keyProperty` 把主键写回传入的 `emp` 对象。

##### 5.2 工作经历 Mapper —— 批量插入

`EmpExprMapper.java`：

```java
@Mapper
public interface EmpExprMapper {
    // 批量添加员工工作经历
    void insertBatch(@Param("exprList") List<EmpExpr> exprList);
}
```

**要点**

- 参数必须使用 `@Param("exprList")` 命名，XML 中的 `<foreach collection="exprList">` 才能找到集合。
- 如果是单个未加 `@Param` 的 `List`，MyBatis 只提供 `collection`、`list` 等默认名称，找不到 `exprList`。

#### 六、Mapper XML 文件

`EmpExprMapper.xml`：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.cyy.tliaswebmanagement.mapper.EmpExprMapper">
    <insert id="insertBatch" parameterType="java.util.List">
        insert into emp_expr (emp_id, company, job, begin, end) values
        <foreach collection="exprList" item="expr" separator=",">
            (#{expr.empId}, #{expr.company}, #{expr.job}, #{expr.begin}, #{expr.end})
        </foreach>
    </insert>
</mapper>
```

#### 七、新用到的 XML 标签：foreach

`<foreach>` 用于遍历集合，生成批量 SQL。

```xml
<foreach collection="exprList" item="expr" separator=",">
    (#{expr.empId}, #{expr.company}, #{expr.job}, #{expr.begin}, #{expr.end})
</foreach>
```

**属性说明**

| 属性         | 作用                                       | 本例值     |
| ------------ | ------------------------------------------ | ---------- |
| `collection` | 要遍历的集合名称，与 `@Param` 名称一致     | `exprList` |
| `item`       | 每次遍历的元素变量名，`#{expr.xxx}` 中使用 | `expr`     |
| `separator`  | 每次循环结果之间的分隔符                   | `,`        |
| `open`       | 遍历开始前拼接的无段（可选）               | 无         |
| `close`      | 遍历开始前拼接的片段（可选）               | 无         |

如果 `exprList` 中有两条数据，最终生成的 SQL 类似：

```sql
insert into emp_expr (emp_id, company, job, begin, end) values
(1, 'A公司', 'Java开发', '2023-01-01', '2024-01-01'),
(1, 'B公司', '前端开发', '2024-02-01', '2025-02-01')
```

#### 八、数据库表结构

员工表中 `id` 是自增主键，所以插入时可以不给 `id`，由数据库自动生成。

```mysql
-- 员工表
create table emp(
    id int unsigned primary key auto_increment comment 'ID,主键',
    username varchar(20) not null unique comment '用户名',
    password varchar(50) default '123456' comment '密码',
    name varchar(10) not null comment '姓名',
    gender tinyint unsigned not null comment '性别, 1:男, 2:女',
    phone char(11) not null unique comment '手机号',
    job tinyint unsigned comment '职位',
    salary int unsigned comment '薪资',
    image varchar(300) comment '头像',
    entry_date date comment '入职日期',
    dept_id int unsigned comment '部门ID',
    create_time datetime comment '创建时间',
    update_time datetime comment '修改时间'
) comment '员工表';

-- 员工工作经历表
create table emp_expr(
    id int unsigned primary key auto_increment comment 'ID, 主键',
    emp_id int unsigned null comment '员工ID',
    begin date null comment '开始时间',
    end date null comment '结束时间',
    company varchar(50) null comment '公司名称',
    job varchar(50) null comment '职位'
) comment '工作经历';
```

#### 九、代码中的注意事项

1. 当前 `EmpExprMapper.java` 中的 `insertBatch` 方法需要加 `@Param("exprList")`，否则 XML 的 `<foreach collection="exprList">` 会报错。
2. 当前 `EmpExprMapper.xml` 顶部可能错误地保留了 Java 注解字符串写法（包含引号和 `+`），必须是纯 XML 内容，例如上面的标准写法。
3. `EmpExpr` 实体类的字段与数据库列保持一致：
   - `empId` ↔ `emp_id`
   - `company` ↔ `company`
   - `job` ↔ `job`
   - `begin` ↔ `begin`
   - `end` ↔ `end`
4. 新增员工涉及两张表的写入，目前 Service 方法没有加 `@Transactional`；若希望出现异常时一起回滚，可在 `save` 方法上添加 `@Transactional`。

#### 十、核心知识速记

- `@PostMapping` + `@RequestBody`：接收新增员工 JSON。
- `@Options(useGeneratedKeys = true, keyProperty = "id")`：插入后获取自增主键并回填。
- `<foreach>`：批量生成 `values (...), (...)`，`collection` 对应 `@Param` 名称，`item` 是元素变量名，`separator` 是拼接分隔符。
- 插入员工后再遍历设置 `empId`，从而建立员工与工作经历的一对多关系。

## 事务管理

### 介绍与操作

**背景**：保存员工的基本信息成功了，而保存工作经历失败了，是否可行？

- 不可以
- 这属于同一个完整的业务操作，如果保存员工信息成功了，保存工作经历信息失败了，就会造成数据库数据的不完整、不一致

#### 介绍

**概念**：事务 是一组操作的集合，它是一个不可分割的工作单位。事务会把所有的操作作为一个整体一起向系统提交或撤销
操作请求，即这些操作 要么同时成功，要么同时失败。

> [!CAUTION]
>
> 默认MySQL的事务是自动提交的，也就是说，当执行一条DML（数据操作）语句，MySQL会立即隐式的提交事务

#### 操作

事务控制主要的三步操作：开启事务、提交事务/回滚事务

- 需要在这组操作执行之前，先开启事务  ( `start transaction; / begin;`)。
- 所有操作如果全部都执行成功，则提交事务 ( `commit;` )。
- 如果这组操作中，有任何一个操作执行失败，都应该回滚事务 ( `rollback` )。

### Spring事务管理-介绍

**注解：**@Transactional

**作用：**就是在当前这个方法执行开始**之前**来开启事务，方法执行完毕之后提交事务。如果在这个方法执行的过程当中**出现了异常**，就会进行事务的回滚操作。

**位置**：**业务层的方法上**、类上、接口上

- 方法上：当前方法交给spring进行事务管理

- 类上：当前类中所有的方法都交由spring进行事务管理 

- 接口上：接口下所有的实现类当中所有的方法都交给spring 进行事务管理

- > [!TIP]
  >
  > 建议在业务层的方法上加@Transactional注解，而且是加在**多次对数据库进行增删改**的方法上

  

### Spring事务管理-进阶&事务四大特性

- 异常回滚的属性：`rollbackFor `
- 事务传播行为：`propagation`

#### rollbackFor

- @Transactional注解中的一个属性，可以指定出现哪些异常时回滚
- 默认情况下，**只有出现RuntimeException才会回滚**

#### propagation

事务传播行为：指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行事务控制

例如：两个事务方法，一个A方法，一个B方法。在这两个方法上都添加了@Transactional注解，就代表这两个方法都具有事务，而在A方法当中又去调用了B方法。

所谓事务的传播行为，指的就是在A方法运行的时候，首先会开启一个事务，在A方法当中又调用了B方法， B方法自身也具有事务，那么B方法在运行的时候，到底是**加入到A方法的事务当中**来，还是B方法**在运行的时候新建一个事务**？这个就涉及到了事务的传播行为。

**通过 propagation 属性来指定传播行为**

| 属性值        | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| REQUIRED      | 【默认值】需要事务，有则加入，无则创建新事务                 |
| REQUIRES_NEW  | 需要新事务，无论有无，总是创建新事务                         |
| SUPPORTS      | 支持事务，有则加入，无则在无事务状态中运行                   |
| NOT_SUPPORTED | 不支持事务，在无事务状态下运行，如果当前存在已有事务，则挂起当前事务 |
| MANDATORY     | 必须有事务，否则抛异常                                       |
| NEVER         | 必须没事务，否则抛异常                                       |

> [!NOTE]
>
> 对于这些事务传播行为，只需要关注以下两个就可以了：
>
> 1. REQUIRED（默认值）
> 2. REQUIRES_NEW

> [!TIP]
>
> - **REQUIRED：**大部分情况下都是用该传播行为即可。
> - **REQUIRES_NEW：**当我们不希望事务之间相互影响时，可以使用该传播行为。比如：下订单前需要记录日志，不论订单保存成功与否，都需要保证日志记录能够记录成功。



#### 事务四大特性

| 特性   | 英文        | 含义                                                         |
| ------ | ----------- | ------------------------------------------------------------ |
| 原子性 | Atomicity   | 事务是不可分割的最小单元，要么全部成功，要么全部失败         |
| 一致性 | Consistency | 事务完成时，必须使所有的数据都保持一致状态                   |
| 隔离性 | Isolation   | 数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行 |
| 持久性 | Durability  | 事务一旦提交或回滚，它对数据库中的数据的改变就是永久的       |

**补充说明**

- **原子性（Atomicity）** ：原子性是指事务包装的一组sql是一个不可分割的工作单元，事务中的操作要么全部成功，要么全部失败。
- **一致性（Consistency）**：一个事务完成之后数据都必须处于一致性状态。
  - 如果事务成功的完成，那么数据库的所有变化将生效。
  - 如果事务执行出现错误，那么数据库的所有变化将会被回滚(撤销)，返回到原始状态。
- **隔离性（Isolation）**：多个用户并发的访问数据库时，一个用户的事务不能被其他用户的事务干扰，多个并发的事务之间要相互隔离。
  - 一个事务的成功或者失败对于其他的事务是没有影响。
- **持久性（Durability）**：一个事务一旦被提交或回滚，它对数据库的改变将是永久性的，哪怕数据库发生异常，重启之后数据亦然存在。

## 文件上传

```java
@Slf4j
@RestController
public class UploadController {
    @PostMapping("/upload")
    public Result upload(String username,Integer age, MultipartFile file) throws IOException {
        log.info("接收参数：{}，{}，{}", username, age, file.getOriginalFilename());
        String originalFilename = file.getOriginalFilename();
        String extension = originalFilename.substring(originalFilename.lastIndexOf("."));
        String newFileName = UUID.randomUUID().toString() + "." + extension;
        file.transferTo(new File("C:/javatmp/upload/" + newFileName));
        return Result.success();
    }
}
```

```yaml
  servlet:
    multipart:
      # 最大单个文件大小
      max-file-size: 10MB
      # 最大请求大小
      max-request-size: 100MB
```

## 删除员工

### Controller接收请求参数

请求参数： DELETE /emps?ids=1,2,3

**方式一：在Controller方法中通过数组来接收**

多个参数，默认可以将其封装到一个数组中，需要保证前端传递的参数名 与 方法形参名称保持一致。

```java
/**
* 批量删除员工
*/
@DeleteMapping
public Result delete(Integer[] ids){
    log.info("批量删除部门: ids={} ", Arrays.asList(ids));
    return Result.success();
}
```

**方式二：在Controller方法中通过集合来接收**

也可以将其封装到一个List\<Integer> 集合中，如果要将其封装到一个集合中，需要在集合前面加上 `@RequestParam` 注解。

```Java
/**
* 批量删除员工
*/
@DeleteMapping
public Result delete(@RequestParam List<Integer> ids){
    log.info("批量删除部门: ids={} ", ids);
    empService.deleteByIds(ids);
    return Result.success();
}
```

### 员工删除功能代码与报错总结

#### 一、涉及代码

删除员工功能涉及 Controller → Service → Mapper 接口 → Mapper XML 四个环节，共五份代码：

**1. Controller：EmpController.java**

```java
// (批量)删除员工
@DeleteMapping
public Result delete(@RequestParam List<Integer> ids) {
    log.info("删除员工：{}", ids);
    empServiceImpl.delete(ids);
    return Result.success();
}
```

**2. Service 接口：EmpService.java**

```java
public void delete(List<Integer> ids);
```

**3. Service 实现类：EmpServiceImpl.java**

```java
// (批量)删除员工
@Transactional(rollbackFor = {Exception.class})
public void delete(List<Integer> ids) {
    empMapper.deleteByIds(ids);
    empExprMapper.deleteByEmpIds(ids);
}
```

**4. Mapper 接口：EmpMapper.java**

```java
// (批量)删除员工
void deleteByIds(List<Integer> ids);
```

**5. Mapper XML：EmpMapper.xml**

```xml
<delete id="deleteByIds">
    delete from emp where id in
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</delete>
```

**6. Mapper 接口：EmpExprMapper.java**

```java
// 根据员工id(批量)删除工作经历
void deleteByEmpIds(List<Integer> empIds);
```

**7. Mapper XML：EmpExprMapper.xml**

```xml
<delete id="deleteByEmpIds">
    delete from emp_expr where emp_id in
    <foreach collection="empIds" item="empId" open="(" separator="," close=")">
        #{empId}
    </foreach>
</delete>
```

#### 二、遇到的问题

删除员工时报错，核心异常信息为：

```cmd
org.apache.ibatis.binding.BindingException: Parameter 'ids' not found.
Available parameters are [arg0, collection, list]
```

#### 三、报错原因

MyBatis 批量删除的 List 参数没有加 `@Param` 注解，导致 XML 中 `collection="ids"` 和 `collection="empIds"` 无法匹配到参数的名称。

当方法只有一个 List 参数且未加 `@Param` 时，MyBatis 只能使用默认别名 `list` / `collection` / `arg0`，找不到 `ids`，因此抛出 BindingException。

#### 四、修复方案

在 Mapper 方法参数上添加 `@Param` 注解，使参数名与 XML 中 `collection` 的属性值保持一致。

修改后为：

```java
// EmpMapper.java
void deleteByIds(@Param("ids") List<Integer> ids);

// EmpExprMapper.java
void deleteByEmpIds(@Param("empIds") List<Integer> empIds);
```

#### 五、知识点总结

单独使用 `List` / `Collection` 作为 Mapper 参数时，建议用 `@Param("名字")` 显式指定：

- 单个普通参数（如 `Integer`、`String`）时，MyBatis 不关注参数名，XML 直接写 `#{id}` / `#{name}` 即可。
- 多个参数或单个 `List` / `Collection` / 数组参数时，MyBatis 需要知道参数对应的名称，否则只能使用默认别名 `list` / `collection` / `arg0`。
- 加 `@Param` 后，参数名清晰、可读性好，是推荐写法。

## 修改员工

### 查询回显

#### 一、目标

回显员工基本信息，并同时回显该员工的工作经历列表。前端打开"编辑员工"页面时，需要根据 id 一次拿到：员工基本信息 + 部门名称 + 工作经历列表。

#### 二、实现方式

采用"分别查询再拼装"：先查员工基本信息，再查工作经历，最后把工作经历封装到员工的 `exprList` 字段中。

```java
public Emp getById(Integer id) {
    Emp emp = empMapper.getById(id);                 // 查询员工基本信息（含部门名称）
    List<EmpExpr> empExprList = getEmpExprByEmpId(id); // 查询工作经历列表
    emp.setExprList(empExprList);                      // 封装工作经历
    return emp;
}

public List<EmpExpr> getEmpExprByEmpId(Integer id) {
    return empExprMapper.getByEmpId(id);
}
```

#### 三、涉及的 SQL

员工基本信息（`EmpMapper.xml`）通过 LEFT JOIN 拿到部门名称：

```xml
<select id="getById" resultType="com.cyy.tliaswebmanagement.pojo.Emp">
    select e.*, d.name as deptName
    from emp e
    left join dept d on e.dept_id = d.id
    where e.id = #{id}
</select>
```

工作经历列表（`EmpExprMapper.xml`）按 emp_id 查询并按开始时间排序：

```xml
<select id="getByEmpId" resultType="com.cyy.tliaswebmanagement.pojo.EmpExpr">
    select * from emp_expr where emp_id = #{empId} order by begin
</select>
```

#### 四、实体字段

- `Emp` 中新增 `deptName`（部门名称）和 `exprList`（工作经历列表），均为非表字段，只用于回显。
- `EmpExpr` 对应 `emp_expr` 表的字段。

#### 五、存在的问题与改进方向

- 单条回显是两次查询，接近"先查主表再查子表"，在单条场景性能没问题。
- 若将来列表页也要回显工作经历，不应写成"N+1"，而应升级为"批量 IN 查询 + 按 empId 分组拼装"，固定两次查询。

---

### 修改数据

#### 一、目标

修改员工基本信息，同时更新员工的工作经历列表。前端提交的是一个包含 `exprList` 的 `Emp` 对象。

#### 二、实现方式

采用"先删后增"的方式更新工作经历，并用事务保证整体成功或整体回滚。

```java
@Transactional(rollbackFor = Exception.class)
public void update(Emp emp) {
    emp.setUpdateTime(LocalDateTime.now());
    empMapper.update(emp);

    // 删除该员工原有的全部工作经历
    empExprMapper.deleteByEmpIds(Collections.singletonList(emp.getId()));

    // 批量插入前端传回的工作经历
    List<EmpExpr> exprList = emp.getExprList();
    if (exprList != null && !exprList.isEmpty()) {
        for (EmpExpr expr : exprList) {
            expr.setEmpId(emp.getId());
        }
        empExprMapper.insertBatch(exprList);
    }
}
```

#### 三、涉及的 SQL

修改员工基本信息（`EmpMapper.xml`），注意**不更新 create_time**，避免把创建时间清空：

```xml
<update id="update">
    update emp set username = #{username}, name = #{name}, gender = #{gender},
           phone = #{phone}, job = #{job}, salary = #{salary}, image = #{image},
           entry_date = #{entryDate}, dept_id = #{deptId}, update_time = #{updateTime}
    where id = #{id}
</update>
```

删除旧工作经历（`EmpExprMapper.xml`）：

```xml
<delete id="deleteByEmpIds">
    delete from emp_expr where emp_id in
    <foreach collection="empIds" item="empId" open="(" separator="," close=")">
        #{empId}
    </foreach>
</delete>
```

批量插入新工作经历（`EmpExprMapper.xml`）：

```xml
<insert id="insertBatch" parameterType="java.util.List">
    insert into emp_expr (emp_id, company, job, begin, end) values
    <foreach collection="exprList" item="expr" separator=",">
        (#{expr.empId}, #{expr.company}, #{expr.job}, #{expr.begin}, #{expr.end})
    </foreach>
</insert>
```

#### 四、遇到的问题及解决

- 问题 1：之前的 `EmpExprMapper.xml` 中有一个 `update` 语句，用 `<foreach>` 直接拼多条 update，导致 SQL 语法错误。
  解决：删除该错误 update，改用"先删后增"，逻辑更清晰、更可靠，且能覆盖新增、删除、修改三种情况。

- 问题 2：原 `EmpMapper.xml` 的 update 里面包含 `create_time = #{createTime}`，如果前端未回传 createTime，会把创建时间覆盖为 null。
  解决：从 update 语句中移除 `create_time`，只更新业务字段和 update_time。

- 问题 3：修改工作经历必须保证整体成功。
  解决：`update` 方法加上 `@Transactional(rollbackFor = Exception.class)`，删除和插入在同一个事务中，任一失败则回滚。

#### 五、关键知识点

- 修改工作经历采用"先删后增"，用一个事务包住删除与插入，保证数据一致。
- `@Transactional` 默认只对 RuntimeException 回滚，业务上建议显式 `rollbackFor = Exception.class`。
- 批量插入/删除工作经历都复用了 `insertBatch` 和 `deleteByEmpIds`，无需新增 update 语句。
- 修改员工基本信息时不要更新 `create_time`，避免覆盖创建时间。

### 修改员工-程序优化

**动态sql**

```xml
<update id="update">
        update emp
        <set>
            <if test="username != null and username != ''">username = #{username},</if>
            <if test="name != null and name != ''">name = #{name},</if>
            <if test="gender != null">gender = #{gender},</if>
            <if test="phone != null and phone != ''">phone = #{phone},</if>
            <if test="job != null">job = #{job},</if>
            <if test="salary != null">salary = #{salary},</if>
            <if test="image != null and image != ''">image = #{image},</if>
            <if test="entryDate != null">entry_date = #{entryDate},</if>
            <if test="deptId != null">dept_id = #{deptId},</if>
            <if test="updateTime != null">update_time = #{updateTime},</if>
        </set>
        where id = #{id}
    </update>
```

**\<set>标签**

- 会自动生成set关键字
- 会自动删除更新字段后多余的"，"

## 全局异常处理

**当前异常处理**：未作处理

**全局异常处理器**

```java
/**
 * 全局异常处理器
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 处理参数类型不匹配异常
     *
     * @param e 参数类型不匹配异常
     * @return 统一的错误结果
     */
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public Result handleTypeMismatch(MethodArgumentTypeMismatchException e) {
        log.warn("参数类型不匹配: {}", e.getMessage());
        return Result.error("参数类型不匹配，请检查请求参数");
    }


    /**
     * 处理重复键异常
     *
     * @param e 重复键异常
     * @return 统一的错误结果
     */
    @ExceptionHandler(DuplicateKeyException.class)
    public Result handleDup(DuplicateKeyException e) {
        log.warn("重复键异常: {}", e.getMessage());
        return Result.error("数据已存在，请检查输入");
    }

    /**
     * 处理请求体绑定/校验异常
     *
     * @param e 参数校验异常
     * @return 统一的错误结果
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result handleValid(MethodArgumentNotValidException e) {
        log.warn("参数校验失败: {}", e.getMessage());
        return Result.error("参数校验失败，请检查请求参数");
    }

    /**
     * 处理通用异常（兜底）
     *
     * @param e 异常对象
     * @return 统一的错误结果
     */
    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e) {
        log.error("系统异常", e);
        return Result.error("系统异常，请联系管理员");
    }
}

```



**如何定义一个全局异常处理器?**

- @RestControllerAdvice = @ControllerAdvice + @ResponseBody
- @ExceptionHandler

## 员工信息统计

### 职位统计

#### 1. 目标

统计各职位对应的员工人数，供前端图表展示（如饼图、柱状图）。

#### 2. 实现方式

- Mapper 层用 `GROUP BY job` 按职位编号分组统计人数。
- Service 层把职位编号（1~5）翻译成中文名称，并封装成两个对齐的列表。
- 返回 `JobOption` 对象，包含 `jobList`（职位名称列表）和 `dataList`（人数列表）。

#### 3. Mapper 查询

```xml
<select id="countJob" resultType="java.util.HashMap">
    select job, count(*) as count
    from emp
    group by job
</select>
```

#### 4. Service 逻辑

```java
public JobOption countJob() {
    List<Map<String, Object>> list = empMapper.countJob();

    Map<Integer, String> jobNameMap = new HashMap<>();
    jobNameMap.put(1, "班主任");
    jobNameMap.put(2, "讲师");
    jobNameMap.put(3, "学工主管");
    jobNameMap.put(4, "教研主管");
    jobNameMap.put(5, "咨询师");

    List<String> jobList = new ArrayList<>();
    List<Integer> dataList = new ArrayList<>();
    for (Map<String, Object> map : list) {
        Integer jobId = toInteger(map.get("job"));
        String jobName = jobNameMap.getOrDefault(jobId, String.valueOf(jobId));
        jobList.add(jobName);
        dataList.add(toInteger(map.get("count")));
    }
    return new JobOption(jobList, dataList);
}
```

#### 5. 封装类 JobOption

```java
public class JobOption {
    private List<String> jobList;   // 职位名称列表
    private List<Integer> dataList; // 对应人数列表
}
```

#### 6. Controller 接口

```java
@GetMapping("/empJobData")
public Result getEmpJobData() {
    JobOption jobOption = empServiceImpl.countJob();
    return Result.success(jobOption);
}
```

#### 7. 返回示例

请求 `GET /report/empJobData`：

```json
{
  "code": 1,
  "msg": "success",
  "data": {
    "jobList": ["班主任", "讲师", "教研主管"],
    "dataList": [15, 23, 8]
  }
}
```

---

### 性别统计

#### 1. 目标

统计男女员工人数，供前端图表展示。

#### 2. 实现方式

- Mapper 层用 `GROUP BY gender` 按性别编号分组统计人数。
- Service 层把性别编号（1:男，2:女）翻译成中文名称。
- 返回 `List<Map<String, Object>>`，每项包含 `name`（性别名称）和 `value`（人数），适配前端 ECharts 的 `name/value` 结构。

#### 3. Mapper 查询

```xml
<select id="countGender" resultType="java.util.Map">
    select gender, count(*) as count
    from emp
    group by gender
</select>
```

#### 4. Service 逻辑

```java
public List<Map<String, Object>> countGender() {
    List<Map<String, Object>> list = empMapper.countGender();

    Map<Integer, String> genderNameMap = new HashMap<>();
    genderNameMap.put(1, "男");
    genderNameMap.put(2, "女");

    List<Map<String, Object>> result = new ArrayList<>();
    for (Map<String, Object> map : list) {
        Integer genderId = toInteger(map.get("gender"));
        String genderName = genderNameMap.getOrDefault(genderId, String.valueOf(genderId));

        Map<String, Object> item = new HashMap<>();
        item.put("name", genderName);
        item.put("value", toInteger(map.get("count")));
        result.add(item);
    }
    return result;
}
```

#### 5. Controller 接口

```java
@GetMapping("/empGenderData")
public Result getEmpGenderData() {
    List<Map<String, Object>> genderData = empServiceImpl.countGender();
    return Result.success(genderData);
}
```

#### 6. 返回示例

请求 `GET /report/empGenderData`：

```json
{
  "code": 1,
  "msg": "success",
  "data": [
    { "name": "男", "value": 15 },
    { "name": "女", "value": 23 }
  ]
}
```

---

### 公共辅助方法

两个统计都用到了 `toInteger` 安全转换：

```java
private Integer toInteger(Object value) {
    if (value instanceof Number) {
        return ((Number) value).intValue();
    }
    if (value != null) {
        try {
            return Integer.parseInt(value.toString());
        } catch (NumberFormatException ignored) {
            // 忽略转换失败
        }
    }
    return null;
}
```

作用：把 MyBatis 从数据库查回的 `Object`（可能是 Integer、Long、String）安全转成 `Integer`，避免 `ClassCastException`。

### 关键知识点

- 都用 `GROUP BY` 聚合 + `count(*)` 统计。
- MyBatis 返回的 Map 中，key 是列名/别名（`job`、`gender`、`count`），value 是实际值。
- 编号到中文名称通过手动 `Map` 映射实现。
- `getOrDefault` 保证未知编号也能正常展示，不会报错或返回 `null`。
- 职位统计返回 `JobOption`（两个 List），性别统计返回 `List<Map>`（name/value），两者结构不同，取决于前端图表需要的格式。
