# 班级管理 重点笔记（结合代码）

> 模块：班级管理（`ClazzController` + `ClazzServiceImpl` + `ClazzMapper`）
> 基础路径：`/clazzs`　|　统一返回：`Result`（`code=1` 成功，`code=0` 失败）

## 一、接口与实现概览

| 接口                | 方法       | 路径                  | 核心逻辑                          |
| ------------------- | ---------- | --------------------- | --------------------------------- |
| 条件分页查询（2.1） | `page`     | `GET /clazzs`         | PageHelper + 动态条件             |
| 查询所有员工（2.2） | `listAll`  | `GET /emps/list`      | 班主任下拉                        |
| 新增（2.3）         | `save`     | `POST /clazzs`        | 自动补 `create/update_time`       |
| 按ID查询（2.4）     | `findById` | `GET /clazzs/{id}`    | 主键查询                          |
| 修改（2.5）         | `update`   | `PUT /clazzs`         | 动态 `<set>` + 刷新 `update_time` |
| 删除（2.6）         | `delete`   | `DELETE /clazzs/{id}` | 先校验学生再删，事务回滚          |
| 查询所有班级（3.1） | `listAll`  | `GET /clazzs/list`    | 学员下拉                          |

## 二、怎么实现 & 为什么这么实现

### 1. 条件分页查询（`page`）

- **实现**：`PageHelper.startPage(page, pageSize)` 启动分页，紧接着调用 `clazzMapper.list(clazzQueryParam)`，拿到 `List<Clazz>` 后强转为 `Page<Clazz>`，取出 `p.getTotal()`（总条数）和 `p.getResult()`（当前页），封装为 `PageResult<Clazz>`（`total` + `rows`）。
- **为什么**：直接返回 `List` 前端拿不到总记录数，无法渲染分页组件；PageHelper 通过拦截器改写 SQL，自动生成 `count` 与 `limit`，减少手写分页代码。

### 2. 动态条件拼接（`ClazzMapper.xml #list`）

- **实现**：`<where>` + `<if>` 动态拼接：
  - `name` 非空 → `c.name like concat('%', #{name}, '%')`
  - `begin` 非空 → `c.end_date >= #{begin}`
  - `end` 非空 → `c.end_date <= #{end}`
  - 其余 `order by c.update_time desc`
- **实现回显字段**：`left join emp e on c.master_id = e.id`，取 `e.name as masterName`；用 `case when` 结合 `curdate()` 实时算 `status`（未开班/已开班/已结课）。
- **为什么**：`master_id` 只存 ID，前端要显示姓名，必须联表；`status` 不是表字段，用日期动态判断节省一张状态表，且不会因时间推移过期。

### 3. 新增（`save`）

- **实现**：`clazz.setCreateTime(now)`、`setUpdateTime(now)`，再 `clazzMapper.add(clazz)`。
- **为什么**：时间由后端统一生成，前端不用传，保证各端格式一致（`LocalDateTime`）。

### 4. 修改（`update`）

- **实现**：`clazz.setUpdateTime(now)`，走 `clazzMapper.update`（XML 中 `<set>` + `<if>`，仅更新非空字段），`where id = #{id}`。
- **为什么**：修改接口只传部分字段（如只改教室），`<set>` 不会把未传字段覆盖成 null。

### 5. 删除（`deleteById`）— 重点

- **实现**：
  1. `studentMapper.countByClazzId(id)` 统计该班级学生数量；
  2. 若 `> 0` 抛 `BusinessException("该班级下有学生, 不能直接删除")`；
  3. 否则 `clazzMapper.deleteById(id)`；
  4. 方法加 `@Transactional(rollbackFor = Exception.class)`。
- **为什么**：`student.clazz_id` 外键指向班级，若不校验直接删班，学生数据会指向不存在的班级（脏数据）；加事务保证“校验+删除”要么都成功要么都回滚，异常时 `BusinessException` 被 `GlobalExceptionHandler` 捕获返回 `Result.error(...)`。

