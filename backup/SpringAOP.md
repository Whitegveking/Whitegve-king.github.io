# Spring AOP

**AOP**：Aspect Oriented Programming(**面向切面编程、面向方面编程)**，可简单理解为就是面向特定方法编程

场景：案例中部分业务方法运行较慢，定位执行耗时较长的接口，此时需要统计每一个业务方法的执行耗时

优势：

1. 减少重复代码
2. 代码无侵入
3. 提高开发效率
4. 维护方便

> [!NOTE]
>
> AOP是一种思想，而在Spring框架中对这种思想进行的实现，那我们要学习的就是Spring AOP



## AOP基础

### AOP快速入门

**统计所有控制层方法的执行耗时**

```java
package com.cyy.tliaswebmanagement.aspect;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

import java.util.Arrays;

/**
 * 操作日志切面（AOP）
 *
 * <p>拦截所有 Controller 方法，记录方法名、请求参数、执行耗时。
 * 这里用 {@code @Around} 环绕通知，能在方法执行前后插入逻辑。</p>
 *
 * <p>作用：把“日志记录”“耗时统计”“权限校验”这类与业务无关的横切逻辑，
 * 从业务代码中剥离出来，避免散落在每个接口里。</p>
 */
@Slf4j
@Aspect
@Component
public class OperationLogAspect {

    /**
     * 切点：所有 com.cyy.tliaswebmanagement.controller 包下的任意方法
     */
    @Pointcut("execution(* com.cyy.tliaswebmanagement.controller.*.*(..))")
    public void controllerPointcut() {
    }

    /**
     * 环绕通知：统计方法执行耗时并记录入参
     *
     * @param pjp 连接点
     * @return 方法返回值
     * @throws Throwable 异常透传
     */
    @Around("controllerPointcut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        String method = pjp.getSignature().toShortString();
        Object[] args = pjp.getArgs();
        log.info("[AOP] 进入 {}，参数 = {}", method, Arrays.toString(args));

        Object result;
        try {
            result = pjp.proceed();
        } catch (Throwable e) {
            log.info("[AOP] {} 执行异常：{}", method, e.getMessage(),e);
            throw e;
        }

        long cost = System.currentTimeMillis() - start;
        log.info("[AOP] 退出 {}，耗时 = {}ms，返回 = {}", method, cost, result);
        return result;
    }
}

```



### AOP核心概念

- **连接点**:JoinPoint，可以被AOP控制的方法(暗含方法执行时的相关信息)
- **通知**:Advice，指那些重复的逻辑，也就是共性功能(最终体现为一个方法)
- **切入点**:PointCut，匹配连接点的条件，通知仅会在切入点方法执行时被应用
- **切面**:Aspect，描述通知与切入点的对应关系(通知+切入点)
- **目标对象**:Target，通知所应用的对象

### AOP调用链

```java
浏览器
  │  GET /emps/1
  ▼
Tomcat
  │
  ▼
DispatcherServlet (Spring MVC 前端控制器)
  │  1. 寻找能处理 /emps/1 的 Handler，找到 EmpController#getById
  │  2. 但 Spring 容器里存的其实是 代理对象(AOP 代理)，不是原始 EmpController
  ▼
proxy.getById(1)          ← 外部调用的入口，代理对象的方法
  │  3. 代理拦截到调用，命中切点
  ▼
[目标方法没执行] → OperationLogAspect.around(pjp)
  │  4. 记录开始时间、方法名、参数
  ▼
pjp.proceed()             ← 关键：穿透到原始对象，真正执行
  │
  ▼
realEmpController.getById(1)          ← 真正的原始方法
  │  5. 执行业务逻辑：empServiceImpl.getById(1)
  │  6. 返回  Result.success(emp)
  ▼
around() 得到 result = Result{code:1, msg:success, data:{Emp...}}
  │  7. 计算耗时，打日志
  │  8. return result;  ← 原样交还
  ▼
代理对象把 result 返回给 DispatcherServlet
  │
  ▼
DispatcherServlet 把 Result 序列化成 JSON 写回响应体
  │
  ▼
浏览器收到 {"code":1,"msg":"success","data":{...}}
```

