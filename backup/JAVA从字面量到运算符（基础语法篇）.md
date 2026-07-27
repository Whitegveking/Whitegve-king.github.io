# 0、字面量

表示数据在程序中怎么写。

**字面量**：程序中的数据。

| 字面量类型 |        说明        |    举例    |
| :--------: | :----------------: | :--------: |
|  整数类型  |       直接写       |  18，-88   |
|  小数类型  | 直接写，加上小数点 |    26.7    |
| 字符串类型 |         ""         |   "上班"   |
|  字符类型  |         ''         |    '男'    |
|  布尔类型  |  布尔值，表示真假  | true,false |
|   空类型   |   一个特殊的空值   |    null    |

<img width="411" height="288" alt="Image" src="https://github.com/user-attachments/assets/e06bf28e-3b11-47d8-bd45-04515d10ea69" />

# 1、变量

变量是存储数据的小空间，而不是里面的数据，是将数据放入变量中进行处理。

经常发生改变的数据。

## 变量的定义格式

**数据类型 变量名 = 数据值**

**例：int a = 10**

数据类型：为空间中存储的数据加入类型***限制***

## 变量的注意事项	

一个变量只能存一个值	

变量名不允许重复定义

变量在使用之前一定要进行赋值

一条语句可以定义多个变量，也可以连续赋值（尽量不使用）

<img width="1284" height="609" alt="Image" src="https://github.com/user-attachments/assets/1c57be8c-fb3a-4057-90e4-3dcaa51ec318" />

# 2、计算机的存储规则

在计算机中，任意数据都是以二进制的方式存储的

## 计算机的存储单元

1Byte = 8bit 

1字节 = 8比特

（比较基础直接跳过）

# 3、JAVA中的基本数据类型

## 整数

<img width="1137" height="360" alt="Image" src="https://github.com/user-attachments/assets/cd9fe271-72ed-4135-be17-06309bbc14ad" />

根据数据的取值范围选择不同的数据类型

## 小数

<img width="1065" height="318" alt="Image" src="https://github.com/user-attachments/assets/30335fba-24da-43db-90f5-073450750125" />

```java
package literal;

public class LiteralDemo2
{
    public static void main(String[] args)
    {
        // 定义8种数据类型的变量:
        // 整数类型:byte、short、int、long浮点数类型:float、double字符类型:char
        // 布尔类型:boolean
        // 变量的定义格式:
        // 数据类型变量名=数据值;
        byte b = 1;
        System.out.println(b);

        short s = 2;
        System.out.println(s);

        int i = 3;
        System.out.println(i);

        long l = 4L; // long类型的变量值后面要加L
        System.out.println(l);

        float f = 5.0f; // float类型的变量值后面要加f
        System.out.println(f);

        double d = 6.0;
        System.out.println(d);

        char c = 'A';
        System.out.println(c);
        
        boolean bool = true;
        System.out.println(bool);


    }
}

```

# 4、标识符

## 标识符的命名规则

标识符：代码中所有我们自己起的名字

**要求：**

1. 由数字、字母、下划线_、美元符$组成
2. 不能以数字开头
3. 不能是关键字
4. 区分大小写
5. 见名知意
6. 驼峰命名法

### 驼峰命名法

<img width="1218" height="507" alt="Image" src="https://github.com/user-attachments/assets/3db30c9e-49e7-4143-a806-12aa6792f559" />

# 5、键盘录入

**重点是`Scanner`类**

```java
package literal;

import java.util.Scanner;

public class LiteralDemo4
{
    static void main()
    {
        // 创建Scanner对象,用于接收键盘输入
        Scanner sc = new Scanner(System.in);
        // Scanner可以用于接收各种类型的数据,包括整数、浮点数、字符串等
        // 1.接收整数
        System.out.println("请输入一个整数:");
        int i = sc.nextInt();
        System.out.println("您输入的整数是:" + i);

        // 2.接收浮点数
        System.out.println("请输入一个浮点数:");
        double d = sc.nextDouble();
        System.out.println("您输入的浮点数是:" + d);
        
        // 3.接收字符串
        System.out.println("请输入一个字符串:");
        String str = sc.next();
        System.out.println("您输入的字符串是:" + str);
    }
}

```

# 运算符

## 1、算术运算符

1. 自增自减运算符
2. 赋值运算符
3. 关系运算符
4. 逻辑运算符
5. 三元运算符

```java
package literal;

public class LiteralDemo5
{
    static void main()
    {
        // 1、整数计算
        // 整数相除时,结果只保留整数部分,小数部分会被舍弃
        int a = 10;
        int b = 3;
        System.out.println(a + b); // 13
        System.out.println(a - b); // 7
        System.out.println(a * b); // 30
        System.out.println(a / b); // 3
        System.out.println(a % b); // 1

        // 2、浮点数计算
        // 浮点数计算时,结果可能会有精度损失
        double c = 10.0;
        double d = 3.0;
        System.out.println(c + d); // 13.0
        System.out.println(c - d); // 7.0
        System.out.println(c * d); // 30.0
        System.out.println(c / d); // 3.3333333333333335
        System.out.println(c % d); // 1.0
    }
}
```

