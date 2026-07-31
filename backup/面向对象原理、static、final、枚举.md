# 面向对象原理

## JAVA中对象的内存分配

1. 加载`.class`文件
2. 声明局部变量
3. 堆中开辟空间（对象）
4. 默认初始化
5. 显式初始化
6. 构造方法初始化
7. 赋值地址值

内存模型

<img width="855" height="750" alt="Image" src="https://github.com/user-attachments/assets/87ff2406-4337-4388-a542-3f252929cfd5" />

- 方法出栈之后，方法里面的变量全部消失
- 如果没有任何地方使用堆中的对象，那么对象也会从堆里面消失
- 方法区里面的字节码信息一般不会消失，除非关闭虚拟机（idea）



## 对象在方法中进行传递

1. 把一个对象传递给方法，实际传递的是对象的内存地址
2. 当多个变量指向同一个对象的时候，只要有一个变量修改了对象中的属性，其他变量再次访问就是修改之后的结果了

```java
package PrincipleDemo1;

public class Memory
{
    static void main()
    {
        Student stu = new Student();
        stu.getInfo();
        Student stu1 = stu;
        stu1.setAge(199);
        stu1.getInfo();
    }
}
```

```cmd
"C:\JAVA tools\bin\java.exe" "-javaagent:C:\JAVA tools\IntelliJ IDEA 2026.2.0.1\lib\idea_rt.jar=55963" -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath C:\Users\72982\IdeaProjects\oopPrinciple\out\production\oopPrinciple PrincipleDemo1.Memory
无参构造方法被调用
姓名：null，年龄：0，性别：null，身高：0.0
姓名：null，年龄：199，性别：null，身高：0.0
```

## JAVA中`this`关键字详解

<img width="885" height="765" alt="Image" src="https://github.com/user-attachments/assets/d668eb0f-52de-4876-8ac1-aca9905b1f3d" />

**`this`的本质**：代表所在方法***调用者***的内存地址

## `static`关键字

### `static`关键字修饰成员变量

**`static`**:表示**静态**，是JAVA的修饰符，用来修饰（成员变量/成员方法）

**特点**：叫做静态变量，被该类所有对象共享

**调用方式**

1. 类名调用（推荐）
2. 对象名调用

```java
package AdvancedDemo1;

public class Student
{
    String name;
    int age;
    static String teacher ;

    public Student()
    {
    }

    public Student(String name, int age)
    {
        this.name = name;
        this.age = age;
    }
}
```

```java
package AdvancedDemo1;

public class Test
{
    static void main()
    {
        /*
        学生有如下属性:姓名，年龄，老师
        一个班级中，所有学生都是共享同一个老师
        第一名学生:小诗诗，19岁
        第二名学生:小丹丹，20岁
        最初都是小雯老师上课，有一天小丹丹申请换老师，换成了阿玮老师
        利用static模拟上述效果
        */
        Student student1 = new Student("小诗诗", 19);
        Student student2 = new Student("小丹丹", 20);
        // 初始化老师
        Student.teacher = "小雯老师";
        System.out.println(student1.name + "的老师是" + Student.teacher);
        System.out.println(student2.name + "的老师是" + Student.teacher);

        // 小丹丹申请换老师
        Student.teacher = "阿玮老师";
        System.out.println(student1.name + "的老师是" + Student.teacher);
        System.out.println(student2.name + "的老师是" + Student.teacher);
    }

}
```

```cmd
"C:\JAVA tools\bin\java.exe" "-javaagent:C:\JAVA tools\IntelliJ IDEA 2026.2.0.1\lib\idea_rt.jar=60344" -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath C:\Users\72982\IdeaProjects\oopAdvanced\out\production\oopAdvanced AdvancedDemo1.Test
小诗诗的老师是小雯老师
小丹丹的老师是小雯老师
小诗诗的老师是阿玮老师
小丹丹的老师是阿玮老师
```

#### `static`关键字的内存解析

**静态变量随着类的加载而加载，优先于对象出现**

<img width="1689" height="882" alt="Image" src="https://github.com/user-attachments/assets/9419b178-61a5-45e0-8303-e7af5eb35cfc" />

#### `static`静态变量小结

**特点**：

- 被`static`关键字修饰的变量叫做静态变量，被这个类所有对象共享
- 不属于某一个对象，属于整个类
- 随着类的加载而加载，优先于对象而存在

**调用方式**：

- 方式一：类名调用（推荐）
- 方式二：对象名调用（不推荐）

### `static`关键字修饰成员方法

**特点**:

- static修饰的方法叫做静态方法
- 该方法多用在**测试类**和**工具类**中
- Javabean类中很少会用