### 6. 查询所有班级（`listAll`）

- **实现**：`clazzMapper.listAll()` 返回 `List<Clazz>`，按 `update_time desc`。
- **为什么**：给学员管理界面做下拉框，数量少、无需分页。

## 三、容易出现的报错 & 解决

### 1. `BusinessException：该班级下有学生，不能直接删除`

- **原因**：班级下仍有学生，前端直接删班。
- **解决**：删除前校验 `countByClazzId`，用业务异常拦截并展示“不能直接删除”。这属于数据完整性保护，属预期行为。

### 2. 时间格式报错 `Failed to convert ... LocalDate`

- **原因**：前端传 `yyyy-MM-dd`，而字段是 `LocalDate`，默认无法自动转换。
- **解决**：在 `beginDate`/`endDate` 上加 `@DateTimeFormat(pattern = "yyyy-MM-dd")`；查询参数 `begin`/`end` 同样加该注解。

### 3. 查询结果 `status`、`masterName` 为 null

- **原因**：`status`、`masterName` 是非表字段，`list` 里 `select c.*` 未包含它们，或 `<resultType>` 映射依赖 `map-underscore-to-camel-case`。
- **解决**：`list` 中显式 `e.name as masterName` 与 `case ... end as status`；确保 `application.yml` 开启 `map-underscore-to-camel-case: true`。

### 4. 分页查询不带分页（返回全量）

- **原因**：`PageHelper.startPage` 之后又执行了别的查询，或分页参数 `page/pageSize` 未绑上。
- **解决**：`startPage` 后紧跟目标查询；`ClazzQueryParam` 设置默认值 `page=1、pageSize=10`；避免在 `page` 方法内部先跑其它 SQL。

### 5. `update` 把不该清的字段置空

- **原因**：`<set>` 里某字段的 `<if test="... != null">` 漏掉，或前端传了 null。
- **解决**：所有可空字段都加 `<if test="xx != null">`（字符串再加 `and xx != ''`），只更新非空字段。

## 四、关键 POJO 枚举说明

- `subject`：1:java、2:前端、3:大数据、4:Python、5:Go、6:嵌入式
- `status`：未开班（`begin_date > curdate()`）、已结课（`end_date < curdate()`）、已开班（其它）

## 五、相关类

- Controller：`com.cyy.tliaswebmanagement.controller.ClazzController`
- Service：`com.cyy.tliaswebmanagement.service.impl.ClazzServiceImpl`
- Mapper：`ClazzMapper` + `ClazzMapper.xml`
- POJO：`Clazz`、`ClazzQueryParam`、`ClazzOption`

# 学员管理 重点笔记（结合代码）

> 模块：学员管理（`StudentController` + `StudentServiceImpl` + `StudentMapper`）
> 基础路径：`/students`　|　统一返回：`Result`（`code=1` 成功，`code=0` 失败）

## 一、接口与实现概览

| 接口                | 方法              | 路径                                   | 核心逻辑              |
| ------------------- | ----------------- | -------------------------------------- | --------------------- |
| 查询所有班级（3.1） | `listAll`         | `GET /clazzs/list`                     | 班级下拉              |
| 条件分页查询（3.2） | `page`            | `GET /students`                        | PageHelper + 条件过滤 |
| 新增（3.3）         | `save`            | `POST /students`                       | 自动补时间字段        |
| 按ID查询（3.4）     | `get`             | `GET /students/{id}`                   | 主键查询              |
| 修改（3.5）         | `update`          | `PUT /students`                        | 动态 `<set>`          |
| 批量删除（3.6）     | `delete`          | `DELETE /students/{ids}`               | `foreach` in 批量     |
| 违纪处理（3.7）     | `updateViolation` | `PUT /students/violation/{id}/{score}` | 累加违纪次数/分数     |

## 二、怎么实现 & 为什么这么实现

