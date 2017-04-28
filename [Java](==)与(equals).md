
---
title: 「Java」(==)与(equals)
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Java
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x01 两种比较方法

- `==`
    
    - 基本类型
        
        对于基本类型，`==` 的功能是比较值。
        
    - `Object`
        
        比较对象在内存中的地址。

- `equals`
    
    基本类型无equals方法。Object对象默认equals的实现如下：
    
    ``` java
    /**
     * ...
     * @param   obj   the reference object with which to compare.
     * @return  {@code true} if this object is the same as the obj
     *          argument; {@code false} otherwise.
     * @see     #hashCode()
     * @see     java.util.HashMap
     */
    public boolean equals(Object obj) {
        return (this == obj);
    }
    ```
    
    有很长一段注释，最终的实现我们可以看到还是用的 `==` 来比较两个对象在内存中的地址。


## 0x02 equals() & hashCode()

对于equals的默认实现——比较对象在内存中的地址——有时候可能并不是我们期望的，比如有一个Student类：

``` java
class Student{
   private int id;
   private String name;

   public Student(int id){
       this.id = id;
   }

   // ... getter and setter
}
```

当 Student.id 相同时，我们更愿意认为这是同一个学生，而下面这个测试是无法通过的：

``` java
@Test
public void equalsStudent() throws Exception {
   Student st1 = new Student(2333);
   Student st2 = new Student(2333);
   assertEquals(st1, st2);
   // assertNotEquals(st1, st2);
}
```

为了达到我们的目的——相同的学号就认为是同一个人——我们可以重写Student类的 equals 方法：

``` java
class Student {
   private int id;
   private String name;

   public Student(int id) {
       this.id = id;
   }

   @Override
   public boolean equals(Object o) {
       if (null == o) {
           return false;
       }
       Student std = (Student) o;
       if (this.id == std.id) {
           return true;
       }
       return super.equals(o);
   }

   // ... getter and setter
}
```

修改之后上面的测试就可以通过了。Demo毕竟是简单的，当我们在实际的使用中需要重写equals方法时还是需要遵守它的生成规则，这里贴出来供参考：

``` shell
public boolean equals(Object obj)

    指示其他某个对象是否与此对象“相等”。

    equals 方法在非空对象引用上实现相等关系：

        自反性：对于任何非空引用值 x，x.equals(x) 都应返回 true。
        对称性：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。
        传递性：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。
        一致性：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改。
        对于任何非空引用值 x，x.equals(null) 都应返回 false。 

    Object 类的 equals 方法实现对象上差别可能性最大的相等关系；即，对于任何非空引用值 x 和 y，当且仅当 x 和 y 引用同一个对象时，此方法才返回 true（x == y 具有值 true）。

    注意：当此方法被重写时，通常有必要重写 hashCode 方法，以维护 hashCode 方法的常规协定，该协定声明相等对象必须具有相等的哈希码。

    参数：
        obj - 要与之比较的引用对象。 
    返回：
        如果此对象与 obj 参数相同，则返回 true；否则返回 false。
    另请参见：
        hashCode(), Hashtable
``` 

事情还没有结束。看下面代码：

``` java
@Test
public void studentSet() throws Exception {
   Student st1 = new Student(2333);
   Student st2 = new Student(2333);

   Set<Student> stds = new HashSet<>();
   stds.add(st1);
   stds.add(st2);
   assertEquals(1, stds.size());
}
```

你会发现这段代码测试不通过。set不是重复对象只会保留一份吗，为什么不是 1 呢？

这里就要介绍 hashCode() 方法：

``` shell
public int hashCode()

    返回该对象的哈希码值。支持此方法是为了提高哈希表（例如 java.util.Hashtable 提供的哈希表）的性能。

    hashCode 的常规协定是：

        在 Java 应用程序执行期间，在对同一对象多次调用 hashCode 方法时，必须一致地返回相同的整数，前提是将对象进行 equals 比较时所用的信息没有被修改。从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。
        如果根据 equals(Object) 方法，两个对象是相等的，那么对这两个对象中的每个对象调用 hashCode 方法都必须生成相同的整数结果。
        如果根据 equals(java.lang.Object) 方法，两个对象不相等，那么对这两个对象中的任一对象上调用 hashCode 方法不 要求一定生成不同的整数结果。但是，程序员应该意识到，为不相等的对象生成不同整数结果可以提高哈希表的性能。 

    实际上，由 Object 类定义的 hashCode 方法确实会针对不同的对象返回不同的整数。（这一般是通过将该对象的内部地址转换成一个整数来实现的，但是 JavaTM 编程语言不需要这种实现技巧。）

    返回：
        此对象的一个哈希码值。
    另请参见：
        equals(java.lang.Object), Hashtable
```

