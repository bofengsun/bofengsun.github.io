---
title: 'Java基础知识'
date: 2019-11-07 16:05:09
tags: [Java]
published: true
hideInList: false
feature: /post-images/java-ji-chu-zhi-shi.png
---
万丈高楼平地起，掌握并牢记基础知识，是学习后面知识的根基，下面我们一起回顾一下Java的基础知识吧！
<!-- more -->

## 一、基础部分

### 1、基本数据类型8中

```JAVA
四种整数型：byte(8)、short(16)、int(32)、long(64)

二种浮点型：float(32)、double(64)

一种字符型：char(16)

一种布尔型：boolean
```

### 2、Object类常用方法

```java
Equals
Hashcode
toString
wait
notify
notifyAll
clone
getClass
finalize
```

### 3、值传递、引用传递

Java中是值传递，对于基本数据类型，传递的是值的副本，对于引用类型传递的是对象地址的副本。

```java
package com.itbofeng;

public class TestValuePassRefPass {
    public static void main(String[] args) {
        int i=100;
        String str="aa";
        changeInt(i);
        changeString(str);
        System.out.println(i);//100
        System.out.println(str);//aa
    }
    private static void changeInt(int i) {
        i=0;
    }
    private static void changeString(String str) {
        str="bb";
    }
}
```

### 4、数组实例化

```JAVA
1、数组一旦实例化，它的长度就是固定的

2、静态实例化:```java int[] a=new int[]{1,3,3}``` , ```java int[] a={1,3,3}```
   动态实例化：实例化数组的时候，只指定了数组程度，数组中所有元素都是数组类型的默认值

3、数组反序
  1、将原数组中的元素进行逆序（原数组改变，占用内存小）
  public void arrayReverse() {
        int []originArray={1,2,3,4,5}
        int temp = 0;
        for (int i = 0; i < length / 2; i++) {
            temp = originArray[i];
            originArray[i] = originArray[length - i - 1];
            originArray[length - i - 1] = temp;
        }
  }
  2、创建新数组逆序存储原数组元素（原数组不变，占用内存大）
  public void arrayReverse() {
        int []originArray={1,2,3,4,5}
        int length=originArray.length;
        int []reverseArray = new int[length];
        for (int i = 0; i < length; i++) {
            reverseArray[i] = originArray[length - i - 1];
        }
  }
```

### 5、&和&&的区别 、|和||的区别 

```java
	&和&&都可以用作逻辑与的运算符，表示逻辑与（and），当运算符两边的表达式的结果都为true时，整个运算结果才为true，否则，只要有一方为false，则结果为false。
	&&具有短路的功能,&不具备短路的功能
	|和||都可以用作逻辑或的运算符，表示逻辑或（or），当运算符两边的表达式的结果有一个为true时，整个运算结果才为true，否则，只有全部为false，则结果为false。
	||具有短路的功能,|不具备短路的功能
```

### 6、String和StringBuilder，StringBuffer的区别

```Java
	1、String对象不可变，因此在进⾏任何内容上的修改时都会创建新的字符串对象，修改操作太多造成资源浪费。
    2、StringBuilder和StringBuffer修改字符串不是创建新的对象，而是在原对象上面进行修改。
    3、StringBuffer线程安全的是 StringBuilder是线程不安全的，StringBuilder效率比StringBuffer高。
```

### 7、HashMap、HashTable、 ConcurrentHashMap 的区别

```Java
    1、HashMap不是线程安全的、HashTable、ConcurrentHashMap是线程安全的。
    2、HashTable是synchronized锁定方法，在高并发环境下效率比较低，ConcurrentHashMap采用分段锁，从而增加并发性能。
    备注：在源码讲解中会分别对HashMap、HashTable、ConcurrentHashMap进行讲解。
```

### 8、switch 语法

```java
    1、在switch（exp）中，exp只能是一个整数表达式或者枚举常量（更大字体），整数表达式可以是int基本类型或Integer包装类型，由于，byte,short,char都可以隐含转换为int，所以，这些类型以及这些类型的包装类型也是可以的
    2、从Java SE 7开始，switch 支持字符串 String 类型了，同时 case 标签必须为字符串常量或字面量。
    3、Long、Double不支持 非整数表达式或者枚举常量以及字符串
```