### 1. 条件分页查询（`page`）

- **实现**：`PageHelper.startPage(page, pageSize)` → `studentMapper.list(param)` → 强转 `Page<Student>` → `new PageResult<>(p.getTotal(), p.getResult())`。
- **为什么**：与班级一致，返回 `total + rows` 供前端分页组件使用。
- **注意**：`StudentQueryParam` 的 `name/degree/clazzId` 为可选；当前 `StudentMapper.xml #list` 是 `SELECT * FROM student`，**尚未做条件过滤与联表**，所以 `name/degree/clazzId`/`clazzName` 实际不生效，需后续完善（见问题 5）。

### 2. 新增（`save`）

- **实现**：`setCreateTime/setUpdateTime(now)` → `studentMapper.add(student)`。
- **为什么**：统一后端生成时间；`clazz_id` 关联班级，`violation_count/violation_score` 默认由 DB 或前端决定。

### 3. 修改（`update`）

- **实现**：`setUpdateTime(now)` → `studentMapper.update(student)`；XML `<set>` + `<if>` 仅更新非空字段，含 `violation_count/violation_score`。
- **为什么**：支持部分字段更新，同时违纪接口可直接复用 `update` 写回违纪数据。

### 4. 批量删除（`delete`）

- **实现**：`@Transactional` → `studentMapper.deleteById(ids)`；XML 用 `<foreach>` 生成 `id in (?,?,?)`，`collection="ids"`。
- **为什么**：前端勾选多行删除，一次 SQL 删除，`foreach` 避免拼接字符串 SQL 注入；加事务避免部分删除成功、部分失败。

### 5. 违纪处理（`updateViolation`）— 重点

- **实现**：
  1. `getById(id)` 查学员；
  2. `violationCount` 为空则当作 0，`count + 1`；`violationScore` 为空则当作 0，`scoreSum + score`（注意 `score` 是本次扣分，路径参数，可为负？按业务应为正数扣分）；
  3. 写入 `violation_count`、`violation_score`，刷新 `update_time`；
  4. `studentMapper.update(student)`。
- **为什么**：采用**读-改-写**而不是直接 `update set violation_count = violation_count + 1`，是因为需要把两次字段都作为整体更新，且走了统一 `update`，逻辑简单。但非原子，高并发下会丢更新（见问题 6）。
- **类型注意**：`violationCount/violationScore` 是 `Short`，累加后需强转 `(short)`。

## 三、容易出现的报错 & 解决

### 1. `violation_count/violation_score` 报 null 指针

- **原因**：新增学员时未初始化，`updateViolation` 里直接 `student.getViolationCount()` 拿到 null。
- **解决**：`count = student.getViolationCount() == null ? 0 : student.getViolationCount()`。

### 2. 类型转换错误 `int cannot be converted to Short`

- **原因**：`count + 1` 结果是 `int`，字段是 `Short`。
- **解决**：`student.setViolationCount((short) (count + 1))`、`setViolationScore((short)(scoreSum + score))`。

### 3. 批量删除 `in` 语法错误 / 空列表

- **原因**：`ids` 为空时 `<foreach>` 生成 `in ()` 报错。
- **解决**：在 Service 判断 `ids` 非空再删除；或前端保证至少选一条。

### 4. `@PathVariable List<Integer> ids` 绑定失败

- **原因**：路径 `/students/1,2,3` 中逗号分隔，需框架自动拆分，部分环境不识别。
- **解决**：保证请求路径格式如 `/students/1,2,3`；Spring Boot 默认支持逗号分隔转 `List`。若不行，改用 `@RequestParam` `ids=1,2,3`。

### 5. `name/degree/clazzId` 查询条件不生效、`clazzName` 为 null

- **原因**：当前 `#list` 是 `SELECT * FROM student`，未拼接 `<where>`，也未 `left join clazz`。

