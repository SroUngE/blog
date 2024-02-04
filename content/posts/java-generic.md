---
title: "Java 泛型的深入思考：从历史起源到最佳实践"
description: "一个关于Java泛型的简短的研究调查和思考，依据官方的参考资料，试图理解泛型存在的意义，以及它的最佳实践。"
date: 2023-11-01
tags: ["Java"]
featuredImage: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
featuredImagePreview: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
draft: false
---

<!--more-->

本篇文章会尽量引用官方的文档，以及具有权威性的文章。
 [Generics tutorial by The Java™ Tutorials - Oracle](https://docs.oracle.com/javase/tutorial/java/generics/)  
[Effective Java - Joshua Bloch)](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual)  
[Generics in the Java Programming Language - Gilad Bracha)](https://www.oracle.com/technetwork/java/javase/generics-tutorial-159168.pdf)  
[When To Use Generics)](https://go.dev/blog/when-generics) 

## 泛型的历史

当这些理论出现之后，泛型变得可能：

- 类型推断（Type Interface）：20 世纪 70 年代，自动确定表达式的类型
- 多态（Polymorphism）：20 世纪 60 年代，允许在不同类型的数据上执行相同的操作

最早在 1973 年， 由Robin Milner(1991 年图灵奖获得者) 创建的ML(Meta Language)编程语言，引入了泛型的概念。ML 引入了多态类型和类型推断的概念，这对泛型编程起到了重要的影响。

在 ML 之后，泛型概念逐渐在其他编程语言中得到了采纳和发展。C++引入了 template 作为一种泛型编程的方式，它首次出现在 20 世纪 80 年代。

```C++
#include <iostream>
template <typename T>
T getMax(const T& a, const T& b) {
    return (a > b) ? a : b;
}

int main() {
    int intResult = getMax(5, 10);
    double doubleResult = getMax(3.14, 2.71);
    return 0;
}
```

Java的首个版本是于1995年发布的，当时它的主要目的是创建一种适用于嵌入式设备和网络编程的跨平台语言。然而随着时间的推移，类型安全性变得更加重要，因此在2004年，Java1.5引入了泛型。
> It's all about type safety！


## 泛型的意义

所以泛型存在的意义是什么？ 下面引用 Java 官方文档的一个介绍：
> In a nutshell, generics enable _types_ (classes and interfaces) to be parameters when defining classes, interfaces and methods.  
> 在定义类，接口，方法的时候，泛型使得类型（类和接口）得以成为参数。


这句话翻译过来就是，泛型是一种把类型传递为参数的机制，因此泛型的别名也叫**参数化类型(type parameters)**。那这样做有什么好处呢？或者说用泛型我们为了达到什么目的？

引用自 Java 官方文档：
1. **编译时强类型检查**：Java 编译器会对泛型代码应用强类型检查，使得类型问题在编译阶段而不是运行时被暴露
2. **消除类型转换**：`Cast-iron guarantee: the implicit casts added by the compilation of generics never fail.`由编译器产生的类型转换永远不会失败。
3. **代码重用**：使得不同的类型可以使用一样的代码

```java
// The following code snippet without generics requires casting:
List list = new ArrayList();
list.add("hello");
String s = (String) list.get(0);

// When re-written to use generics, the code does not require casting:
List<String> list = new ArrayList<>();
list.add("hello");
String s = list.get(0);   // no cast
```

## 泛型的最佳实践

Go 编程有一条通用准则：
> write Go programs by writing code, not by defining types. 

这句话的意思是，如果从定义泛型开始写代码，那你可能搞错了方向，从编写函数开始，如果发现使用泛型更好，再使用泛型。

所以什么时候使用泛型更好？先说结论：

> 如果你发现你写的代码几乎完全一样，区别只是代码里使用的类型不同，那就要考虑使用泛型来实现。


********
> 以下内容来自 《Effective Java》 第五章节 - 泛型
### 1. 请不要使用原生态类型

每一种泛型都定义了一个原生态类型（Raw Type），即不带任何实际类型参数的泛型名称。例如`List<E>`相对应的原生态类型就是`List`。
```java
List list = new ArrayList();
```
**原生态类型就像是从泛型声明中删除了所有泛型信息一样。失去了泛型在安全性和描述性方面的所有优势** 那为什么 java 语言的设计者还要允许这样使用呢？这是为了提供兼容性。

那么`List`和`List<Object>`有什么区别呢？  
原生态类型`List`逃避了泛型检查，失去了类型安全性，而参数化类型`List<Object>`则明确的告知编译器，它能持有任意类型的对象。

### 2. 消除非受检异常

当编译器无法从表达式中解析出当前类型的具体类型的时候，就会出现非受检警告：  
```
unchecked assignment: 未检查的赋值  
unchecked cast: 未检查的转换
```
**每一个未受检警告都说明运行时可能出现`ClassCastException`**，要尽可能地消除每一个非受检警告（Unchecked Warning），因为如果消除了所有警告，就可以确保代码是类型安全的。      
如果无法消除警告，同时可以证明警告引起的代码是类型安全的，只有在这种情况下，才可以用一个@SuppressWarnings(unchecked)注解来禁止这种警告。**应该始终在尽可能小的范围内使用这个注解**


> 更多深入的内容可以查看 《Java Generics and Collections》第8章


## 泛型擦除
说到泛型擦除之前先要理解下这些概念：
**可具体化(reifiable)类型**，是类型信息在运行时完全可用的类型。包括：

* 基本类型，如int
* 非泛型类型，如String, Runnable, User
* 原生态类型，如List
* 无界通配符的调用，如List<?>

**不可具体化(Non-reifiable types)类型**，是在编译时通过类型擦除移除信息的类型。  
例如，尽管`List<String>` 和 `List<Number>`在编译时被声明为不同类型，但在编译之后都被简化为了`List`类型，因此，JVM 无法在运行时区分这些类型。

不可具体化的类型在编译之后，将被擦除类型信息。来看看下面这些例子：

**有限制的类型擦除**

```java
public class Box<T> {
    
    private T id;

    public T getId() {
        return id;
    }
}
```
对字节码文件反编译后的代码：
```java
public class Box {
    
    public Box() {
    }

    // 这里的类型被擦除为Object
    public Object getId() {
        return id;
    }
    // 这里的类型被擦除为Object
    private Object id;
}
```
**有限制类型擦除**

```java
public class StringBox<T extends String> {
    
    private T id;

    public T getId() {
        return id;
    }
}
```
对字节码文件反编译后的代码：
```java
public class StringBox
{

    public StringBox() {
    }

    // 这里的类型被擦除为String
    public String getId() {
        return id;
    }
    // 这里的类型被擦除为String
    private String id;
}
```
Java 为了保证向下兼容性，它的泛型全部都是在编译期间实现的。编译器执行类型检查和类型推断，然后生成普通的非泛型的字节码。这种就叫做类型擦除。编译器在编译的过程中执行类型检查来保证类型安全，但是在随后的字节码生成之前将其擦除。

**多态与类型擦除的兼容**

```java
public interface Validator<T> {
    boolean validate(T param);
}

public class IntegerValidator implements Validator<Integer>{
    @Override
    public boolean validate(Integer param) {
        return param != null;
    }
}
```
你可能以为反编译的代码是这样的：
```java
public interface Validator {
    boolean validate(Object param);
}

public class IntegerValidator implements Validator {
    @Override
    public boolean validate(Integer param) {
        return param != null;
    }
}
```
然而如果真的是这样的话，此时`IntegerValidator`是无法运行的，因为它**实际创建的`validate(Integer param)`，没有覆盖`Validator`方法里的`validate(Object param)`**，因此Java编译器会添加一个桥接(bridge)方法来实现泛型的多态。以下是真正的`IntegerValidator`反编译后的代码：
```java
public class IntegerValidator
    implements Validator {

    public IntegerValidator() {
    }

    public boolean validate(Integer param) {
        return param != null;
    }

    // 这是真正的重写接口的方法，它调用了validate(Integer param)，起到了桥接的作用
    public volatile boolean validate(Object obj) {
        return validate((Integer)obj);
    }
}
```

**通过反射获取泛型的真正类型**

如果你试图通过`getTypeParameters`获取泛型的类型的话，那你将只会得到占位符：
```java
List<String> list = new ArrayList<>();
System.out.println(Arrays.asList(list.getClass().getTypeParameters()));
// 输出: [E]

Map<String, Integer> map = new HashMap<>();
System.out.println(Arrays.asList(map.getClass().getTypeParameters()));
// 输出：[K,V]
```
但是你可以通过`getGenericSuperclass`来获取泛型信息，但他并不是全能的，他的原理是通过定义在类上的签名，通过反射，获取泛型的类型，因此它只适用于这种情况：
```java
public class GiftBox extends Box<Gift> {
}

@Test
void testGenericType_predefined() {
    GiftBox box = new GiftBox();
    Type genericSuperclass = box.getClass().getGenericSuperclass();
    // 可以拿到，通过类签名GiftBox extends Box<Gift>，得到声明的泛型，然后反射拿到
    System.out.println(genericSuperclass);
    if (genericSuperclass instanceof ParameterizedType) {
        Type actualTypeArgument = ((ParameterizedType)genericSuperclass).getActualTypeArguments()[0];
        System.out.println(actualTypeArgument.getTypeName());
    }
}
/*
 输出：
 com.demo.generics.Box<com.demo.generics.Gift>
 com.demo.generics.Gift
 */
```
对于这种情况，它也还是拿不到的：
```java
@Test
void testGenericType_not_predefined() {
    Box<Gift> box = new Box<>();
    // 无法拿到，Box<Gift> 被类型擦除为Box，此时字节码中没有存在的签名使得可以通过反射得到泛型的类型
    Type genericSuperclass = box.getClass().getGenericSuperclass();
    System.out.println(genericSuperclass);
    if (genericSuperclass instanceof ParameterizedType) {
        Type actualTypeArgument = ((ParameterizedType)genericSuperclass).getActualTypeArguments()[0];
        System.out.println(actualTypeArgument.getTypeName());
    }
}
```