**调用方式**:

- 方式一:类名调用
- 方式二:对象名调用

#### 工具类

不是用来描述一类事物的，也没有`main`方法，而是帮我们做一些事情的类

- 类名见名知意
- 私有化构造方法
- 所有方法定义为静态，通过类名调用

```java
package AdvancedDemo2;

public class ArrayUtil
{
    /*
    * 1.定义一个方法,实现数组的反转
    * 2.定义一个方法,实现数组的复制
    * 3.定义一个方法,实现数组的查找(查找指定元素第一次出现的索引)
    * 4.定义一个方法,实现数组的排序(从小到大)
    * 5.定义一个方法,实现数组的输出
    * */

    private ArrayUtil()
    {
    }

    public static void swap(int[] arr, int i, int j)
    {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
    public static void reverse(int[] arr)
    {
        for (int i = 0; i < arr.length / 2; i++)
        {
            swap(arr, i, arr.length - 1 - i);
        }
    }
    public static int[] copy(int[] arr)
    {
        int[] newArr = new int[arr.length];
        for (int i = 0; i < arr.length; i++)
        {
            newArr[i] = arr[i];
        }
        return newArr;
    }
    public static int indexOf(int[] arr, int value)
    {
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] == value) {
                return i;
            }
        }
        return -1;
    }

    // 冒泡排序
    public static void sort(int[] arr)
    {
        for (int i = 0; i < arr.length - 1; i++)
        {
            for (int j = 0; j < arr.length - 1 - i; j++)
            {
                if (arr[j] > arr[j + 1])
                {
                    swap(arr, j, j + 1);
                }
            }
        }
    }
    public static void printArray(int[] arr)
    {
        System.out.print("[");
        for (int i = 0; i < arr.length; i++)
        {
            System.out.print(arr[i]);
            if (i < arr.length - 1)
            {
                System.out.print(", ");
            }
        }
        System.out.println("]");
    }
}
```

```java
package AdvancedDemo2;

public class Test
{
    static void main()
    {
        int[] arr = {1, 2, 6, 4, 5};
        System.out.println("原数组:");
        ArrayUtil.printArray(arr);

        ArrayUtil.reverse(arr);
        System.out.println("反转后的数组:");
        ArrayUtil.printArray(arr);
        
        ArrayUtil.sort(arr);
        System.out.println("排序后的数组:");
        ArrayUtil.printArray(arr);
        
        int[] newArr = ArrayUtil.copy(arr);
        System.out.println("复制的数组:");
        ArrayUtil.printArray(newArr);
        
        int index = ArrayUtil.indexOf(arr, 4);
        System.out.println("4在数组中的索引是:" + index);
    }
}
```

### 静态的注意事项

- 静态方法只能访问**静态变量**和其他的**静态方法**
- 非静态方法可以访问**静态变量**或者静态方法，也可以访问**非静态的成员变量**和非静态的成员方法
- 静态方法中没有`this`关键字
- **总结**：静态只能调用静态，非静态可以调用所有，静态方法无this

#### `main`方法解析

- `public`:被JVM调用，访问权限最大
- `static`:被JVM调用，类名访问，测试类中其他方法也是静态的
- `void`:不需要给」VM返回值
- `main`:固定的名字，被VM识别
- `String[] args`:用于接收运行的参数，目前不用

## `final`关键字

`final`：**表示最终，不可变。**可以修饰变量、类、方法

`final`修饰变量，此时叫做常量

- 特点1:只能被赋值一次，一旦赋值，无法再次修改。
- 特点2:常量名大写，多个单词之间用下划线隔

细节:

- **基本数据类型**:

  `byte short int long float double char boolean`

​	变量里面记录的是真实的数据

​	`fianl int a = 10;`

- **引用数据类型**:

​	除了上面四类八种，其他所有的数据类型都是引用类型
​		`int[]Student Teacher...`

​	`stu`中记录对象的内存地址，不可改变的是`stu`记录的内存地址

​	而对象里面的属性值，是可以发生改变的

​	`final Student stu = new Student();`

- **综上所述**：

​	`final`修饰那个变量，这个变量里面记录的内容就无法再次发生改变（属性值/地址）

```java
package FinalTest;

public class Student
{
    final String name;
    final int age;

    public Student(String name, int age)
    {
        this.name = name;
        this.age = age;
    }

    public String getName()
    {
        return name;
    }

    public int getAge()
    {
        return age;
    }
}
```

```java
package FinalTest;

public class Test
{
    static void main()
    {
        final int NUMBER = 100;
        System.out.println("NUMBER = " + NUMBER);
        final Student stu = new Student("Alice", 20);
        // stu.name = "Berry";
        System.out.println(stu.name);
        System.out.println(stu.age);
    }
}
```