- **解决**：参考 `ClazzMapper.xml`，把 `#list` 改为：

  ```xml
  select s.*, c.name as clazzName
  from student s left join clazz c on s.clazz_id = c.id
  <where>
    <if test="name != null and name != ''">and s.name like concat('%',#{name},'%')</if>
    <if test="degree != null">and s.degree = #{degree}</if>
    <if test="clazzId != null">and s.clazz_id = #{clazzId}</if>
  </where>
  order by s.update_time desc
  ```

### 6. 违纪累加并发丢更新（隐患）

- **原因**：`updateViolation` 是“查出来再改回去”，两个请求同时读取相同旧值，后写覆盖前写。

- **解决（可选）**：用 SQL 原子更新：

  ```xml
  update student set violation_count = violation_count + 1,
  violation_score = violation_score + #{score}, update_time = now() where id = #{id}
  ```

  或加乐观锁/分布式锁。

### 7. `update` 误把 `clazzId` 或违纪字段置空

- **原因**：`<set>` 中字段 `<if>` 漏写，或前端传入 null。
- **解决**：给所有可空字段都加 `<if test="xx != null">`（字符串加 `and xx != ''`）。

## 四、关键 POJO 枚举说明

- `gender`：1:男、2:女
- `degree`：1:初中、2:高中、3:大专、4:本科、5:硕士、6:博士
- `isCollege`：1:来自院校、0:否

## 五、相关类

- Controller：`com.cyy.tliaswebmanagement.controller.StudentController`
- Service：`com.cyy.tliaswebmanagement.service.impl.StudentServiceImpl`
- Mapper：`StudentMapper` + `StudentMapper.xml`
- POJO：`Student`、`StudentQueryParam`、`StudentOption`

# 学员/员工信息统计 重点笔记（结合代码）

> 模块：统计报表（`ReportController` + `EmpServiceImpl` / `StudentServiceImpl` / `ClazzServiceImpl` + 对应 Mapper）
> 基础路径：`/report`　|　统一返回：`Result`（`code=1` 成功，`code=0` 失败）

## 一、接口与实现概览

| 接口         | 方法                   | 路径                            | 用途   | 底层 Mapper                 |
| ------------ | ---------------------- | ------------------------------- | ------ | --------------------------- |
| 员工职位分布 | `getEmpJobData`        | `GET /report/empJobData`        | 柱状图 | `EmpMapper.countJob`        |
| 员工男女比例 | `getEmpGenderData`     | `GET /report/empGenderData`     | 饼图   | `EmpMapper.countGender`     |
| 学员学历分布 | `getStudentDegreeData` | `GET /report/studentDegreeData` | 柱状图 | `StudentMapper.countDegree` |
| 班级人数分布 | `getStudentCountData`  | `GET /report/studentCountData`  | 柱状图 | `ClazzMapper.countStudent`  |

## 二、怎么实现 & 为什么这么实现

### 1. 员工职位分布（`countJob`）

- **实现**：`EmpMapper.countJob()` 返回每项为 `{job, count}` 的 `List<Map>`；在 Service 里定义 `job 编号→名称` 的 Map（1班主任 2讲师 3学工主管 4教研主管 5咨询师），用 `toInteger` 把 `job`、`count` 转成 `Integer`，拆成 `jobList` 与 `dataList`，封装成 `JobOption`。
- **为什么**：数据库只存职位编号，前端图表要显示中文名称，所以在 Service 层做“编号→名称”翻译；拆成两个并列 List 便于前端直接渲染柱状图 X 轴与 Y 轴。

### 2. 员工男女比例（`countGender`）

- **实现**：`countGender()` 返回 `{gender, count}`；Service 用 `gender→名称` Map（1男 2女），组装成 `List<Map>`，每项 `{name, value}`。
- **为什么**：饼图通常要 `name/value` 结构，因此把性别编号翻译成中文并转为标准结构。

### 3. 学员学历分布（`countDegree`）