### 通知类型

| 通知注解          | 类型       | 执行时机                     | 异常时是否执行                 | 说明                                                   |
| ----------------- | ---------- | ---------------------------- | ------------------------------ | ------------------------------------------------------ |
| `@Around`         | 环绕通知   | 目标方法**前**和**后**都执行 | 由代码决定（可包裹 try/catch） | 最灵活，最强大，能控制目标方法是否执行、能否替换返回值 |
| `@Before`         | 前置通知   | 目标方法**前**执行           | 不执行（在被拦截方法之前）     | 常用于校验、日志、权限                                 |
| `@After`          | 后置通知   | 目标方法**后**执行           | **无论有无异常都会执行**       | 常用于释放资源、清理                                   |
| `@AfterReturning` | 返回后通知 | 目标方法**正常返回后**执行   | **有异常不会执行**             | 常用于处理后正常返回值                                 |
| `@AfterThrowing`  | 异常后通知 | 目标方法**抛出异常后**执行   | 仅**有异常时**执行             | 常用于异常记录、告警                                   |

> [!CAUTION]
>
> 1. @Around环绕通知需要自己调用 ProceedingJoinPoint.proceed()来让原始方法执行，其他通知不需要考虑目标方法执行
> 2. @Around环绕通知方法的返回值，必须指定为object，来接收原始方法的返回值。

```java
package com.cyy.tliaswebmanagement.aspect;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Slf4j
@Aspect
@Component
public class MyAspect1 {
    @Pointcut("execution(* com.cyy.tliaswebmanagement.controller..*(..))")
    public void controllerPointcut() {
    }

    @Before("controllerPointcut()")
    public void before() {
        log.info("[MyAspect1] Before advice executed");
    }

    @Around("controllerPointcut()")
    public Object around(org.aspectj.lang.ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[MyAspect1] Around advice executed");
        Object result = joinPoint.proceed();
        log.info("[MyAspect1] Around advice result: {}", result);
        return result;
    }

    // 无论是否出现异常都会执行
    @After("controllerPointcut()")
    public void after() {
        log.info("[MyAspect1] After advice executed");
    }
    
    // 出现异常不会运行
    @AfterReturning("controllerPointcut()")
    public void afterReturning() {
        log.info("[MyAspect1] AfterReturning advice executed");
    }
    
    // 只有出现异常才会运行
    @AfterThrowing("controllerPointcut()")
    public void afterThrowing() {
        log.info("[MyAspect1] AfterThrowing advice executed");
    }
}
```

**正常返回时**

```java
@Around 前半部分
    ↓
@Before
    ↓
目标方法执行
    ↓
@AfterReturning
    ↓
@After
    ↓
@Around 后半部分
```

**抛异常时**

```java
// @Around 前半
log.info("[AOP] 进入 EmpController.getById(..)，参数 = [1]");
//   ↓
// @Before
log.info("[@Before] 方法执行前");
//   ↓
// 目标方法抛出 NPE
//   ↓
// @AfterThrowing
log.info("[@AfterThrowing] 抛出异常：NullPointerException");
//   ↓
// @After
log.info("[@After] 方法执行完毕");
//   ↓
// 异常穿透 @Around，向外抛出
```

**环绕通知解析**

环绕通知是"包住目标方法"的那一层，它里面的 `pjp.proceed()` 才是真正去触发"目标方法 + 其他通知"的地方

```java
@Around 前半
    ↓
   [ pjp.proceed() 内部 ]
         ↓
      @Before
         ↓
      目标方法
         ↓
      @AfterReturning / @AfterThrowing
         ↓
      @After
         ↓
   [ proceed() 返回 ]
    ↓
@Around 后半
```