## 枚举

- 枚举是一个特殊的`Javabean`类，这个类的对象是有限个
- 使用场景：订单的状态、月份、星期、游戏角色职业、会议室预约状态、设备状态..

### 枚举的定义格式

```java
public enum 枚举类名{
    枚举项1,枚举项2,枚举项3;
    属性;
    行为;
}
```

```java
package EnumTest;

public enum OrderState
{

    // 电商项目中，订单的状态只有以下6种，请编写代码实现。
    // 待支付  PAYMENT_PENDING
    // 处理中  PROCESSING
    // 己发货  SHIPPEDDELIVERED
    // 配送中  OUT FOR _DELIVERY
    // 己送达  DILIVERED
    // 己取消  CANCELLED

    // 列出所有对象
    PAYMENT_PENDING("待支付"),
    PROCESSING("处理中"),
    SHIPPED("己发货"),
    OUT_FOR_DELIVERY("配送中"),
    DELIVERED("己送达"),
    CANCELLED("己取消");
    final private String name;

    OrderState(String name)
    {
        this.name = name;
    }

    public String getName()
    {
        return name;
    }
}
```

```java
package EnumTest;

public class Test
{
    static void main()
    {
        System.out.println(OrderState.PAYMENT_PENDING.getName());
        System.out.println(OrderState.PROCESSING.getName());
        System.out.println(OrderState.SHIPPED.getName());
        System.out.println(OrderState.OUT_FOR_DELIVERY.getName());
        System.out.println(OrderState.DELIVERED.getName());
        System.out.println(OrderState.CANCELLED.getName());
        OrderState o1 = OrderState.PAYMENT_PENDING;
        switch(o1)
        {
            case PAYMENT_PENDING-> System.out.println("订单状态为:待支付");
            case PROCESSING -> System.out.println("订单状态为:处理中");
            case SHIPPED -> System.out.println("订单状态为:己发货");
            case OUT_FOR_DELIVERY -> System.out.println("订单状态为:配送中");
            case DELIVERED -> System.out.println("订单状态为:己送达");
            case CANCELLED -> System.out.println("订单状态为:己取消");
        }
    }
}
```

所有的枚举项，默认使用`public static final`修饰

### 枚举的注意事项

- 每一个枚举项，都是该枚举类的对象
- 枚举项在底层其实就是常量，默认用`public static final`修饰
- 枚举类的第一行上必须是枚举项，每个枚举项之间用逗号隔开，以分号作为结尾
- 枚举类的构造方法必须是`private`修饰，不让外界创建本类的对象
- 编译器会给枚举类新增两个默认存在的方法：`values(),valueOf()`

- `values()`:表示获取本类所有的枚举项
- `valueOf()`：表示获取一个指定的枚举项	

- 枚举类的构造方法默认使用`private`修饰

**默认用`public static final`修饰**

```JAVA
PS C:\Users\72982\IdeaProjects\oopAdvanced\out\production\oopAdvanced\EnumTest> javap .\OrderState.class
Compiled from "OrderState.java"
public final class EnumTest.OrderState extends java.lang.Enum<EnumTest.OrderState> {
  public static final EnumTest.OrderState PAYMENT_PENDING;
  public static final EnumTest.OrderState PROCESSING;
  public static final EnumTest.OrderState SHIPPED;
  public static final EnumTest.OrderState OUT_FOR_DELIVERY;
  public static final EnumTest.OrderState DELIVERED;
  public static final EnumTest.OrderState CANCELLED;
  public static final EnumTest.OrderState aa;
  public static EnumTest.OrderState[] values();
  public static EnumTest.OrderState valueOf(java.lang.String);
  public java.lang.String getName();
  static {};
}
```

**`values()、valueOf()`**:

```java
package EnumTest;

public class Test2
{
    static void main()
    {
        OrderState[] arr = OrderState.values();
        for(OrderState j:arr)
        {
            System.out.println(j);
        }

        System.out.println("--------------------------");
        System.out.println(OrderState.valueOf("PAYMENT_PENDING"));
    }
}
```

```cmd
"C:\JAVA tools\bin\java.exe" "-javaagent:C:\JAVA tools\IntelliJ IDEA 2026.2.0.1\lib\idea_rt.jar=56915" -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath C:\Users\72982\IdeaProjects\oopAdvanced\out\production\oopAdvanced EnumTest.Test2
PAYMENT_PENDING
PROCESSING
SHIPPED
OUT_FOR_DELIVERY
DELIVERED
CANCELLED
aa
--------------------------
PAYMENT_PENDING
```