## 2、类型转换

数字运算：类型不一样不能运算，需要转换成相同类型才能计算。

1、**把取值范围小的转换为取值范围大的（隐式转换）**

不同类型的数据进行计算，默认采取隐式转换，Java自动转换，无需我们写代码

<img width="969" height="180" alt="Image" src="https://github.com/user-attachments/assets/86597ab2-eb28-4542-9639-99344dac25e9" />

2、**强制转换，把取值范围大的转换为取值范围小的**

去掉不要的，有可能导致精度丢失

<img width="1014" height="203" alt="Image" src="https://github.com/user-attachments/assets/5a7daab5-6f63-4c18-8740-889c308495f9" />

强制转换与C语言类似

```java
package literal;

public class LiteralDemo6
{
    static void main()
    {
        // 强制类型转换
        byte a = 10;
        byte b = 20;
        // byte、short、char类型的变量在进行运算时,
        // 会先转换为int类型,再进行计算,所以需要强制类型转换
        byte c = (byte) (a + b);
        System.out.println(c);
        // 2、强制类型转换可能会导致精度损失
        short s1 = 100;
        short s2 = 200;
        byte d = (byte) (s1 + s2);
        System.out.println(d); // 44
    }
}

```

## 3、字符运算

例如字母的大小写转换，通过`ASCII`码表进行

```java
package Operator;

public class OperatorDemo1
{
    static void main()
    {
        char A = 'A';
        char a = (char) (A + 32);
        System.out.println(a);
    }
}
```

## 4、字符串运算

字符串只有**`+`**操作，没有其他操作

**任意数据`+`字符串**都是拼接操作，并产生一个新的字符串

<img width="375" height="156" alt="Image" src="https://github.com/user-attachments/assets/1f514ba4-4292-4f2c-bd0a-3c68f5431d3a" />

记住计算的顺序是从左到右，10和8先进行相加运算再与"岁"进行字符串运算

运算期间的数据类型会发生改变

```java
package Operator;

public class OperatorDemo2
{
    static void main()
    {
        // 我要知道7242秒是几时几分几秒
        int time = 7242;
        int hour = time / 3600;
        int minute = (time % 3600) / 60;
        int second = time % 60;
        System.out.println(hour + "时" + minute + "分" + second + "秒");
    }
}

```

## 5、自增自减运算符

<img width="390" height="95" alt="Image" src="https://github.com/user-attachments/assets/2f4c522e-0f07-42fd-8fec-30f4a696b7aa" />

与C++类似，跳过。

## 6、赋值运算符

<img width="993" height="405" alt="Image" src="https://github.com/user-attachments/assets/d5aea534-1385-49b3-95ca-f088ed7f127a" />

也与C++类似，跳过。

## 7、关系运算符

<img width="1086" height="429" alt="Image" src="https://github.com/user-attachments/assets/d644ee89-b464-46a6-9c1b-171567fe4a43" />

## 8、逻辑运算符

<img width="1092" height="294" alt="Image" src="https://github.com/user-attachments/assets/f7aec5a6-ff99-48f1-ad8b-5bf0f5f2e242" />

和C++的写法不太一样，需要注意一下

## 9、短路逻辑运算符

<img width="1095" height="222" alt="Image" src="https://github.com/user-attachments/assets/338dc814-fdb9-4922-bac1-867e1404a4ff" />

这样写效率高一些，就这样用了，还更符合习惯一点

```java
package Operator;

import java.util.Scanner;

public class OperatorDemo4
{
    static void main()
    {
        // 寻找7的有缘数，定义一个两位整数，只要该数字包含7或者是7的倍数，就是7的有缘数
        int num = 0;
        System.out.println("输入一个两位整数:");
        Scanner sc = new Scanner(System.in);
        num = sc.nextInt();
        if (num % 7 == 0 || num / 10 == 7 || num % 10 == 7)
        {
            System.out.println(num + "是7的有缘数");
        } else
        {
            System.out.println(num + "不是7的有缘数");
        }
        sc.close();
    }
}

```

## 10、三元运算符

<img width="528" height="225" alt="Image" src="https://github.com/user-attachments/assets/a16f1788-0c35-4594-b1fa-cfa21667f3df" />

```java
package Operator;

import java.util.Scanner;

public class OperatorDemo5
{
    static void main()
    {
        // 输入两个整数，输出其中较大数
        int a = 0;
        int b = 0;
        System.out.println("请输入两个整数:");
        Scanner sc = new Scanner(System.in);
        a = sc.nextInt();
        b = sc.nextInt();
        int max = (a > b) ? a : b;
        System.out.println("较大数是:" + max);
        sc.close();
    }
}
```

## 11、运算符的优先级

<img width="705" height="972" alt="Image" src="https://github.com/user-attachments/assets/62b7195a-1e63-4f94-a667-cfc4c1eae281" />

**小括号()优先于所有**