| 场景 | 正常返回顺序      | 抛异常顺序               |
| ---- | ----------------- | ------------------------ |
| 第 1 | `@Around` 前半    | `@Around` 前半           |
| 第 2 | `@Before`         | `@Before`                |
| 第 3 | 目标方法          | 目标方法（抛异常）       |
| 第 4 | `@AfterReturning` | `@AfterThrowing`         |
| 第 5 | `@After`          | `@After`                 |
| 第 6 | `@Around` 后半    | 异常向外抛（若无 catch） |

### 通知顺序

当有多个切面的切入点都匹配到了目标方法，目标方法运行时，**多个通知方法都会被执行**。

**执行顺序：**

不同切面类中，默认按照切面类的**类名字母排序**：

- 目标方法前的通知方法：**字母排名靠前的先执行**
-  目标方法后的通知方法：**字母排名靠前的后执行**

用`@Order(数字)`加在切面类上来控制顺序

- 目标方法前的通知方法:数字小的先执行
- 目标方法后的通知方法:数字小的后执行

### 切入点表达式

**介绍**：描述切入点方法的一种表达式。

**作用**：用来决定项目中的哪些方法需要加入通知

**常见形式**：

1. execution(......)`：根据方法的签名来匹配
2. annotation(......)：根据注解匹配

#### @execution

> ● `execution` 主要根据方法的返回值、包名、类名、方法名、方法参数等信息来匹配，语法为：
>
> ```
> execution(访问修饰符?   返回值   包名.类名.?方法名(方法参数)   throws 异常?)
> ```

> ● 其中带 `?` 的表示可以省略的部分
>
> 1. 访问修饰符：可省略（比如：`public`、`protected`）
> 2. 包名.类名：可省略
> 3. `throws` 异常：可省略（注意是方法上声明抛出的异常，不是实际抛出的异常）

**解读**

这段讲的是 **`execution()` 切入点表达式的完整语法**，核心是：

```java
execution(
    访问修饰符?      // 可选，如 public / private / protected，不写则匹配任意
    返回值类型        // 必填，如 void / Result / *
    包名.类名.?方法名(参数类型列表)   // 方法定位部分，用 . 分隔，? 表示可省略
    throws 异常?     // 可选
)
```

**对你那个 `@Before` 例子逐段拆解**

```java
execution(public void com.itheima.service.impl.DeptServiceImpl.delete(java.lang.Integer))
```

| 片段                                       | 含义                           |
| ------------------------------------------ | ------------------------------ |
| `public`                                   | 访问修饰符，限定为 public 方法 |
| `void`                                     | 返回值类型，必须是 void        |
| `com.itheima.service.impl.DeptServiceImpl` | 精确到 DeptServiceImpl 这个类  |
| `.delete`                                  | 方法名必须是 delete            |
| `(java.lang.Integer)`                      | 参数必须恰好是一个 `Integer`   |
| `throws`                                   | 未写，表示任意异常都可以       |

**三种"可省略"的具体效果**

1. **省略访问修饰符**：`execution(void com.itheima...delete(...))` → 匹配任意访问权限的 `delete` 方法。
2. **省略包名.类名**：只写方法名和参数，范围会放大，比如 `execution(void delete(java.lang.Integer))`。
3. **省略 throws**：无影响，因为 `throws` 一般很少用，Spring 关键按方法签名匹配，声明异常通常不作为重点。

> [!CAUTION]
> 注意：图片特别强调 `throws 异常` 指的是**代码里声明抛出**的异常（`throws IOException`），而不是方法运行**实际抛出**的异常。

**可以使用通配符描述切入点**

1. `*`：单个独立的任意符号，可以通配任意返回值、包名、类名、方法名、任意类型的一个参数，也可以通配包、类、方法名的一部分
   `execution(* com.*.service.*.update*(*))`
2. `..`：多个连续的任意符号，可以通配任意层级的包，或任意类型、任意个数的参数
   `execution(* com.itheima.DeptService.*(..))`

#### @annotation

用于匹配标识有特定注解的方法，不太建议使用

### 连接点

在Spring中用JoinPoint抽象了连接点，用它可以获得方法执行时的相关信息，如目标类名、方法名、方法参数等。

- 对于 @Around 通知，获取连接点信息只能使用 ProceedingJoinPoint
-  对于其它四种通知，获取连接点信息只能使用 JoinPoint，它是ProceedingJoinPoint 的父类型

### 通过AOP记录日志

通过AOP记录接口的如下信息：

```java
源IP
源端口
URI
请求报文
请求时间
响应报文
响应时间
接口耗时(ms)
```

**pojo类**

```java
package com.cyy.tliaswebmanagement.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class RequestLog {
    /** 主键 */
    private Long id;

    /** 源IP */
    private String clientIp;

    /** 源端口 */
    private Integer clientPort;

    /** 请求URI */
    private String uri;

    /** 请求方式 GET/POST/PUT/DELETE */
    private String httpMethod;

    /** Controller类名.方法名 */
    private String classMethod;

    /** 方法参数（Controller 方法入参序列化） */
    private String methodParams;

    /** 请求报文 */
    private String requestBody;

    /** 请求查询参数（URL 上的 query string） */
    private String queryString;

    /** 请求时间 */
    private LocalDateTime requestTime;

    /** 响应报文 */
    private String responseBody;

    /** 响应状态（HTTP 状态码或业务结果 code） */
    private Integer status;

    /** 异常信息 */
    private String errorMsg;

    /** 响应时间 */
    private LocalDateTime responseTime;

    /** 接口耗时(毫秒) */
    private Long costMs;
}