### 9、大数计算思路

```java
	考点：
        1、计算机原理
        2、计算机中的算术运算会发生越界的情况
        3、具备一定的面向对象的设计思想
    思路：
        1、计算机原理（原码、反码、补码）
        2、面向对象设计
    实现：
        1、这个类内部有两个成员变量，一个表示符号，另一个用字节数组表示数值的二进制数
        2、有一个构造方法，把一个包含有多位数值的字符串转换到内部的符号和字节数组中
        3、提供加减乘除的功能
    代码：
        public class BigInteger{
            byte sign;
            byte[] val;
            public Biginteger(String val)	{
                sign =a ;
                val = ;
            }
            public BigInteger add(BigInteger other)	{

            }
            public BigInteger subtract(BigInteger other)	{

            }
            public BigInteger multiply(BigInteger other){

            }
            public BigInteger divide(BigInteger other){

            }
        }

```

### 10、"=="和equals方法  

```java
	1、对于引用类型，涉及了两块内存，对象本身占用一块内存（堆），变量也占用一块内存（栈），例如Objet obj = new Object();变量obj是一个内存，new Object()是另一个内存，此时，变量obj所对应的内存中存储的数值就是对象占用的那块内存的首地址。
	2、==
      ==操作符专门用来比较基本数据类型的值是否相等，以及引用类型的变量地址是否相等，即比较栈中值是否相等
    3、equals
      equals方法是Java Object类中的方法，在Object类中equals的实现是等同于==，如果一个对象没有重写equals方法，那么此时equals方法等同于==，但是在许多对象比较中，例如字符串，我们实际需要比较的对象的值是否相等，而不是变量的值是否相等，那么此时需要重写equals
```

### 11、基本数据类型和包装类型的区别

```Java
    1、基本类型和包装类型的默认值不同，基本类型有自己的默认值，包装类型默认值是null
    2、基本类型==比较的是值，包装类型==比较的是变量地址
    3、包装类型提供了很多其他的方法
```

### 12、作用域public，private，protected，以及不写时（friendly）区别  

|  作用域    | 当前类  | 同一包  | 子孙类  | 其他包  |
| :-------: | :----: | :----: | :----: | :----: |
|  public   |   √    |   √    |   √    |   √    |
| protected |   √    |   √    |   √    |   ×    |
| friendly  |   √    |   √    |   ×    |   ×    |
|  private  |   √    |   ×    |   ×    |   ×    |

### 13、静态变量和实例变量的区别  

```Java
    1、在语法定义上的区别：静态变量前要加static关键字，而实例变量前则不加
    2、在程序运行时的区别：实例变量属于某个对象的属性，必须创建了实例对象，其中的实例变量才会被分配空间，才能使用这个实例变量。静态变量不属于某个实例对象，而是属于类，所以也称为类变量，只要程序加载了类的字节码，不用创建任何实例对象，静态变量就会被分配空间，静态变量就可以被使用了。总之，实例变量必须创建对象后才可以通过这个对象来使用，静态变量则可以直接使用类名来引用。
```

### 14、重载(Overload)&重写（Override）

```JAVA
    1、重载：是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。
    2、重写：子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。
```

### 15、continue &  break 

```JAVA
    1、continue:在循环语句中,它会中断正常的控制流程，跳出当次循环,然后继续下一次循环；
    2、break：可用在循环,判断等语句中,用于退出当前语句，在循环语句中就是退出当前循环；
```

### 16、final 关键字

### 17、this 关键字

### 18、transient 关键字

### 19、instanceof 关键字

### 20、字符型常量和字符串常量的区别 

### 21、成员变量与局部变量的区别有那些

### 22、自动拆箱装箱

### 23、接口&抽象类

### 24、Java 引用

### 25、hashCode&equals

### 26、Comparable和Comparator



## 二、面向对象部分

## 三、WEB相关部分

❤️❤️❤️未完待续❤️❤️❤️


