---
layout: post
title: '重写equals为什么一定要重写hashcode'
date: 2020-02-16
author: yza
color: rgb(255,210,32)
cover: '../assets/java.jpg'
tags: java java基础
typora-copy-images-to: ..\assets\image
---



## 为什么重写equals方法时，要求必须重写hashCode方法？

### **1 equals方法**

Object类中默认的实现方式是  :  return this == obj  。那就是说，只有this 和 obj引用同一个对象，才会返回true。当不重写equals和hashcode时，对象属性改变不影响equals判断结果以及hashcode取值。

而我们往往需要用equals来判断 2个对象是否等价，而非验证他们的唯一性。这样我们在实现自己的类时，就要重写equals.

 

按照约定，equals要满足以下规则。

- **自反性**:  x.equals(x) 一定是true
- **对null**:  x.equals(null) 一定是false
- **对称性**:  x.equals(y)  和 y.equals(x)结果一致
- **传递性**:  a 和 b equals , b 和 c  equals，那么 a 和 c也一定equals。
- **一致性**:  在某个运行时期间，2个对象的状态的改变不会影响equals的决策结果，那么，在这个运行时期间，无论调用多少次equals，都返回相同的结果。

 

举例：重写equals方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 class Test
 2 {
 3     private int num;
 4     private String data;
 5 
 6     public boolean equals(Object obj)
 7     {
 8         if (this == obj)
 9             return true;
10 
11         if ((obj == null) || (obj.getClass() != this.getClass()))
12             return false;
13 
14         //能执行到这里，说明obj和this同类且非null。
15         Test test = (Test) obj;
16         return num == test.num&& (data == test.data || (data != null && data.equals(test.data)));
17     }
18 
19     public int hashCode()
20     {
21         //重写equals，也必须重写hashCode。具体后面介绍。
22     }
23 
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

### **2 hashCode方法**

这个方法返回对象的散列码，返回值是int类型的散列码。
对象的散列码是为了更好的支持基于哈希机制的Java集合类，例如 Hashtable, HashMap, HashSet 等。


关于hashCode方法，一致的约定是：

- 在某个运行时期间，只要对象的（字段的）变化不会影响equals方法的决策结果，那么，在这个期间，无论调用多少次hashCode，都必须返回同一个散列码。
- 如果2个对象通过equals调用后返回是true，那么这个2个对象的hashCode方法也必须返回同样的int型散列码如果2个对象通过equals返回false，他们的hashCode返回的值**可能相同**。也就是说，如果没有重写equals和hashcode，可能两个对象==为false但默认的hashcode相同，也就放在同一个桶里。如果重写了，两个对象equals不同，但hashcode恰好相同，这时俩对象也在同一个桶里。(然而，程序员必须意识到，hashCode返回独一无二的散列码，会让存储这个对象的hashtables更好地工作。)

**重写了euqls方法的对象必须同时重写hashCode()方法。**

 

### **3 为什么必须重写hashCode方法？**

在上面的例子中，Test类对象有2个字段，num和data，这2个字段代表了对象的状态，他们也用在equals方法中作为评判的依据。那么， 在hashCode方法中，这2个字段也要参与hash值的运算，作为hash运算的中间参数。这点很关键，这是为了遵守：**2个对象equals，那么 hashCode一定相同规则。**

也是说，参与equals函数的字段，也必须都参与hashCode 的计算。

 

### **4 重写hashCode时注意事项**

重写hashCode方法时除了上述一致性约定，还有以下几点需要注意：

（1）返回的hash值是int型的，防止溢出。

（2）不同的对象返回的hash值应该尽量不同。（为了hashMap等集合的效率问题）

（3）《Java编程思想》中提到一种情况

“设计hashCode()时最重要的因素就是：无论何时，对同一个对象调用hashCode()都应该产生同样的值。如果在讲一个对象用put()添加进HashMap时产生一个hashCdoe值，而用get()取出时却产生了另一个hashCode值，那么就无法获取该对象了。所以如果你的hashCode方法依赖于对象中易变的数据，用户就要当心了，因为此数据发生变化时，hashCode()方法就会生成一个不同的散列码”。

举个例子

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Test {
    
    private int num;
    private String data;
    
    public Test(int num,String data){
        this.num = num;
        this.data = data;
    }
    
    public void setNum(int num) {
        this.num = num;
    }

    public boolean equals(Object obj)
    {
        if (this == obj)
            return true;

        if ((obj == null) || (obj.getClass() != this.getClass()))
            return false;

        Test test = (Test) obj;
        return num == test.num&& (data == test.data || (data != null && data.equals(test.data)));
    }

    public int hashCode()
    {
        int hash = 7;
        hash = 31*hash+num;
        hash = 31*hash+data.hashCode();
        return hash;
        
    }
    
    public static void main(String[] args) {
        
        Map<Test,Integer> map = new HashMap<>();
        Test t1 = new Test(21,"ouym");
        map.put(t1, 1);
        t1.setNum(20);
        System.out.println(map.get(t1));
        
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

输出值为null，我的天呐，hashMap取不到值了。

不重写equals和hashCode方法的话是不依赖于对象属性的变化的，也就是说这里使用默认的hashCode方法可以取到值。但是我们重写equal方法的初衷是判定name和num属性都相等的Test对象是相等的，而不是说同一个对象的引用才相等，而num=21和num=20明显不想等，所以这里hashCode返回值不同并不违背设计的初衷。而hashmap需要用hashcode找到对应的桶，第二次找到的和第一次不同。

#### 引用：https://www.cnblogs.com/ouym/p/8963219.html

#### 感谢！！！