```

**数据库建表**

```mysql
CREATE TABLE `request_log` (
    `id`              BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',
    `client_ip`       VARCHAR(64)     NOT NULL DEFAULT '' COMMENT '源IP',
    `client_port`     INT             NOT NULL DEFAULT 0  COMMENT '源端口',
    `uri`             VARCHAR(255)    NOT NULL DEFAULT '' COMMENT '请求URI',
    `http_method`     VARCHAR(16)     NOT NULL DEFAULT '' COMMENT 'GET/POST/PUT/DELETE',
    `class_method`    VARCHAR(255)    NOT NULL DEFAULT '' COMMENT 'Controller类名.方法名',
    `request_body`    TEXT            NULL COMMENT '请求报文',
    `request_time`    DATETIME(3)     NOT NULL COMMENT '请求时间',
    `response_body`   TEXT            NULL COMMENT '响应报文',
    `response_time`   DATETIME(3)     NOT NULL COMMENT '响应时间',
    `cost_ms`         BIGINT          NOT NULL DEFAULT 0  COMMENT '接口耗时(毫秒)',
    PRIMARY KEY (`id`),
    KEY `idx_request_time` (`request_time`),
    KEY `idx_client_ip`    (`client_ip`),
    KEY `idx_uri`          (`uri`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_general_ci
  COMMENT = '接口请求日志表';
```

**Aspect**

```java
package com.cyy.tliaswebmanagement.aspect;

import com.cyy.tliaswebmanagement.anno.OperationLog;
import com.cyy.tliaswebmanagement.mapper.RequestLogMapper;
import com.cyy.tliaswebmanagement.pojo.RequestLog;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;
import java.util.Arrays;

/**
 * 接口请求日志切面（AOP）。
 *
 * <p>通过 {@code @Around} 环绕通知拦截所有标注了 {@link OperationLog} 的 Controller,
 * 在方法执行前后采集源IP、源端口、请求URI、请求/响应报文、请求/响应时间以及接口耗时,
 * 并封装成 {@link com.cyy.tliaswebmanagement.pojo.RequestLog RequestLog} 对象写入数据库。</p>
 *
 * <p>作用：把“接口调用日志采集”这类与业务无关的横切逻辑从 Controller 中剥离,
 * 避免在每个接口里重复编写同样的采集与落库代码。</p>
 */

@Slf4j
@Aspect
@Component
public class OperationLogAspect {

    @Autowired
    RequestLogMapper requestLogMapper;

    /**
     * 切点：除DemoUploadController外的所有 com.cyy.tliaswebmanagement.controller 包下的任意方法
     */
    @Pointcut("@within(com.cyy.tliaswebmanagement.anno.OperationLog)")
    public void controllerPointcut() {
    }

    /**
     * 环绕通知：统计方法执行耗时并记录接口请求日志
     *
     * @param pjp 连接点
     * @return 方法返回值
     * @throws Throwable 异常透传
     */
    @Around("controllerPointcut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        // 1. 拿 HTTP 请求对象
        ServletRequestAttributes attrs = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = null;
        if (attrs != null) {
            request = attrs.getRequest();
        }
        // 2. 记录开始时间 start
        long start = System.currentTimeMillis();
        // 3. 采集请求侧数据（IP/端口/URI/方法/参数/请求时间）
        String ip = null; // 源ip
        Integer port = null; // 源端口
        String uri = null;  // 请求URI
        String httpMethod = null;   // 请求方式 GET/POST/PUT/DELETE
        if (request != null) {
            ip = request.getRemoteAddr();
            port = request.getRemotePort();
            uri = request.getRequestURI();
            httpMethod = request.getMethod();
        }
        String classMethod = pjp.getSignature().getDeclaringTypeName() + "." + pjp.getSignature().getName();    // Controller类名.方法名
        String methodParams = Arrays.toString(pjp.getArgs());   // 方法参数（Controller 方法入参序列化）
        String queryString = null;  // 请求查询参数（URL 上的 query string）
        if (request != null) {
            queryString = request.getQueryString();
        }
        LocalDateTime requestTime = LocalDateTime.now().truncatedTo(ChronoUnit.MILLIS);    // 请求时间（毫秒）

        // 4. 执行目标方法，并采集响应侧数据（用 try/catch/finally 区分成功/异常）
        String responseBody = null; // 响应报文
        Integer status = null;  // 响应状态（HTTP 状态码：200 成功 / 500 异常）
        String errorMsg = null; // 异常信息
        Object result;
        try {
            result = pjp.proceed();
            if (result != null) {
                responseBody = result.toString();
            }
            status = 200;
        } catch (Throwable e) {
            errorMsg = e.getMessage();
            status = 500;
            throw e;
        } finally {
            // 5. 无论成功还是异常，都计算响应时间与耗时并落库
            LocalDateTime responseTime = LocalDateTime.now().truncatedTo(ChronoUnit.MILLIS);  // 响应时间（毫秒）
            long costMs = System.currentTimeMillis() - start;  // 接口耗时(毫秒)

            // 6. 组装 RequestLog 并调用 requestLogMapper.insertRequestLog(requestLog) 落库
            RequestLog requestLog = new RequestLog();
            requestLog.setClientIp(ip);
            requestLog.setClientPort(port);
            requestLog.setUri(uri);
            requestLog.setHttpMethod(httpMethod);
            requestLog.setClassMethod(classMethod);
            requestLog.setMethodParams(methodParams);
            // requestLog.setRequestBody();
            requestLog.setQueryString(queryString);
            requestLog.setRequestTime(requestTime);
            requestLog.setResponseBody(responseBody);
            requestLog.setStatus(status);
            requestLog.setErrorMsg(errorMsg);
            requestLog.setResponseTime(responseTime);
            requestLog.setCostMs(costMs);
            requestLogMapper.insertRequestLog(requestLog);
        }
        return result;
    }
}

```

