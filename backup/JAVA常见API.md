# JAVA常见API

**API**：JDK提供各种功能的JAVA类

**不需要导包的情况**：

1. 使用本包中的类
2. 使用`java.lang`包下的类

```java
import java.util.Random;

public class Test
{
    static void main()
    {
        Random rand = new Random();
        // [0.0-1.0)
        double v = rand.nextDouble();
        System.out.println(v);
    }
}
```

## 字符串String

`String`类，定义在`java.lang`包下，无需导包。JAVA中的所有字符串文字都为此类对象

字符串的内容是**不可变的**，它的对象在创建后不能被更改

### 字符串的赋值

1. 直接赋值
2. `new`关键字

```java
package APIdemo02;

public class Test
{
    static void main()
    {
        // 直接赋值
        String s = "abc";
        System.out.println("s = " + s);
        // 空参构造
        String s1 = new String();
        System.out.println("s1 = " + s1);
        // 有参构造
        String s2 = new String(s);
        System.out.println("s2 = " + s2);
        // 字符数组构造
        char[] chs = {'a','b','c','d','e'};
        String s3 = new String(chs);
        System.out.println("s3 = " + s3);
        // 字节数组构造
        // 通过ASCII码转换
        byte[] bytes = {97, 98, 99, 100, 101};
        String s4 = new String(bytes);
        System.out.println("s4 = " + s4);

        System.out.println(s.hashCode());
        System.out.println(s1.hashCode());
        System.out.println(s2.hashCode());
        System.out.println(s3.hashCode());
        System.out.println(s4.hashCode());
    }
}


```

```cmd
"C:\JAVA tools\bin\java.exe" "-javaagent:C:\JAVA tools\IntelliJ IDEA 2026.2.0.1\lib\idea_rt.jar=60377" -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath C:\Users\72982\IdeaProjects\API\out\production\API APIdemo02.Test
s = abc
s1 = 
s2 = abc
s3 = abcde
s4 = abcde
96354
0
96354
92599395
92599395
```

### 字符串的比较

`boolean equals(要比较的字符串)` 完全一样结果才是`true`,否则是`false`

`boolean equalsIgnoreCase(要比较的字符串)`忽略大小写的比较

```java
package APIdemo03;

public class Test
{
    static void main()
    {
        String username = "zhangsan";
        String rightUsername = "zhangsan";
        System.out.println("username.equals(rightUsername) = " + username.equals(rightUsername));

        username = "Zhangsan";
        System.out.println("username.equals(rightUsername) = " + username.equals(rightUsername));

        System.out.println("username.equalsIgnoreCase(rightUsername) = " + username.equalsIgnoreCase(rightUsername));
    }
}
```

```cmd
"C:\JAVA tools\bin\java.exe" "-javaagent:C:\JAVA tools\IntelliJ IDEA 2026.2.0.1\lib\idea_rt.jar=52194" -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath C:\Users\72982\IdeaProjects\API\out\production\API APIdemo03.Test
username.equals(rightUsername) = true
username.equals(rightUsername) = false
username.equalsIgnoreCase(rightUsername) = true
```

### 字符串遍历

`char charAt(index)`方法

```java
package APIdemo04;

import java.util.Arrays;
import java.util.Scanner;
public class Test
{
    static void main()
    {
        String s;
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入一个字符串");
        s = sc.next();
        for (int i = 0; i < s.length(); i++) {
            System.out.println(s.charAt(i));
        }
        int[] arr = {1,2,3};
        String s1 = Arrays.toString(arr);
        System.out.println("s1 = " + s1);
    }
}

```



```cmd
"C:\JAVA tools\bin\java.exe" "-javaagent:C:\JAVA tools\IntelliJ IDEA 2026.2.0.1\lib\idea_rt.jar=61139" -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath C:\Users\72982\IdeaProjects\API\out\production\API APIdemo04.Test
请输入一个字符串
123
1
2
3
s1 = [1, 2, 3]
```

### 字符串的截取、替换和其他方法

**截取**：`String substring(int beginIndex,int endIndex)`

- 包头不包尾，包左不包右，左闭右开区间
- 返回值为截取的小串  

```java
package APIdemo05;

public class Test
{
    static void main()
    {
        String str = "abcdefg";
        String sub = str.substring(1,5);
        System.out.println("sub = " + sub);
        // 字符串的加密，保留用户名的第一个字符，后续用***代替
        String username = "zhangsan";
        char firstChar = username.charAt(0);
        String encryptedUsername1 = firstChar + "***";
        System.out.println("encryptedUsername1 = " + encryptedUsername1);

        String encryptedUsername2 = username.substring(0,1)+"***";
        System.out.println("encryptedUsername2 = " + encryptedUsername2);
    }
}
```

```java
package APIdemo06;

public class Test
{
    // 写一个使用replace函数进行字符替换的Demo
    static void main()
    {
        String str = "hello world";
        String newStr = str.replace("world", "java");
        System.out.println("newStr = " + newStr);
    }

}
```

```java
package APIdemo06;

public class Test
{
    // 写一个使用replace函数进行字符替换的Demo
    static void main()
    {
        String str = "hello world";
        String newStr = str.replace("world", "java");
        System.out.println("newStr = " + newStr);

        // 包含
        boolean c = str.contains("hello");
        System.out.println("c = " + c);
        // 判断开头、结尾
        boolean s = str.startsWith("h");
        System.out.println("s = " + s);
        boolean e = str.endsWith("d");
        System.out.println("e = " + e);
        // 查找
        int index = str.indexOf("o");
        System.out.println("index = " + index);
        // 判断是否为空
        System.out.println("str.isEmpty() = " + str.isEmpty());
        // 转字符数组
        char[] chars = str.toCharArray();
        System.out.println("chars = " + java.util.Arrays.toString(chars));
        // 大小写转换
        System.out.println("str.toUpperCase() = " + str.toUpperCase());
        System.out.println("str.toLowerCase() = " + str.toLowerCase());
        // 去除头尾空格
        String str2 = "  hello world  ";
        System.out.println("str2 = " + str2);
        System.out.println("str2.trim() = " + str2.trim());
    }

}
```

```cmd
"C:\JAVA tools\bin\java.exe" "-javaagent:C:\JAVA tools\IntelliJ IDEA 2026.2.0.1\lib\idea_rt.jar=56169" -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath C:\Users\72982\IdeaProjects\API\out\production\API-String APIdemo06.Test
newStr = hello java
c = true
s = true
e = true
index = 4
str.isEmpty() = false
chars = [h, e, l, l, o,  , w, o, r, l, d]
str.toUpperCase() = HELLO WORLD
str.toLowerCase() = hello world
str2 =   hello world  
str2.trim() = hello world
```

