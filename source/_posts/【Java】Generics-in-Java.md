---
title: 【Java】Generics in Java
subtitle: It’s all about type safety
author: lazytime
url_suffix: genericjava
tags:
  - Java
categories:
  - Java
keywords: generics in java
description: java中使用
copyright: true
date: 2023-07-27 11:32:00
---

# 原文

[Generics in Java. It’s all about type safety | by Salitha Chathuranga | Medium](https://medium.com/@salithachathuranga94/generics-in-java-3c791555e924)

# 引言

It’s all about type safety。

所有内容都是关于线程安全。

![](https://miro.medium.com/v2/resize:fit:875/1*77SYONEADgKEPiyVPqUb-A.png)

Hi all!!!

I thought of writing a widely used but less discussed topic in Java. That is **Generics**! We use it, but majority of the developers don’t know about it as I have experienced.

本次编写的有关内容是讨论较少主题，“**泛型**”，实际上我们经常使用（实际上天天都在用），但是大部分人并不了解。

Let me clear this…have you ever used List or **ArrayList** in Java? Most probably, answer should be YES. Right? Without collections, we can’t even think of an easy way of handling data. So, do you remember how we define an ArrayList?

让我澄清一下......您在Java中使用过List或**ArrayList**吗？答案很可能是 "是"。对不对？如果没有集合，我们甚至无法想象处理数据的简单方法。那么，你还记得我们是如何定义ArrayList的吗？

<!-- more -->

```java
List<Integer> numbers = new ArrayList<>(); // with Generics
```

This is the way we declare it. So, we have used generics. 😃 Here, `_<Integer>`_ is the Generic we passed. That is a Type. After we create list like this, you can **only add integers** to the list.

在上面的例子中我们在泛型中指定`<Integer>`，之后我们创建的List**只能添加**整型类型数据。

You may remember, if we define the list like below, we would be able to add any type of data which is extended from Object super class, to the list.

如果我们在定义List的时候不指定任何泛型，我们可以添加任意类型的数据，这些数据是从对象超类中扩展出来的。

```java
List numbers = new ArrayList(); // without Generics
```

We can achieve Type Safety for this List, after we add generics.

为了实现类型安全，之后我们可以添加泛型。

**Generics** means **parameterized types.** Java let us to create a single class, interface, and method that can be used with different types of data(objects) within the Generics domain.

泛型也叫参数化类型，Java允许我们创建单一的类、接口和方法，这些类、接口和方法可用于泛型域内的不同类型的数据（对象）。

**Advantages in Generics would be:**

泛型的优点如下：

- Code Reusability — we can use a common code with multiple object types

代码复用性：我们可以使用通用代码包含多种不同对象类型。

- Compile-time Type Checking — Java will check the generics code at the compile time against errors

编译时期的类型检查：实现Java在编译时期进行类型检查。

- Type Safety — we can restrict adding unnecessary data

类型安全：可以限制添加不必要的数据

- Usage in Collections — Collections need object types to deal with data

集合中使用：集合需要对象类型的数据，泛型可以更好的控制。

_Let’s take and example to explain why we need Generics.._

下面举个栗子介绍为什么需要泛型。

Imagine you have to print Numbers and Texts using a printer class. Printer has a method that accepts the data while creating it.

想象一下你需要数字或者文本类型的打印机对象，打印机在创建时有一个接受数据的方法。

In traditional way, we will have to creat 2 classes since we have 2 types of data: Number(Integer) and Text(String).

按照传统的方式，我们会创建两个对象，根据需要的打印机类型构建字符串和整型（打印机）对象。

```java
public class TextPrinter {  
    private final String data;  
  
    public TextPrinter(String data) {  
        this.data = data;  
    }  
  
    public void print() {  
        System.out.println("print::: " + data);  
    }  
}
```

```java
public class NumberPrinter {  
    private final Integer data;  
  
    public NumberPrinter(Integer data) {  
        this.data = data;  
    }  
  
    public void print() {  
        System.out.println("print::: " + data);  
    }  
}
```

How to use:

```java
public class GenericsMain {  
    public static void main(String[] args) {  
        NumberPrinter numberPrinter = new NumberPrinter(5);  
        numberPrinter.print(); // output = print::: 5  
        TextPrinter textPrinter = new TextPrinter("Hello");  
        textPrinter.print();   // output = print::: Hello  
    }  
}
```

You can see we have **code duplication**! Data type is the only difference here!

你会发现这里有重复代码，这里仅仅是对象类型不同。

We can simply use a Printer with a Generic here. Then we will only have 1 Printer! 😎

只需要非常简单的添加一个泛型，

Let’s deep dive into Generics and see how we achieve this… 😎

让我们深入了解泛型，看看如何实现这一点...... 

# Create a Generic 创建通用型

I’m taking the above simple example and will show how to create a **Generic** Printer.

我将以上面的简单示例来说明如何创建一个**通用**打印机。

```java
public class Printer<T> {  
    private final T data;  
  
    public Printer(T data) {  
        this.data = data;  
    }  
  
    public void print() {  
        System.out.println("print::: " + data);  
    }  
}
```

How to use:

```java
Printer<Integer> integerPrinter = new Printer<>(5);  
integerPrinter.print();   // output = print::: 5  
  
Printer<String> stringPrinter = new Printer<>("Hello");  
stringPrinter.print();   // output = print::: Hello  
  
Printer<Double> doublePrinter = new Printer<>(45.34);  
doublePrinter.print();   // output = print::: 45.34  
  
Printer<Long> longPrinter = new Printer<>(5L);  
longPrinter.print();z    // output = print::: 5
```

Now we only have 1 class! It accepts a Type. Here, T is used to denote the Type as a common standard. We can even create printer objects for other data types also like Double/Long. **Code reusability** is achieved in style. 😎

现在我们只需要一个类就可以完成构建两种不同类型的打印机，这里的 T 表示作为通用标准的类型，我们甚至可以把这个T改为  Double/Long 类型，最终实现了 “代码重用性” 的风格。

We can create Generic classes which accepts more than 1 type. Look at the below example. It accepts an Integer and a String both.

我们可以创建接受多种类型的通用类。请看下面的示例，它同时接受一个整数和一个字符串。

```java
public class MultiPrinter<T, V> {  
    private final T data1;  
    private final V data2;  
  
    public MultiPrinter(T data1, V data2) {  
        this.data1 = data1;  
        this.data2 = data2;  
    }  
  
    public void print() {  
        System.out.println("print::: " + data1 + " : " + data2);  
    }  
}
```

```java
MultiPrinter<Integer, String> multiPrinter = new MultiPrinter<>(5, "Hello");  
multiPrinter.print(); // output = print::: 5 : Hello
```

**Java Type Naming conventions**

- E — Element (used in Collections)
- K — Key (Used in Map)
- N — Number
- T — Type
- V — Value (Used in Map)
- S, U, V etc. — 2nd, 3rd, 4th types

**Java类型命名规则**

- E - 元素（在集合中使用）
- K - 键（在地图中使用）
- N - 数字
- T - 类型
- V - 值（在映射中使用）
- S、U、V 等 - 第二、第三、第四类型

# Bounded Generics 有限泛型

This is an advanced version of Generics. We can restrict more and achieve **more type safety** with bounded Generics.

这是泛型的高级版本。通过有界泛型，我们可以限制更多，实现**多的类型安全**。

Let’s say we have an **AnimalPrinter** class which can only print animal details. No other objects are allowed to be used with it. How to achieve this?

假设我们有一个**AnimalPrinter**类，它只能打印动物的详细信息。不允许使用其他对象。如何实现？

```java
public class Animal {  
    private final String name;  
    private final String color;  
    private final Integer age;  
  
    public Animal(String name, String color, Integer age) {  
        this.name = name;  
        this.color = color;  
        this.age = age;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public String getColor() {  
        return color;  
    }  
  
    public Integer getAge() {  
        return age;  
    }  
  
    @Override  
    public boolean equals(Object o) {  
        if (this == o) return true;  
        if (o == null || getClass() != o.getClass()) return false;  
        Animal animal = (Animal) o;  
        return Objects.equals(name, animal.name) && Objects.equals(color, animal.color) && Objects.equals(age, animal.age);  
    }  
  
    @Override  
    public int hashCode() {  
        return Objects.hash(name, color, age);  
    }  
}  

public class Cat extends Animal {  
    public Cat(String name, String color, Integer age) {  
        super(name, color, age);  
    }  
}  
  
public class Dog extends Animal {  
    public Dog(String name, String color, Integer age) {  
        super(name, color, age);  
    }  
}

public class AnimalPrinter<T extends Animal> {  
    private final T animalData;  
  
    public AnimalPrinter(T animalData) {  
        this.animalData = animalData;  
    }  
  
    public void print() {  
        System.out.println("Name::: " + animalData.getName());  
        System.out.println("Color::: " + animalData.getColor());  
        System.out.println("Age::: " + animalData.getAge());  
    }  
}

```

In this class, T extends Animal part does the job! We have limited our generic for Dog and Cat!

```java
AnimalPrinter<Cat> animalPrinter1 = new AnimalPrinter<>(new Cat("Jim", "brown", 2));  
animalPrinter1.print();  
AnimalPrinter<Dog> animalPrinter2 = new AnimalPrinter<>(new Dog("Rocky", "black", 5));  
animalPrinter2.print();

```

If we try to define the printer with another Object type, compiler will complain like this => _“Type parameter ‘java.lang.Object’ is not within its bound; should extend ‘generics.Animal’_

如果我们尝试使用其他对象类型定义打印机，编译器将发出如下警告 => _"类型参数'java.lang.Object'不在其绑定范围内；应该扩展'generics.Animal'_"。

# Multiple Bounds 多重边界

Let’s say we want to add some more features to the Printer generic. We can achieve it like this.

比方说，我们想为打印机通用程序添加更多的功能。我们可以这样实现

```java
public class AnimalPrinter<T extends Animal & Serializable> {  
    ..................  
}
```

I have provided Serializable functionality using Serializable interface. There are important things to remember here.

- We must implement interface in our child classes(Cat and Dog).
- Class should come first and the & and interface.
- Only 1 class can be extended since Java does not support multiple inheritance.

我使用Serializable接口提供了Serializable功能。这里有一些重要的事情需要记住。
- 我们必须在子类（Cat和Dog）中实现接口。
- 类应该放在前面，然后是 **&** 和 **接口**。
- 由于Java不支持多重继承，所以只能扩展一个类。

# Wildcards With Generics 通用通配符

Wildcards are represented by the question mark _?_ in Java, and we use them to refer to an unknown type. This can be used as a parameter type with Generics. Then it will accept any type. I have used a List of any object as a method argument using wild card, in the below code.

通配符在Java中用问号 _?_ 表示，我们用它来代指**未知类型**。通配符在Java中用问号 _?_ ，然后它将接受任何类型。在下面的代码中，我使用通配符将任意对象的List作为方法参数。

```java
public static void printList(List< ?> list) {  
    System.out.println(list);  
}  

printList(  
    Arrays.asList(  
        new Cat("Jim", "brown", 2),  
        new Dog("Rocky", "black", 5)  
    )  
);  
printList(Arrays.asList(50, 60));  
printList(Arrays.asList(50.45, 60.78));  
  
// output:  
// [generics.Cat@b1fa3959, generics.Dog@62294cd9]  
// [50, 60]  
// [50.45, 60.78]
```

List can be of any type now!!!

List 现在可以是任意类型的。

1️⃣ **Upper Bounded Wild Cards**

1️⃣ **上界通配符**

Consider this example:

考虑下面的例子

```java
public static void printAnimals(List<Animal> animals) {  
      animals.forEach(Animal::eat);  
}
```

If we imagine a subtype of _Animal_, such as a _Dog_, we can’t use this method with a list of _Dog_, even though _Dog_ is a subtype of _Animal_. We can do this with a wild card.

如果我们想象一个 _Animal_ 的子类型，例如 _Dog_ ，我们就不能在 _Dog_ 的列表中使用这个方法，尽管 _Dog_ 是 _Animal_ 的子类型。

我们可以使用通配符来实现。

```java
public static void printAnimals(List<? extends Animal> animals) {  
    ...  
}
```

Now this method works with type **_Animal_** and all **_its subtypes_**.

现在该方法适用于 **_Animal_**  类型和所有 **_子类型_**。

```java
printAnimals(  
    Arrays.asList(  
        new Cat("Jim", "brown", 2),  
        new Dog("Rocky", "black", 5)  
    )  
);
```

This is called an **upper-bounded wildcard**, where type **_Animal_** is the upper bound.

这被称为 **上界通配符** ，其中 **_Animal_** 类型是上界。

2️⃣ **Lower Bounded Wild Cards**

2️⃣ **下限通配符**

We can also specify wildcards with a lower bound, where the unknown type has to be a **super type of the specified type**. Lower bounds can be specified using the **_super_** keyword followed by the specific type.

我们还可以指定带有下限的通配符，其中未知类型必须是指定类型的 **超类型** 。可以使用 **_super_** 关键字指定下限，后面跟上特定的类型。

Example:

举例

```java
public static void addIntegers(List<? super Integer> list){  
    list.add(new Integer(70));  
}
```

# Generic Methods 通用方法

Imagine we need a method which takes different data types and do something. We can create a Generic method for this and reuse it.

想象一下，我们需要一个接收不同数据类型的方法来做一些事情。我们可以为此创建一个通用方法并重复使用。

```java
public static <T> void call(T data) {  
    System.out.println(data);  
}  

call("hello");  
call(45);  
call(15.67);  
call(5L);  
call(new Dog("Rocky", "black", 5));  
  
/* output:  
    hello  
    45  
    15.67  
    5  
    generics.Dog@62294cd9  
*/ 
```

If we want to return data instead of VOID, we can do that also.

如果我们想返回数据而不是VOID，我们也可以这样做。

```java
public static <T> T getData(T data) {  
    return data;  
}  
  
System.out.println(getData("Test"));   // output: Test
```

We can accept multiple data types also in a generic method.

我们也可以在泛型方法中接受多种数据类型。

```java
public static <T, V> void getMultiData(T data1, V data2) {  
    System.out.println("data 1: " + data1);  
    System.out.println("data 2: " + data2);  
}  

getMultiData(50, "Shades of Grey");
  
```

I think I have covered almost all the things to be learnt in Generics. So, this would be an ideal article for you to practice Generics in Java. ❤️

我认为已经覆盖了泛型使用的大部分场景. 因此，这将是您练习Java泛型的理想文章。❤️

I will bring you another Java stuff next time.

下次我会给您带来另一款Java产品。

Bye guys! 🙌

再见 🙌