当我们使用用哈希表实现的工具类时，这个方法的价值就体现了。由于之前我们没有重写这个方法，把Student对象存入HashSet时还是按照之前系统默认的方式计算他们的hash值，导致Set中存在两个Student对象。

下面做一个简单修改：

``` java
class Student {
   private int id;
   private String name;

   public Student(int id) {
       this.id = id;
   }

   @Override
   public boolean equals(Object o) {
       if (null == o) {
           return false;
       }

       // 有时候instanceof不合适的时候可以考虑用getClass()方法
       if (getClass() != o.getClass()) {
           return false;
       }

       if (o instanceof Student) {
           Student std = (Student) o;
           if (this.id == std.id) {
               return true;
           }
       }

       return super.equals(o);
   }

   @Override
   public int hashCode() {
       return this.id; 
       // return super.hashCode();
   }

   // ... getter and setter
}
```

再运行下面的测试，你会发现和我们期待的一致了：

``` java
@Test
public void studentSet() throws Exception {
   Student st1 = new Student(2333);
   Student st2 = new Student(2333);

   Set<Student> stds = new HashSet<>();
   stds.add(st1);
   stds.add(st2);
   assertEquals(1, stds.size());
}
```

Demo中直接将id作为hashcode返回不是一种好的生成方式，具体的生成规则请参考上面的注释。


## 0x03 Demo

再来看一个demo：

``` java
@Test
public void stringTest() throws Exception {
   String str1 = "abc";
   String str2 = "abc";
   String str3 = new String("abc");

   System.out.println(str1 == str2);
   System.out.println(str1 == str3);

   assertEquals(str1, str2);
   assertEquals(str1, str3);
}
```

结果会怎样？没错，测试通过，输出：true，false。

首先我们来看下String类重写的equals()和hashCOde():

``` java
@Override 
public boolean equals(Object other) {
   if (other == this) {
     return true;
   }
   if (other instanceof String) {
       String s = (String)other;
       int count = this.count;
       if (s.count != count) {
           return false;
       }
       if (hashCode() != s.hashCode()) {
           return false;
       }
       for (int i = 0; i < count; ++i) {
           if (charAt(i) != s.charAt(i)) {
               return false;
           }
       }
       return true;
   } else {
       return false;
   }
}

@Override 
public int hashCode() {
   int hash = hashCode;
   if (hash == 0) {
       if (count == 0) {
           return 0;
       }
       for (int i = 0; i < count; ++i) {
           hash = 31 * hash + charAt(i);
       }
       hashCode = hash;
   }
   return hash;
}
```

从上面可以看出，String类重写了这两个方法，equals()中的逻辑是比较字符串中每个字符是否相同。因此 str1, str2, str3相同就可以理解了。
对于 `str1 == str2` 和 `str1 != str3` 这涉及到不同字符串创建方法。

> [在开始这个例子之前，同学们需要知道JVM处理String的一些特性。Java的虚拟机在内存中开辟出一块单独的区域，用来存储字符串对象，这块内存区域被称为字符串缓冲池。当使用 String a = "abc"这样的语句进行定义一个引用的时候，首先会在字符串缓冲池中查找是否已经相同的对象，如果存在，那么就直接将这个对象的引用返回给a，如果不存在，则需要新建一个值为"abc"的对象，再将新的引用返回a。String a = new String("abc");这样的语句明确告诉JVM想要产生一个新的String对象，并且值为"abc"，于是就在堆内存中的某一个小角落开辟了一个新的String对象。](https://github.com/GeniusVJR/LearningNotes/blob/master/Part2/JavaSE/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.md) 

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

