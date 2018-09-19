---
layout: post
title: Java Reflection 反射机制 
date: 2018-8-31 10:30:24.000000000 +09:00
tag: Spring
---

&emsp;&emsp;最近在做东西的时候再次用到Java的反射机制，突然想起自己在之前的项目上做动态方法调用的时候也用过反射，但是始终没搞明白是个什么东西，今天重新学习并整理记录下来。

### 概念
>&emsp;&emsp;JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

>&emsp;&emsp;JAVA反射（放射）机制：“程序运行时，允许改变程序结构或变量类型，这种语言称为动态语言”。从这个观点看，Perl，Python，Ruby是动态语言，C++，Java，C#不是动态语言。但是JAVA有着一个非常突出的动态相关机制：Reflection，用在Java身上指的是我们可以于运行时加载、探知、使用编译期间完全未知的classes。换句话说，Java程序可以加载一个运行时才得知名称的class，获悉其完整构造（但不包括methods定义），并生成其对象实体、或对其fields设值、或唤起其methods。

### Java类反射中所必须的类
&emsp;&emsp;Java的类反射所需要的类并不多，它们分别是：`Field`、`Constructor`、`Method`、`Class`、`Object`，下面我将对这些类做一个简单的说明。

>&emsp;&emsp;Field类：提供有关类或接口的属性的信息，以及对它的动态访问权限。反射的字段可能是一个类（静态）属性或实例属性，简单的理解可以把它看成一个封装反射类的属性的类。

>&emsp;&emsp;Constructor类：提供关于类的单个构造方法的信息以及对它的访问权限。这个类和Field类不同，Field类封装了反射类的属性，而Constructor类则封装了反射类的构造方法。

>&emsp;&emsp;Method类：提供关于类或接口上单独某个方法的信息。所反映的方法可能是类方法或实例方法（包括抽象方法）。 这个类不难理解，它是用来封装反射类方法的一个类。

>&emsp;&emsp;Class类：类的实例表示正在运行的 Java 应用程序中的类和接口。枚举是一种类，注释是一种接口。每个数组属于被映射为 Class 对象的一个类，所有具有相同元素类型和维数的数组都共享该 Class 对象。

>&emsp;&emsp;Object类：每个类都使用 Object 作为超类。所有对象（包括数组）都实现这个类的方法。

### 使用
&emsp;&emsp;简单讲一下通过反射动态调用方法。

{% highlight java %}
package com.pigetest.util;

import java.lang.reflect.Method;

public class PrivateMethodTestHelper {
    public static Object invoke(String clazzName,String methodName,Object...params){
        try {
            Class<?> clazz=Class.forName(clazzName);
            Object obj=clazz.newInstance();
            Method[] methods = clazz.getDeclaredMethods();
            Method callMethod=null;
            for(Method method:methods){
                if(method.getName().equals(methodName)){
                    callMethod=method;
                    break;
                }
            }
            callMethod.setAccessible(true);
            return (Object) callMethod.invoke(obj,params);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void main(String[] args) {
        int value=(Integer) PrivateMethodTestHelper.invoke("com.pigetest.util.AddNumber","addNumber",1,2);
        System.out.println(value);
    }

}
{% endhighlight %}