- **实现**：`StudentMapper.countDegree()` 用 SQL `case degree when 1 then '初中' ... else '其他' end as name, count(*) as value ... group by degree order by degree`，结果直接映射到 `StudentOption`（含 `name`、`value`）。
- **为什么**：学历编号到中文映射直接放到 SQL，更省事，且 `group by degree` 保证每种学历一行。

### 4. 班级人数分布（`countStudent`）

- **实现**：`ClazzMapper.countStudent()` 用 `left join student s on c.id = s.clazz_id` + `group by c.name`，`count(s.id)` 统计人数；`ClazzServiceImpl.countStudent()` 遍历 `List<Map>`，把 `name`/`count` 分别放入 `clazzList`/`dataList`，封装为 `ClazzOption`。
- **为什么**：用户希望统计“每个班级有多少学生”，用 `left join` 保留没有学生的班级（count=0），`group by c.name` 按班级聚合。

## 三、统计层面共用的细节

- **`toInteger`**：数据库 `count` 常为 `Long` 或 `BigInteger`，`name`/`gender`/`job` 可能为数字字符串，统一转 `Integer`，不支持 Number 时 `parseInt`，失败返回 `null`，避免 `ClassCastException`。
- **封装对象**：
  - `JobOption`：`jobList` + `dataList`
  - `StudentOption`：`name` + `value`
  - `ClazzOption`：`clazzList` + `dataList`

## 四、容易出现的报错 & 解决

### 1. `ClassCastException`：`job`/`count` 类型不一致

- **原因**：MyBatis 对 `count(*)` 返回 `Long`，不同数据库/驱动可能返回 `BigInteger`，强转 `Integer` 会抛异常。
- **解决**：统一用 `toInteger(Object)`：`Number` 则 `intValue()`，字符串则 `parseInt`，无法转换返回 `null`。

### 2. `Map.get("name")` 取到 null / 键名大小写不一致

- **原因**：SQL 别名 `as name`、`as count` 大小写或数据库返回大小写与代码 `get("name")` 不一致；或 `map-underscore-to-camel-case` 影响。
- **解决**：SQL 别名固定小写且与代码一致；规避使用下划线别名（不要用 `as total_count` 再用 `get("total_count")`，不同驱动键名不同），最稳妥的是在 SQL 里别名与代码 key 完全一致。

### 3. `group by` 非聚合列报错（`ONLY_FULL_GROUP_BY`）

- **原因**：MySQL `sql_mode=ONLY_FULL_GROUP_BY` 下，`select` 的列必须都在 `group by` 里或用聚合函数。
- **解决**：`countJob` 用 `group by job`、`countGender` 用 `group by gender`、`countDegree` 用 `group by degree`、`countStudent` 用 `group by c.name`，确保 select 的非聚合列都被 group by 覆盖。

### 4. 统计结果顺序不稳定

- **原因**：未加 `order by`，前端渲染可能乱序。
- **解决**：`countDegree` 加 `order by degree`，`countStudent` 加 `order by count desc`；`countJob`/`countGender` 可自行按需排序。

### 5. 班级人数统计漏掉 0 人班级

- **原因**：用 `inner join` 或只统计有学生的班级。
- **解决**：用 `left join`，即使某班级无学生也会出现（count=0）。

### 6. 前端图表标签为数字（如“1”“2”而非“班主任/男/本科”）

- **原因**：后端直接返回编号，未做翻译。
- **解决**：在 Service 层翻译，或 SQL `case` 映射；职位、性别在 Java 层翻译，学历在 SQL 层翻译。

## 五、相关类

- Controller：`com.cyy.tliaswebmanagement.controller.ReportController`
- Service：`EmpServiceImpl` / `StudentServiceImpl` / `ClazzServiceImpl`
- Mapper：`EmpMapper.xml` / `StudentMapper.xml` / `ClazzMapper.xml`
- POJO：`JobOption`、`StudentOption`、`ClazzOption`