# 1、判断语句（if）

## 判断语句的细节

1. **if语句大括号的位置**：采用紧凑风格	
2. **if语句大括号的省略**：大括号中只有一行语句可省略大括号
3. **小括号后不能有分号**：会断开if语句结构
4. **判断布尔类型的变量**：直接把变量写在小括号中即可

## 语法格式

```java
if(关系表达式){
    语句体;
}
```

## 使用场景

判断代码是否满足执行的条件

# 2、选择语句（switch）

## 1、`default`的位置和省略

**位置：**`case`和`default`的位置可以任意书写，一般来讲,`case`从小到大依次书写，`default`写在最下面

**省略：**`default`可以省略，但如果所有的case都不匹配则没有任何的输出结果

## 2、`case`穿透

**执行流程:**

1. 拿着小括号中表达式的值跟下面的case进行匹配
2. 如果匹配上了，就会执行case里面的语句体，遇到break结束整个的switch(正常情况)
3. 如果在执行语句体的时候没有看到break，那么程序会继续执行下一个case的语句体，真到遇到break或者运行完整个的switch为止

**应用场景**

```java
package judge;

public class SwitchDemo1
{
    static void main()
    {
        // 判断季节
        int month = 5;
        switch (month) {
            case 3:
            case 4:
            case 5:
                System.out.println("春季");
                break;
            case 6:
            case 7:
            case 8:
                System.out.println("夏季");
                break;
            case 9:
            case 10:
            case 11:
                System.out.println("秋季");
                break;
            case 12:
            case 1:
            case 2:
                System.out.println("冬季");
                break;
            default:
                System.out.println("月份输入错误!");
                break;
        }
    }
}
```

**`switch`新特性**

1. 箭头标签
2. case后面可以写多个值
3. switch可以有运行结果
4. yield关键字

```java
package judge;

public class SwitchDemo2
{
    static void main()
    {
        int number = 3;

        switch (number) {
            case 1 -> System.out.println("-");
            case 2 -> System.out.println("二");
            case 3 -> System.out.println("三");
            default -> System.out.println("没有这个星期");
        }

        int a = 10;
        int b = 10;

        String operator = "+";
        int result = switch (operator) {
            case "+" -> a + b;
            case "-" -> a - b;
            case "*" -> a * b;
            case "/" -> a / b;
            default -> 0;
        };
        System.out.println(result);
    }
}

```

# 3、循环语句

## for循环

```java
for(int i=0;i<10;i++){
    ----------;
}
```

格式也跟C++一样

## while循环

```java
while(条件判断语句){
循环体语句
条件控制语句
}
```

**for和while的区别:**

1. for循环中:知道循环次数或者循环的范围
2. while循环:不知道循环的次数和范围，只知道循环的结束条件。

## do while循环

```java
do{
    
}while()
```

先执行后判断，循环体内容至少执行一次。

# 4、数组

**数组：**是一种容器，可以用来存储==同种数据类型==的多个值，存值时会存在隐式类型转换

**特点：**

1. 空间上连续
2. 一旦确定长度，长度不可变

## 数组的静态初始化

**初始化:**是指在定义变量、数组、对象的时候进行赋值 ，**定义+赋值**同时进行

**静态初始化:**创建数组的时候，直接给数组赋值

```java
int arr[] = new int[]{1,2,3};
int arr[] = {1,2,3}; // 这样写也是可以的，像C++的初始化列表
```

```java
package array;

public class ArrayDemo1
{
    static void main()
    {
        // 数组的静态初始化:
        // 1.定义数组存储3位同学的年龄
        // 2.定义数组存储5位同学的身高
        // 3.定义数组存储3位同学的名字

        int[] age = {18, 19, 20};
        double[] height = {1.75, 1.80, 1.65, 1.70, 1.85};
        String[] name = {"张三", "李四", "王五"};
    }
}
```



## 数组的元素访问

**获取、修改**

```java
package array;

public class ArrayDemo2
{
    static void main()
    {
        /*
         * 1、获取数据
         * 2、修改数据
         */
        int[] arr = {1, 2, 3, 4, 5};
        // 1、获取数据
        int value = arr[0];
        // 2、修改数据
        arr[0] = 10;
        System.out.println(value);
        System.out.println(arr[0]);
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i] = (i + 1) * 10);
        }
    }
}
```



## 数组的遍历

```java
package array;

public class ArrayDemo3
{
    static void main()
    {
        // 遍历数组中的元素
        int[] arr = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
        // 使用增强for循环遍历数组中的元素,基于迭代器的方式遍历数组中的元素
        for (int value : arr) {
            System.out.println(value);
        }
    }
}

```



## 数组的动态初始化

**动态初始化：**创建数组的时候指定长度，由系统为数组分配初始值

```java
int[] arr = new int[3]
```

```java
package array;

import java.util.Scanner;

public class ArrayDemo4
{
    static void main()
    {
        // 对一个长度为5的int类型数组进行动态初始化
        int[] arr = new int[5];
        for (int i = 0; i < arr.length; i++) {
            int value = 0;
            Scanner sc = new Scanner(System.in);
            System.out.println("请输入第" + (i + 1) + "个元素:");
            value = sc.nextInt();
            arr[i] = value;
        }
        for (int i = 0; i < arr.length; i++) {
            System.out.println("第" + (i + 1) + "个元素是:" + arr[i]);
        }
    }
}
```

## 数组的常见问题

**最大的问题：**数组的索引越界

## 练习

```java
package array;

public class ArrayDemo5
{
    static void main()
    {
        // 定义一个长度为10的整型数组，存入1-100之间的随机数，要求随机数不能重复
        int[] arr = new int[10];
        for (int i = 0; i < arr.length; ) {
            int randomNum = (int) (Math.random() * 100 + 1);
            boolean isDuplicate = false;
            for (int j = 0; j <= i; j++) {
                if (arr[j] == randomNum) {
                    isDuplicate = true;
                    break;
                }
            }
            if (!isDuplicate) {
                arr[i] = randomNum;
                i++; // 只有在没有重复的情况下才增加索引i
            }
        }
        for (int value : arr) {
            System.out.println(value);
        }
    }
}
```

```java
package array;

import java.util.Arrays;

public class ArrayDemo6
{
    static void main()
    {
        // 用快慢指针法对有序数组进行去重
        int[] arr = {1, 2, 2, 3, 4, 4, 5, 6, 6, 7};
        int slow = 0;
        for (int fast = 1; fast < arr.length; fast++) {
            if (arr[fast] != arr[slow]) {
                arr[++slow] = arr[fast];
            }
        }
        int[] newArr = Arrays.copyOf(arr, slow + 1);
        // 打印去重后的数组
        for (int i = 0; i <= slow; i++) {
            System.out.println(newArr[i]);
        }
    }
}

```

