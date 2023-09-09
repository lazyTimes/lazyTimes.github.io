---
title: 【Java】Effective Lambda Expressions in Java
subtitle: lambda的英文博文介绍
author: lazytime
url_suffix: effectivelambda
tags:
  - Java
categories:
  - Java
keywords: Lambda,Expressions
description: Effective Lambda Expressions
copyright: true
date: 2023-08-10 16:53:42
---
## 原文

[Effective Lambda Expressions in Java | by Bubu Tripathy | Medium](https://medium.com/@bubu.tripathy/effective-lambda-expressions-in-java-2d4061dde77a)

## Introductory

Lambda expressions were introduced in Java 8 to allow functional programming in Java. They are a concise way to express functions that can be used as data and provide a more functional approach to programming. Lambda expressions can be used in a variety of ways, from simple expressions to complex functions. In this article, we will discuss 20 best practices of using lambda expressions in Java with examples for each.

<!-- more -->

Lambda 表达式在 Java 8 中引入，允许在 Java 中进行函数式编程。Lambda表达式是表达可用作数据的函数的一种简洁方式，并为编程提供了一种功能性更强的方法。Lambda表达式的使用方式多种多样，从简单的表达式到复杂的函数。在本文中，我们将讨论在Java中使用lambda表达式的20个最佳实践，并分别举例说明。

## Use Lambda expressions to create functional Interfaces 使用Lambda表达式创建功能接口

A functional interface is an interface that contains a single abstract method. Functional interfaces are used extensively in Java to represent functions and Lambdas.

功能接口是一个包含单个抽象方法的接口。**功能接口**在Java中广泛用于表示函数和Lambdas。

> When using Lambda expressions to create functional interfaces, the Lambda expression is used to define the implementation of the abstract method. The Lambda expression takes the same parameters as the abstract method, and returns a value that represents the result of the method.
> 
> 当时用Lambda 表达式创建功能接口的时候，Lambda 表达式用于定义抽象方法实现。Lambda 表达式接受和抽象方法相同的参数，并且返回代表方法结果的值。

Here is an example of using Lambda expressions to create a functional interface for a calculator:

下面是使用Lambda 表达式来创建功能接口计算器的案例：

```java
@FunctionalInterface  
interface Calculator {  
    int calculate(int a, int b);  
}  
  
public class LambdaDemo {  
    public static void main(String[] args) {  
        Calculator add = (a, b) -> a + b;  
        Calculator subtract = (a, b) -> a - b;  
  
        System.out.println("10 + 5 = " + add.calculate(10, 5));  
        System.out.println("10 - 5 = " + subtract.calculate(10, 5));  
    }  
}
```

In this example, we are using a Lambda expression to define the implementation of the `calculate()` method of the Calculator interface. The `calculate()` method takes two integer values as input (a and b), and returns an integer value representing the result of the calculation.

在这个例子中，我们使用Lambda 表达式定义Calculator 接口的 `calculate()` 方法的实现。`calculate()` 方法接收两个整型参数作为输入（a 和 b），并且返回一个整型结果代表计算结果

Note that in this example, we are using the _@FunctionalInterface_ annotation to indicate that the Calculator interface is a functional interface. This annotation is not strictly necessary, but it can help to clarify the intent of the interface.

注意在这个例子中我们使用了 **@FunctionalInterface**  注解指定在 Calculator 接口的接口方法上面。这个注解严格上来来说是**非必要**的， 但它有助于明确界面的意图。

## Use the right syntax for Lambda Expressions 为Lambda表达式使用正确的语法

When using Lambda expressions in Java, it is important to use the correct syntax for defining and using them. The syntax of a Lambda expression consists of three parts: the parameter list, the arrow operator, and the body.

在Java中使用Lambda表达式， 使用正确的语法定义和使用它们非常重要的。Lambda表达式的语法由三部分组成：**参数列表**、**箭头操作符**和**主体**。

The parameter list [specifies] the input parameters that the Lambda expression will take. If there are no parameters, an empty parameter list must still be specified, as shown here:

参数列表指定了Lambda表达式将使用的输入参数，如果没有任何参数，一个空的参数化列表依然必须被指定，它的表现形式如下：

```java
() -> System.out.println("Hello, world!");
```

The arrow operator separates the parameter list from the body of the Lambda expression. The arrow operator can be either a hyphen and greater-than symbol (->), or the word “goes to” written as an equals sign followed by a greater-than symbol (=>).

箭头操作符将参数列表与Lambda表达式的主体分开，箭头运算符可以是`-`字符和大于号组合(->)，也可以是等号和大于号组合(=>)。

```java
// hyphen and greater-than symbol  
x -> x * x  
```

```java
// equals sign followed by a greater-than symbol  
(x, y) => x + y
```

The body of the Lambda expression specifies the behavior of the expression. The body can be a single expression, or a block of statements enclosed in braces. If the body is a single expression, the braces are optional.

Lambda 表达式的主体指定表达式的行为，主题可以是一个单独的表达式，也可以是用大括号括起来的语句块，如果主体是单个表达式，则大括号是可选的。

函数表达式

```java
// single expression  
x -> x * x  
```

语句块

```java
// block of statements  
(x, y) -> {  
    int sum = x + y;  
    System.out.println("The sum is: " + sum);  
}
```

It is also important to use the correct type for Lambda expressions when they are assigned to variables or parameters. When a Lambda expression is assigned to a variable or parameter, the type of the variable or parameter must be a functional interface that is compatible with the Lambda expression.

当Lambda表达式设置变量或者参数的时候，使用正确的类型也是十分重要。当一个Lambda 表达式设置了变量或者参数，变量或参数必须是与**Lambda表达式兼容的功能接口**。

For example, the following Lambda expression:

比如下面这个函数表达式的例子：

```java
(x, y) -> x + y
```

can be assigned to a variable of type _IntBinaryOperator_:

这里只能够设置变量类型为 _IntBinaryOperator_ ：

```java
IntBinaryOperator add = (x, y) -> x + y;
```

## Use Lambda expressions to implement Functional Programming 使用 Lambda 表达式实现函数式编程

Functional programming is a programming paradigm that emphasizes the use of functions and functional composition to solve problems. In Java, Lambda expressions can be used to implement functional programming by creating and using functions as first-class objects.

函数式编程是一种编程范式，强调使用函数和函数组合来解决问题。在Java中，可以使用Lambda表达式来实现函数式编程，将函数作为头等对象来创建和使用。

> Functions in functional programming are defined by their behavior, rather than their implementation. This means that a function can be treated as an object, and passed around as a parameter or returned as a result. This is where Lambda expressions come in — they allow us to define functions as expressions, and pass them around as objects.
> 
> 函数式编程当中行为是通过具体行为而不是实现定义的，也就是说函数可以是对象，并且把参数传递作为结果返回。Lambda表达式的强大之处就在于此，可以把函数作为表达式，并且把对应的对象进行传递。

Here is an example of using Lambda expressions to implement functional programming in Java:

下面是使用Lambda表达式使Java实现函数式编程。

```java
public class FunctionalProgrammingDemo {  
    public static void main(String[] args) {  
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);  
        numbers.stream()  
            .filter(n -> n % 2 == 0)  
            .map(n -> n * n)  
            .forEach(System.out::println);  
    }  
}
```

In this example, we are using Lambda expressions to implement functional programming in Java. The code first creates a list of integers, and then uses a stream to process the list using functional operations.

在本示例中，我们使用Lambda表达式在Java中实现函数式编程。代码首先创建一个整数列表，然后使用`stream`的函数式操作来处理该列表。

The filter() operation uses a Lambda expression to define a predicate that filters out all odd numbers from the list. The map() operation uses another Lambda expression to define a function that squares each even number in the list. Finally, the _forEach()_ operation uses a method reference to print each squared even number to the console.

`filter()` 操作使用 Lambda 表达式定义一个`predicate`，`predicate`中定义了过滤列表所有奇数的操作，`map() `操作使用另一个Lambda表达式定义一个函数，该函数将列表中的每个元素求平方，最后使用 forEach 的方式，把每个处理之后的数据打印出来。

By using Lambda expressions to implement functional programming, you can create more expressive and modular code, and solve problems in a more concise and efficient way. Lambda expressions provide a powerful and flexible way to implement functional programming in Java and are a key feature of the Stream API in Java 8 and later versions.

使用函数表达式实现函数式编程，你可以创建更多更具表现力和模块化的代码，解决问你也会更加的简单高效，Lambda表达his提供一个强大并且流畅的方法在Java中实现函数式编程，这是Java 8及更高版本中流API的关键特性。

## Use Lambda expressions to create Anonymous Classes 使用 Lambda 表达式创建匿名类

Anonymous classes are often used in Java to create objects with a specific behavior, such as event listeners or runnable tasks.

在Java中，匿名类通常用于创建具有特定行为的对象，如事件监听器或可运行任务。

Lambda expressions can be used to create anonymous classes in Java. When using Lambda expressions to create anonymous classes, the Lambda expression is used to define the behavior of the anonymous class.

Lambda表达式可用于在Java中创建匿名类。使用Lambda表达式创建匿名类时，Lambda表达式用于定义匿名类的行为。

Here is an example of using Lambda expressions to create an anonymous class that implements the Runnable interface:

下面是一个使用Lambda表达式创建一个实现Runnable接口的匿名类的示例：

```java
public class AnonymousClassDemo {  
    public static void main(String[] args) {  
        Runnable runnable = () -> {  
            System.out.println("Hello from anonymous class!");  
        };  
        Thread thread = new Thread(runnable);  
        thread.start();  
    }  
}
```

In this example, we are using a Lambda expression to create an anonymous class that implements the Runnable interface. The code first defines a Lambda expression that simply prints a message to the console. This Lambda expression is then used to create an anonymous class that implements the Runnable interface. Finally, the anonymous class is passed to a Thread object and started.

在这个案例中，我们通过Lambda表达式创建Runnable的匿名类，代码首先定义了一个Lambda表达式，该表达式简单地向控制台打印一条消息。然后使用该Lambda表达式创建一个实现Runnable接口的匿名类。最后，匿名类被传递给Thread对象并启动。

Note that in this example, we are using a Lambda expression with a block of statements enclosed in braces. This is because the Runnable interface has a single abstract method, run(), that takes no arguments and returns no value. Therefore, the Lambda expression must define the behavior of the run() method using a block of statements.

请注意，在本示例中，我们使用的是一个Lambda表达式，其中的语句块用大括号括起来。这是因为**Runnable接口只有一个抽象方法run()**，它不需要参数也不返回值。因此，Lambda表达式必须使用语句块来定义run()方法的行为。

## Use Lambda expressions to create Event Listeners 使用 Lambda 表达式创建事件监听器

Event listeners are used in Java to handle events that occur in a user interface or other system. Events can include things like button clicks, mouse movements, and keyboard inputs.

Java中的事件监听器用于处理用户界面或其他系统中发生的事件。

事件包括按钮点击、鼠标移动和键盘输入等。

> When using Lambda expressions to implement event listeners, the Lambda expression is used to define the behavior that should be executed when the event occurs. The Lambda expression takes an event object as input, and performs some action based on the event.
> 
> 当我们使用Lambda表达式实现事件监听器，通常需要用于定义事件发生时应执行的行为。Lambda表达式接收一个事件对象作为输入，并且根据事件的执行某些操作。

Here is an example of using Lambda expressions to implement an event listener for a button click:

下面是一个使用 Lambda 表达式实现按钮点击事件监听器的示例：

```java
public class ButtonDemo {  
    public static void main(String[] args) {  
        JButton button = new JButton("Click me!");  
        button.addActionListener(event -> System.out.println("Button clicked!"));  
        JFrame frame = new JFrame();  
        frame.add(button);  
        frame.pack();  
        frame.setVisible(true);  
    }  
}
```

In this example, we are using a Lambda expression to define the behavior that should be executed when the button is clicked. The addActionListener() method of the JButton class takes a ActionListener object as input, which is implemented as a Lambda expression in this case. The Lambda expression simply prints a message to the console when the button is clicked.

在本示例中，我们使用Lambda表达式来定义点击按钮时应执行的行为。JButton类的addActionListener()方法将ActionListener对象作为输入，在本例中以Lambda表达式的形式实现。当按钮被点击时，Lambda表达式简单地向控制台打印一条消息。

Note that in this example, we are using the “arrow” notation (->) to separate the parameter list from the body of the Lambda expression. The parameter list consists of a single event object, which represents the button click event. The body of the Lambda expression simply prints a message to the console.

请注意，在本例中，我们使用了 "箭头 "符号 (->) 来分隔参数列表和 Lambda 表达式的主体。参数列表由单个字符串值组成，代表列表中的每个字符串。Lambda 表达式的主体只是将字符串打印到控制台。

## Use Lambda expressions to Iterate over Collections 使用 Lambda 表达式遍历集合

Iterating over collections is a common operation in Java, and Lambda expressions can be used to simplify and streamline this process. When using Lambda expressions to iterate over collections, the Lambda expression is used to define the behavior that should be executed for each element in the collection.

在 Java 中，遍历集合是一种常见操作，而 Lambda 表达式可用于简化和精简这一过程。使用 Lambda 表达式遍历集合时，Lambda 表达式用于定义应针对集合中的每个元素执行的行为。

Here is an example of using Lambda expressions to iterate over a list of strings and print each string to the console:

下面是一个使用 Lambda 表达式遍历字符串列表并将每个字符串打印到控制台的示例：

```java
List<String> strings = Arrays.asList("apple", "banana", "cherry");  
strings.forEach(s -> System.out.println(s));
```

In this example, we are using the forEach() method of the List interface to iterate over a list of strings. The forEach() method takes a Consumer object as input, which is implemented as a Lambda expression in this case. The Lambda expression simply prints each string to the console.

在本示例中，我们使用 List 接口的 forEach() 方法遍历字符串列表。forEach()方法将Consumer对象作为输入，在本例中以Lambda表达式的形式实现。Lambda 表达式简单地将每个字符串打印到控制台。

Note that in this example, we are using the “arrow” notation (->) to separate the parameter list from the body of the Lambda expression. The parameter list consists of a single string value, which represents each string in the list. The body of the Lambda expression simply prints the string to the console.

请注意，在本例中，我们使用了 "箭头 "符号 (->) 来分隔参数列表和 Lambda 表达式的主体。参数列表由单个字符串值组成，代表列表中的每个字符串。Lambda 表达式的主体只是将字符串打印到控制台。

Here is another example of using Lambda expressions to iterate over a map of key-value pairs and print each pair to the console:

下面是另一个使用 Lambda 表达式遍历键值对映射并将每一对键值对打印到控制台的示例：

```java
Map<String, Integer> map = new HashMap<>();  
map.put("apple", 1);  
map.put("banana", 2);  
map.put("cherry", 3);  
```

```java
map.forEach((k, v) -> System.out.println(k + " -> " + v));
```

In this example, we are using the _forEach()_ method of the Map interface to iterate over a map of key-value pairs. The forEach() method takes a _BiConsumer_ object as input, which is implemented as a Lambda expression in this case. The Lambda expression takes two parameters, a key and a value, and simply prints each key-value pair to the console.

在本例中，我们使用 `Map` 接口的 _forEach()_ 方法遍历键值对映射。forEach()方法将 _BiConsumer_ 对象作为输入，在本例中是以 Lambda 表达式的形式实现的。

Lambda 表达式接收两个参数，一个键和一个值，并简单地将每个键值对打印到控制台。

Note that in this example, we are using the “arrow” notation (->) to separate the parameter list from the body of the Lambda expression. The parameter list consists of two values, a key and a value, which represent each key-value pair in the map. The body of the Lambda expression simply prints the key-value pair to the console.

请注意，在这个示例中，我们使用了 "箭头 "符号 (->) 来分隔参数列表和 Lambda 表达式的主体。参数列表由两个值（一个 key 和一个 value）组成，分别代表 map 中的每个键值对。Lambda 表达式的主体只是将键值对打印到控制台。

## Use Lambda expressions to Filter Collections 使用 Lambda 表达式过滤集合

Filtering collections is a common operation in Java, and Lambda expressions can be used to simplify and streamline this process. When using Lambda expressions to filter collections, the Lambda expression is used to define a predicate that selects which elements in the collection should be included or excluded.

过滤集合是 Java 中的一种常见操作，而 Lambda 表达式可用于简化和精简这一过程。使用 Lambda 表达式过滤集合时，Lambda 表达式用于定义一个谓词，该谓词用于选择应包含或排除集合中的哪些元素。

Here is an example of using Lambda expressions to filter a list of integers and create a new list that contains only the even numbers:

下面是一个使用 Lambda 表达式过滤整数列表并创建一个只包含偶数的新列表的示例：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);  
List<Integer> evenNumbers = numbers.stream()  
                                    .filter(n -> n % 2 == 0)  
                                    .collect(Collectors.toList());  
System.out.println(evenNumbers);
```

In this example, we are using the filter() method of the Stream interface to filter a list of integers. The filter() method takes a predicate as input, which is implemented as a Lambda expression in this case. The Lambda expression defines a predicate that tests whether a number is even or not by checking if its remainder is zero when divided by two.

在本例中，我们使用流接口的 filter() 方法过滤整数列表。filter() 方法将一个谓词作为输入，在本例中以 Lambda 表达式的形式实现。Lambda 表达式定义了一个谓词，该谓词通过检查一个数字除以 2 后的余数是否为零来检验该数字是否为偶数。

Note that in this example, we are using the “arrow” notation (->) to separate the parameter list from the body of the Lambda expression. The parameter list consists of a single integer value, which represents each element in the stream. The body of the Lambda expression is a boolean expression that defines the predicate.

请注意，在这个示例中，我们使用了 "箭头 "符号 (->) 来分隔参数列表和 Lambda 表达式的主体。参数列表由单个整数值组成，代表流中的每个元素。Lambda 表达式的主体是定义谓词的布尔表达式。

The filtered elements are then collected into a new list using the collect() method of the Stream interface, which takes a Collector object as input. The Collector object in this case is the toList() method of the Collectors class, which creates a new list of the filtered elements.

然后，使用流接口的 collect() 方法将过滤后的元素收集到一个新的列表中，该方法将收集器对象作为输入。本例中的收集器对象是收集器类的 toList() 方法，它将创建一个包含过滤元素的新列表。

## Use Lambda expressions to Sort Collections 使用 Lambda 表达式对集合进行排序

Sorting is the process of arranging the elements of a collection in a specific order. In Java, collections can be sorted using the _sorted()_ method of the Stream API.

排序是将集合中的元素按特定顺序排列的过程。在Java中，可以使用流API的 _sorted()_ 方法对集合进行排序。 

> When using Lambda expressions to sort collections, the Lambda expression is used to define the _order_ in which the elements should be sorted. 
> 
> 使用 Lambda 表达式对集合进行排序时，Lambda 表达式用于定义元素的排序顺序。
> 
> The Lambda expression takes two elements from the collection as input, and returns an integer value indicating their relative order. 
> 
> Lambda 表达式将集合中的两个元素作为输入，并返回一个整数值，表示它们的相对顺序。
> 
> If the first element should come before the second element in the sorted collection, the Lambda expression returns a negative integer. 
> 
> 如果在排序集合中，第一个元素应在第二个元素之前，则 Lambda 表达式会返回一个负整数。
> 
> If the first element should come after the second element, the Lambda expression returns a positive integer. If the two elements are equal, the Lambda expression returns zero.
> 
> 如果第一个元素位于第二个元素之后，则 Lambda 表达式返回一个正整数。如果两个元素相等，则 Lambda 表达式返回 0。

Here is an example of using Lambda expressions to sort a list of integers:

下面是一个使用 Lambda 表达式对整数列表进行排序的示例：

```java
List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5);  
List<Integer> sortedNumbers = numbers.stream().sorted().collect(Collectors.toList());
```

In this example, we are using a Lambda expression to sort the list of integers in ascending order. The sorted() method uses this Lambda expression to create a new stream that contains the elements of the original list in sorted order.

在这个示例中，我们使用 Lambda 表达式按升序对整数列表进行排序。

sorted()方法使用该Lambda表达式创建一个新的流，其中包含按排序顺序排列的原始列表元素。

Note that in this example, we are not explicitly defining a Lambda expression for the sorted() method. This is because the default behavior of the sorted() method is to sort the elements in natural order. However, you can also provide a custom Lambda expression to the sorted() method to define a custom sort order.

请注意，在这个示例中，我们没有为 sorted() 方法明确定义一个 Lambda 表达式。这是因为 sorted() 方法的默认行为是按自然顺序对元素进行排序。

不过，您也可以为 sorted() 方法提供自定义 Lambda 表达式，以定义自定义排序顺序。

Here is an example of using a Lambda expression to sort a list of strings in descending order:

下面是一个使用 Lambda 表达式对字符串列表进行降序排序的示例：

```java
List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");  
List<String> sortedWords = words.stream().sorted((a, b) -> b.compareTo(a)).collect(Collectors.toList());
```

In this example, we are using a Lambda expression to sort the list of strings in descending order. The Lambda expression takes two strings as input, and returns the result of comparing the second string to the first string using the _compareTo()_ method of the String class.

在本例中，我们使用 Lambda 表达式对字符串列表进行降序排序。Lambda 表达式将两个字符串作为输入，并使用 String 类的 _compareTo()_ 方法返回第二个字符串与第一个字符串的比较结果。
## Use Lambda expressions to Map Collections

Mapping is the process of transforming one collection into another. In Java, collections can be mapped using the map() method of the Stream API.

Map是将一个集合转换成另一个集合的过程。在 Java 中，可以使用流 API 的 map() 方法对集合进行映射。

> When using Lambda expressions to map collections, the Lambda expression is used to define the transformation that should be applied to each element in the collection. The Lambda expression takes an element from the collection as input and returns a new value that represents the transformed element.
> 
> 使用 Lambda 表达式映射集合时，Lambda 表达式用于定义应用于集合中每个元素的转换。Lambda 表达式将集合中的元素作为输入，并返回一个代表转换后元素的新值。

Here is an example of using Lambda expressions to map a list of strings to a list of integers:

```java
List<String> strings = Arrays.asList("1", "2", "3", "4", "5");  
List<Integer> integers = strings.stream().map(Integer::parseInt).collect(Collectors.toList());
```

In this example, we are using a Lambda expression to map the list of strings to a list of integers. The Lambda expression applies the parseInt() method of the Integer class to each string in the list, converting it to an integer value. The map() method then uses this Lambda expression to create a new stream that contains the transformed elements.

在本例中，我们使用 Lambda 表达式将字符串列表映射为整数列表。Lambda 表达式对列表中的每个字符串应用 Integer 类的 parseInt() 方法，将其转换为整数值。然后，map() 方法使用此 Lambda 表达式创建一个包含转换后元素的新流。

Note that in this example, we are using the method reference notation (::) to refer to the parseInt() method of the Integer class. This is a shorthand notation for a Lambda expression that simply calls a single method.

请注意，在本例中，我们使用方法引用符号（::）来引用 Integer 类的 parseInt() 方法。这是一个 Lambda 表达式的简写符号，只需调用一个方法即可。

Here is another example of using a Lambda expression to map a list of strings to a list of uppercase strings:

下面是另一个使用 `Lambda` 表达式将字符串列表映射为大写字符串列表的示例：

```java
List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");  
List<String> uppercaseWords = words.stream().map(s -> s.toUpperCase()).collect(Collectors.toList());
```

In this example, we are using a Lambda expression to map the list of strings to a list of uppercase strings. The Lambda expression applies the toUpperCase() method of the String class to each string in the list, converting it to an uppercase string. The map() method then uses this Lambda expression to create a new stream that contains the transformed elements.

在本例中，我们使用 Lambda 表达式将字符串列表映射为大写字符串列表。Lambda 表达式将 String 类的 toUpperCase() 方法应用于列表中的每个字符串，将其转换为大写字符串。然后，map() 方法使用此 Lambda 表达式创建一个包含转换后元素的新流。
## Use Lambda expressions to Reduce Collections 使用 Lambda 表达式精简集合

Reducing is the process of applying an operation to all the elements of a collection to produce a single result. In Java, collections can be reduced using the reduce() method of the Stream API.

`Reducing` 是对集合中的所有元素进行操作以产生单一结果的过程。在 Java 中，可以使用流 API 的 `reduce() `方法对集合进行还原。

> When using Lambda expressions to reduce collections, the Lambda expression is used to define the operation that should be applied to each element in the collection. The Lambda expression takes two arguments as input: an accumulator and an element from the collection. The accumulator is the result of the previous operation, or an initial value if this is the first operation. The Lambda expression returns a new value that represents the result of applying the operation to the accumulator and the current element.
> 
> 使用 Lambda 表达式还原集合时，Lambda 表达式用于定义应用于集合中每个元素的操作。Lambda 表达式将两个参数作为输入：累加器和集合中的一个元素。累加器是前一次操作的结果，如果是第一次操作，则是初始值。Lambda 表达式返回一个新值，该值代表对累加器和当前元素应用操作的结果。

Here is an example of using Lambda expressions to reduce a list of integers to a single sum:

下面是一个使用 Lambda 表达式将整数列表简化为单个和的示例：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);  
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
```

In this example, we are using a Lambda expression to reduce the list of integers to a single sum. The Lambda expression takes two integer values as input: an accumulator (initialized to zero in this case), and an element from the list. The Lambda expression adds the element to the accumulator, and returns the new value of the _accumulator_. The reduce() method then uses this Lambda expression to apply the operation to each element in the list, producing the final sum.

在本例中，我们使用 Lambda 表达式将整数列表还原为单个和。Lambda 表达式将两个整数值作为输入：一个累加器（本例中初始化为零）和列表中的一个元素。Lambda 表达式将元素加到累加器中，并返回_accumulator_的新值。然后，reduce() 方法使用此 Lambda 表达式对列表中的每个元素进行运算，得出最终总和。

Note that in this example, we are using the “arrow” notation (->) to separate the parameter list  from the body of the Lambda expression. The parameter list consists of two integer values: the accumulator and the current element from the list. The body of the Lambda expression adds the current element to the accumulator using the + operator.

请注意，在本例中，我们使用了 "箭头 "符号 (->) 来分隔参数列表和 Lambda 表达式的主体。参数列表由两个整数值组成：累加器和列表中的当前元素。Lambda 表达式的主体使用 + 运算符将当前元素添加到累加器中。

Here is another example of using a Lambda expression to reduce a list of strings to a single concatenated string:

下面是另一个使用 Lambda 表达式将字符串列表缩减为单个连接字符串的示例：

```java
List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");  
String concatenated = words.stream().reduce("", (a, b) -> a + b);
```

In this example, we are using a Lambda expression to reduce the list of strings to a single concatenated string. The Lambda expression takes two string values as input: an accumulator (initialized to an empty string in this case), and an element from the list. The Lambda expression concatenates the element to the accumulator, and returns the new value of the accumulator. The reduce() method then uses this Lambda expression to apply the operation to each element in the list, producing the final concatenated string.

在本例中，我们使用 Lambda 表达式将字符串列表还原为单个连接字符串。Lambda 表达式将两个字符串值作为输入：一个累加器（本例中初始化为空字符串）和列表中的一个元素。Lambda 表达式将元素连接到累加器，并返回累加器的新值。然后，reduce() 方法使用此 Lambda 表达式对列表中的每个元素进行操作，生成最终的连接字符串。

## Use Lambda expressions to Group Collections  使用 Lambda 表达式对集合进行分组

Grouping is the process of grouping the elements of a collection based on a common property or criterion. In Java, collections can be grouped using the _groupingBy()_ method of the Collectors class.

分组是根据共同属性或标准对集合中的元素进行分组的过程。在 Java 中，可以使用 Collectors 类的 _groupingBy()_ 方法对集合进行分组。

> When using Lambda expressions to group collections, the Lambda expression is used to define the criterion that should be used to group the elements. The Lambda expression takes an element from the collection as input, and returns a value that represents the grouping criterion.
> 
> 使用 Lambda 表达式对集合进行分组时，Lambda 表达式用于定义对元素进行分组的标准。Lambda 表达式将集合中的元素作为输入，并返回一个代表分组标准的值。

Here is an example of using Lambda expressions to group a list of words by their length:

下面是一个使用 Lambda 表达式按单词长度分组的示例：

```java
List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");  
Map<Integer, List<String>> groups = words.stream().collect(Collectors.groupingBy(String::length));
```

In this example, we are using a Lambda expression to group the list of words by their length. The groupingBy() method uses this Lambda expression to create a map that groups the words by their length. The key of each entry in the map is an integer value representing the length of the words, and the value of each entry is a list of words with that length.

在本例中，我们使用 `Lambda` 表达式按单词长度对单词列表进行分组。`groupingBy()` 方法使用此 `Lambda` 表达式创建一个按单词长度分组的映射。映射中每个条目的键是代表单词长度的整数值，每个条目的值是具有该长度的单词列表。

Note that in this example, we are using the method reference notation (::) to refer to the length() method of the String class. This is a shorthand notation for a Lambda expression that simply calls a single method.

请注意，在本例中，我们使用方法引用符号（`::`）来引用 `String` 类的 `length()` 方法。这是简单调用单个方法的 Lambda 表达式的速记符号。

Here is another example of using a Lambda expression to group a list of employees by their department:

下面是另一个使用 Lambda 表达式按部门对员工列表进行分组的示例：

```java
List<Employee> employees = Arrays.asList(  
    new Employee("Alice", "Sales"),  
    new Employee("Bob", "Marketing"),  
    new Employee("Charlie", "Sales"),  
    new Employee("Dave", "Marketing"),  
    new Employee("Eve", "HR")  
);  

Map<String, List<Employee>> groups = employees.stream().collect(Collectors.groupingBy(Employee::getDepartment));
```

In this example, we are using a Lambda expression to group the list of employees by their department. The groupingBy() method uses this Lambda expression to create a map that groups the employees by their department. The key of each entry in the map is a string value representing the department of the employees, and the value of each entry is a list of employees in that department.

在本例中，我们使用 Lambda 表达式按部门对员工列表进行分组。groupingBy()方法使用此 Lambda 表达式创建了一个按部门对员工进行分组的映射。映射中每个条目的键是代表员工部门的字符串值，每个条目的值是该部门的员工列表。
## Use Lambda expressions to Handle Exceptions 使用 Lambda 表达式处理异常

Lambda expressions can be used to handle checked exceptions in Java. Checked exceptions are a type of exception that must be declared in a method’s signature or handled by the caller. When using Lambda expressions to handle checked exceptions, the Lambda expression is used to define the behavior that should be executed in case of an exception, while still allowing the checked exception to be propagated up to the caller.

Lambda 表达式可用于处理 Java 中的校验异常。校验异常是一种必须在方法签名中声明或由调用者处理的异常类型。使用 Lambda 表达式处理校验异常时，Lambda 表达式用于定义异常情况下应执行的行为，同时仍允许将校验异常传播给调用者。

Here is an example of using Lambda expressions to handle a checked exception when reading a file:

下面是一个使用 Lambda 表达式处理读取文件时已检查异常的示例：

```java
List<String> lines = null;  
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {  
    lines = reader.lines()  
                  .map(line -> {  
                      try {  
                          return process(line);  
                      } catch (IOException e) {  
                          throw new UncheckedIOException(e);  
                      }  
                  })  
                  .collect(Collectors.toList());  
} catch (IOException e) {  
    e.printStackTrace();  
}  

private static String process(String line) throws IOException {  
    // process the line  
}  
```

In this example, we are using a Lambda expression to handle a checked exception when processing each line of a file. The code first creates a _BufferedReader_ object that reads from a file named “file.txt”. The lines from the file are then processed using the map() method of the Stream interface. The map() method takes a Function object as input, which is implemented as a Lambda expression in this case. The Lambda expression calls the process() method to process each line, and catches any IOException that may occur. If an IOException occurs, the Lambda expression throws an UncheckedIOException, which is a runtime exception that wraps the original checked exception.

在本示例中，我们使用 Lambda 表达式来处理处理文件每一行时检查到的异常。代码首先创建了一个 _BufferedReader_ 对象，用于读取名为 "file.txt "的文件。

然后使用 Stream 接口的 map() 方法处理文件中的行。

map() 方法将一个 Function 对象作为输入，在本例中实现为一个 Lambda 表达式。Lambda 表达式调用 process() 方法处理每一行，并捕获可能出现的任何 IOException。

如果出现 IO 异常，Lambda 表达式会抛出一个未检查的 IO 异常，这是一个运行时异常，它封装了原始的已检查异常。

Note that in this example, we are using a Lambda expression with a block of statements enclosed in braces. This is because the _process()_ method throws an _IOException_, which must be caught or re-thrown by the Lambda expression.

请注意，在这个示例中，我们使用的是一个 Lambda 表达式和一个用大括号括起来的语句块。这是因为 _process()_ 方法会抛出一个_IOException_，而 Lambda 表达式必须捕获或重新抛出这个 _IOException_ 。

By using Lambda expressions to handle checked exceptions, you can create more concise and readable code, and handle exceptions with greater flexibility and modularity. Lambda expressions provide a powerful and flexible way to handle checked exceptions in Java, and can be used in a wide range of scenarios, such as file I/O, network communication, and database access.

通过使用 `Lambda` 表达式来处理校验异常，您可以创建更简洁、更易读的代码，并以更大的灵活性和模块化来处理异常。Lambda 表达式为在 Java 中处理检查异常提供了一种强大而灵活的方法，可用于文件 I/O、网络通信和数据库访问等多种场景。
## Use Lambda expressions to Handle Null Values 使用 Lambda 表达式处理空值

In Java, NullPointerExceptions (NPEs) can often occur when dealing with null values. Lambda expressions can be used to handle null values in a more concise and expressive way, and to prevent _NullPointerExceptions_ from occurring. Here is an example of using Lambda expressions to handle null values when filtering a list of strings:

在 Java 中，处理空值时经常会出现 `NullPointerException（NPE）`。可以使用 Lambda 表达式以更简洁、更具表现力的方式处理空值，并防止发生 _NullPointerException_ 异常。下面是一个在过滤字符串列表时使用 Lambda 表达式处理空值的示例：

```java
List<String> list = Arrays.asList("apple", null, "banana", "cherry", null);  
List<String> filteredList = list.stream()  
                                .filter(s -> s != null)  
                                .collect(Collectors.toList());  
System.out.println(filteredList);  
```

In this example, we are using the _filter()_ method of the Stream interface to filter a list of strings. The filter() method takes a Predicate object as input, which is implemented as a Lambda expression in this case. The Lambda expression defines a predicate that tests whether a string is not null.

在本例中，我们使用 Stream 接口的 _filter()_ 方法过滤字符串列表。filter()方法将一个谓词对象作为输入，在本例中是以 Lambda 表达式的形式实现的。Lambda 表达式定义了一个谓词，用于测试字符串是否为空。

Note that in this example, we are using the != operator to test for null values. This is because the null value is not equal to any other value, including null itself.

请注意，在本例中，我们使用 != 操作符来测试空值。这是因为null值不等于任何其他值，包括null值本身。
## Use Lambda expressions to perform Parallel Operations 使用 Lambda 表达式执行并行操作

Lambda expressions are anonymous functions in Java that can be used to perform a variety of operations, including parallel operations.

Lambda 表达式是 Java 中的匿名函数（但是细节上和真正的匿名函数不一样），可用于执行各种操作，包括并行操作。

> Parallelism refers to the ability to perform multiple operations simultaneously, thereby reducing the amount of time it takes to complete a task. In Java, parallelism can be achieved using the Stream API and lambda expressions.
> 
> 并行性是指同时执行多个操作的能力，从而减少完成任务所需的时间。在 Java 中，可以使用流 API 和 lambda 表达式来实现并行性

The Stream API provides a _parallelStream()_ method that allows you to create parallel streams. Streams are collections of objects that can be processed sequentially or in parallel. By default, streams are processed sequentially, but you can use the _parallelStream()_ method to process them in parallel.

流 API 提供了一个 _parallelStream()_ 方法，允许您创建并行流。

流是可以按顺序或并行处理的对象集合。默认情况下，流是按顺序处理的，但您可以使用 _parallelStream()_ 方法并行处理它们。

To use lambda expressions to perform parallel operations, you first need to create a stream using the parallelStream() method. You can then use lambda expressions to define the operations that are performed on the elements of the stream. **The Stream API will automatically split the stream into multiple substreams and distribute them across multiple threads, allowing the operations to be performed in parallel.**

要使用 lambda 表达式执行并行操作，首先需要使用 parallelStream() 方法创建一个流。然后，就可以使用 lambda 表达式定义对流元素执行的操作。 **流 API 会自动将流拆分成多个子流，并将它们分配给多个线程，从而允许并行执行操作。

Here’s an example of using lambda expressions to perform a parallel operation:

下面是一个使用 lambda 表达式执行并行操作的示例：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);  

int sum = numbers.parallelStream()  
                .filter(n -> n % 2 == 0)  
                .mapToInt(n -> n)  
                .sum();
```

## Use Lambda expressions to create Custom Collectors  使用 Lambda 表达式创建自定义收集器

A collector is an operation that can be performed on a stream to accumulate the elements of the stream into a final result. Collectors are used with the _collect()_ method of the Stream interface.

收集器是一种可在流上执行的操作，用于将流的元素累加为最终结果。收集器与流接口的 _collect()_ 方法一起使用。

> Java provides a number of built-in collectors that perform common operations like grouping elements, counting elements, and calculating averages. However, you can also create your own custom collectors using lambda expressions.
> 
> Java 提供了许多内置收集器，可执行分组元素、计数元素和计算平均值等常见操作。
> 不过，您也可以使用 lambda 表达式创建自己的自定义收集器。

To create a custom collector using lambda expressions, you need to define a new class that implements the _Collector_ interface. The Collector interface has four methods that you need to implement:

要使用 lambda 表达式创建自定义收集器，需要定义一个实现 _Collector_ 接口的新类。收集器接口有四个方法需要实现：

- The _supplier()_ method returns a function that creates a new mutable result container.
- The _accumulator()_ method returns a function that adds an element to the result container.
- The _combiner()_ method returns a function that combines two result containers.
- The _finisher()_ method returns a function that performs a final transformation on the result container.

- _supplier()_： 方法返回一个函数，用于创建一个新的可变结果容器。
- _Accumulator()_：  方法返回一个将元素添加到结果容器的函数。
- _combiner()_：方法返回一个将两个结果容器组合在一起的函数。
- _finisher()_： 方法返回一个对结果容器执行最终转换的函数。

Here’s an example of creating a custom collector that calculates the average of a stream of integers:

下面是一个创建自定义收集器的示例，用于计算整数流的平均值：

```java
public class AverageCollector implements Collector<Integer, int[], Double> {  

    @Override  
    public Supplier<int[]> supplier() {  
        return () -> new int[2];  
    }  
      
    @Override  
    public BiConsumer<int[], Integer> accumulator() {  
        return (acc, val) -> {  
            acc[0] += val;  
            acc[1]++;  
        };  
    }  
      
    @Override  
    public BinaryOperator<int[]> combiner() {  
        return (acc1, acc2) -> {  
            acc1[0] += acc2[0];  
            acc1[1] += acc2[1];  
            return acc1;  
        };  
    }  
      
    @Override  
    public Function<int[], Double> finisher() {  
        return acc -> ((double) acc[0]) / acc[1];  
    }  
      
    @Override  
    public Set<Characteristics> characteristics() {  
        return Collections.emptySet();  
    }  
}
```

In this example, we define a new class called _AverageCollector_ that implements the Collector interface. The _supplier()_ method returns a function that creates a new integer array with two elements to store the sum and count of the elements in the stream. 

在本例中，我们定义了一个名为 _AverageCollector_ 的新类，该类实现了收集器接口。_supplier()_ 方法返回一个函数，用于创建一个包含两个元素的新整数数组，用于存储数据流中元素的总和与计数。

The accumulator() method returns a function that adds each element to the sum and increments the count. The _combiner()_ method returns a function that combines two integer arrays by adding their corresponding elements. The _finisher()_ method returns a function that calculates the average by dividing the sum by the count. Finally, the _characteristics()_ method returns an empty set because this collector does not have any special characteristics.

累加器()方法返回一个将每个元素加到总和中并递增计数的函数。_combiner()_ 方法返回一个函数，该函数通过将两个整数数组中的相应元素相加来合并两个数组。_finisher()_ 方法返回一个函数，通过用总和除以计数来计算平均值。最后，_characteristics()_ 方法返回空集，因为该收集器没有任何特殊特征。

Once you’ve defined your custom collector, you can use it with the collect() method of the Stream interface like this:

一旦定义了自定义收集器，就可以像这样将其与 Stream 接口的 collect() 方法结合使用：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);  
```

```java
double average = numbers.stream()  
                        .collect(new AverageCollector());
```

In this example, we create a stream of integers and use the collect() method with our custom AverageCollector to calculate the average of the integers in the stream. The lambda expressions we defined in the AverageCollector class are used to perform the accumulation and transformation operations.

在本例中，我们创建了一个整数流，并使用带有自定义 AverageCollector 的 collect() 方法计算流中整数的平均值。我们在 AverageCollector 类中定义的 lambda 表达式用于执行累加和转换操作。
## Use Lambda expressions to create Higher-order Functions 使用 Lambda 表达式创建高阶函数

A higher-order function is a function that takes one or more functions as arguments and/or returns a function as its result. Lambda expressions can be used to create functions that take other functions as arguments or return functions as results.

高阶函数是将一个或多个函数作为参数和/或将一个函数作为结果返回的函数。Lambda 表达式可用于创建将其他函数作为参数或将函数作为结果返回的函数。

> To create a higher-order function using lambda expressions, you can define a method that takes a functional interface as a parameter or return value. A functional interface is an interface that has exactly one abstract method and is annotated with the @FunctionalInterface annotation.
> 
> 要使用 lambda 表达式创建高阶函数，可以定义一个将功能接口作为参数或返回值的方法。功能接口是一个拥有一个抽象方法并使用 @FunctionalInterface 注解的接口。

Here’s an example of creating a higher-order function that takes a function as an argument:

下面是一个创建以函数为参数的高阶函数的示例：

```java
@FunctionalInterface  
interface BinaryOperator<T> {  
    T apply(T t1, T t2);  
}  

public static <T> BinaryOperator<T> compose(BinaryOperator<T> op1, BinaryOperator<T> op2) {  
    return (x, y) -> op1.apply(op2.apply(x, y), y);  
}
```

In this example, we define a functional interface called _BinaryOperator_ that has an _apply()_ method that takes two arguments and returns a result. We then define a static method called _compose()_ that takes two BinaryOperator functions as arguments and returns a new BinaryOperator function that composes the two input functions.

在本例中，我们定义了一个名为 _BinaryOperator_ 的函数接口，它有一个 _apply()_ 方法，该方法接收两个参数并返回一个结果。然后，我们定义了一个名为 _compose()_ 的静态方法，该方法接收两个 **BinaryOperator** 函数作为参数，并返回一个新的 **BinaryOperator** 函数，该函数将两个输入函数合成。

The _compose()_ method takes two BinaryOperator functions as arguments, op1 and op2. It then returns a new BinaryOperator function that applies op1 to the result of applying op2 to its arguments. The lambda expression (x, y) -> op1.apply(op2.apply(x, y), y) defines the new BinaryOperator function. The first argument of the lambda expression (x) is the result of applying op2 to the original arguments, and the second argument (y) is the second original argument.

_compose()_ 方法将 op1 和 op2 这两个二进制操作符函数作为参数。然后，它返回一个新的二进制操作符函数，该函数将 op1 应用于将 op2 应用于其参数的结果。lambda 表达式` (x, y) -> op1.apply(op2.apply(x, y), y)` 定义了新的二元运算符函数。lambda 表达式的第一个参数 (x) 是将 op2 应用于原始参数的结果，第二个参数 (y) 是第二个原始参数。

Here’s an example of using the compose() method to compose two BinaryOperator functions:

下面是一个使用 `compose()` 方法组合两个 `BinaryOperator` 函数的示例：

```java
BinaryOperator<Integer> add = (x, y) -> x + y;  
BinaryOperator<Integer> multiply = (x, y) -> x * y;  

BinaryOperator<Integer> composed = compose(add, multiply);  

System.out.println(composed.apply(2, 3)); // Output: 8
```

In this example, we define two BinaryOperator functions: add, which adds its two arguments, and multiply, which multiplies its two arguments. We then use the compose() method to create a new BinaryOperator function that first multiplies its arguments and then adds the result. We test the composed function by applying it to the values 2 and 3, which should result in 8.

在本例中，我们定义了两个 BinaryOperator 函数：add 和 multiply，前者用于将两个参数相加，后者用于将两个参数相乘。然后，我们使用 compose() 方法创建一个新的 BinaryOperator 函数，首先乘以参数，然后将结果相加。我们对组成的函数进行测试，将其应用于数值 2 和 3，结果应为 8。

Lambda expressions can also be used to create higher-order functions that return functions as results. Here’s an example of creating a higher-order function that returns a UnaryOperator function:

Lambda 表达式还可用于创建高阶函数，将函数作为结果返回。下面是创建返回 UnaryOperator 函数的高阶函数的示例：

```java
@FunctionalInterface  
interface UnaryOperator<T> {  
    T apply(T t);  
}  

public static <T> UnaryOperator<T> addValue(T value) {  
    return x -> x + value;  
}
```

In this example, we define a functional interface called UnaryOperator that has an apply() method that takes one argument and returns a result. We then define a static method called addValue() that takes a value of type T and returns a new UnaryOperator function that adds the value to its argument.

在本例中，我们定义了一个名为 UnaryOperator 的函数接口，它有一个 apply() 方法，该方法接收一个参数并返回一个结果。然后，我们定义了一个名为 addValue() 的静态方法，该方法接收一个 T 类型的值，并返回一个新的 UnaryOperator 函数，将值添加到其参数中。

The addValue() method returns a lambda expression that defines the new UnaryOperator function. The lambda expression x -> x + value takes one argument (x) and adds the value to it.

`addValue()` 方法返回一个 lambda 表达式，该表达式定义了新的 `UnaryOperator` 函数。lambda 表达式` x -> x + value` 接收一个参数`（x）`并将值添加到参数中。

Here’s an example of using the addValue() method to create a new UnaryOperator function:

下面是一个使用 `addValue()` 方法创建新的 `UnaryOperator` 函数的示例：

```java
UnaryOperator<Integer> add5 = addValue(5);  

System.out.println(add5.apply(2)); // Output: 7
```

In this example, we use the addValue() method to create a new UnaryOperator function that adds 5 to its argument. We assign the result to the add5 variable, which is now a function that adds 5 to its argument. We test the add5 function by applying it to the value 2, which should result in 7.

在本例中，我们使用 addValue() 方法创建了一个新的 UnaryOperator 函数，将 5 添加到参数中。我们将结果赋值给 add5 变量，它现在是一个将 5 添入其参数的函数。我们对 add5 函数进行测试，将其应用于数值 2，结果应为 7。

## Use Lambda expressions to create Closures 使用 Lambda 表达式创建闭包

A closure is a function that can access and modify variables in its enclosing scope. In other words, a closure “closes over” the variables in its enclosing scope and can use them as if they were local variables.

闭包是一个可以访问和修改其外层作用域中变量的函数。换句话说，闭包 "关闭 "其外层作用域中的变量，并能像使用局部变量一样使用它们。

> To create a closure using lambda expressions, you can define a lambda expression that references a variable in its enclosing scope. The lambda expression will then capture the value of the variable at the time the lambda expression is created and use that value whenever it is called.
> 
> 要使用 lambda 表达式创建闭包，可以定义一个 lambda 表达式，在其外层作用域中引用一个变量。然后，在创建 lambda 表达式时，lambda 表达式将捕获变量的值，并在调用时使用该值。

Here’s an example of creating a closure using a lambda expression:

下面是一个使用 lambda 表达式创建闭包的示例：

```java
public class ClosureExample {  

    public static void main(String[] args) {  
        int x = 5;  
        Runnable runnable = () -> System.out.println(x);  
        x = 10;  
        runnable.run(); // Output: 10  
    }  
}
```

In this example, we define a variable x with the value 5. We then define a lambda expression that references x and prints its value. Finally, we change the value of x to 10 and call the lambda expression.

在本例中，我们定义了一个值为 5 的变量 x。然后，我们定义一个 lambda 表达式，引用 x 并打印其值。最后，我们将 x 的值改为 10 并调用 lambda 表达式。

When the lambda expression is created, it captures the value of x, which is 5 at the time. When we call the lambda expression, it prints the value of x, which is now 10 because we changed its value after the lambda expression was created. The lambda expression “closes over” the variable x and uses it as if it were a local variable.

创建 lambda 表达式时，它会捕捉 x 的值，当时的值是 5。当我们调用 lambda 表达式时，它会打印出 x 的值，现在是 10，因为我们在创建 lambda 表达式后更改了它的值。lambda 表达式 "关闭 "了变量 x，并像使用局部变量一样使用它。

Here’s another example of creating a closure using a lambda expression:

下面是另一个使用lambda表达式创建闭包的示例：

```java
public class ClosureExample {  

    public static void main(String[] args) {  
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);  
        int sum = 0;  
        numbers.forEach(n -> sum += n);  
        System.out.println(sum); // Output: 15  
    }  
}
```

In this example, we define a list of integers and a variable sum with the initial value 0. We then use a lambda expression to iterate over the list and add each element to the sum variable. Finally, we print the value of sum.

在本例中，我们定义了一个整数列表和一个初始值为 0 的变量 sum。然后，我们使用 lambda 表达式遍历列表，并将每个元素添加到 sum 变量中。最后，我们打印 sum 的值。

The lambda expression “closes over” the variable sum and updates its value each time it is called. This allows us to calculate the sum of the list using a single lambda expression and a single variable, without the need for a separate loop or accumulator.

lambda 表达式 "关闭 "变量 sum，并在每次调用时更新其值。这样，我们就可以使用单个 lambda 表达式和单个变量计算列表的总和，而无需单独的循环或累加器。
## Use Lambda expressions with Method References 在方法引用中使用 Lambda 表达式

> Method references allow you to refer to an existing method by name instead of defining a new lambda expression. This can make code more concise and readable, especially when working with simple functions.
>
> 方法引用允许你通过通过引用一个现有方法，而不是重新定一个新的函数表达式。这会使得代码更加简洁并且可读，特别是在处理简单函数时。

There are four types of method references in Java:

1. Reference to a static method
2. Reference to an instance method of a particular object
3. Reference to an instance method of an arbitrary object of a particular type
4. Reference to a constructor

Java中有四种方法引用类型：

1. 静态方法引用
2. 对特定对象的实例方法的引用
3. 对特定类型的任意对象的实例方法的引用
4. 对构造函数的引用

> To use a method reference, you can replace the lambda expression with a reference to the method by name. The method reference syntax is similar to the lambda expression syntax, but with the method name and class name instead of the parameter list and arrow.
> 
> 使用方法引用，你可以使用引用的方法名称替换Lambda表达式，**方法引用的语法与 lambda 表达式的语法类似**，只是用方法名和类名代替了参数列表和箭头。

Here’s an example of using a method reference to a static method:

下面是使用静态方法引用的例子：

```java

public class MethodReferenceExample {  
    public static void main(String[] args) {  
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);  
        numbers.forEach(System.out::println);  
    }  
}
```
In this example, we define a list of integers and use the forEach() method to print each element to the console. Instead of using a lambda expression to define the action, we use a method reference to the static method System.out.println(). The double colon (::) operator is used to indicate that we want to reference a method instead of defining a new lambda expression.

这个例子中，我们定义一个整型列表，使用`forEach()`方法打印到控制台，这里不需要使用Lambda表达式，而是直接使用静态方法`System.out.println()`的方法引用，**双冒号（::）操作符用于表示我们要引用一个方法**，而不是定义一个新的 lambda 表达式

Here’s an example of using a method reference to an instance method of a particular object:

下面是一个使用方法引用到特定对象的实例方法的示例：

```java
public class MethodReferenceExample {  
    public static void main(String[] args) {  
        List<String> strings = Arrays.asList("apple", "banana", "cherry");  
        strings.sort(String::compareToIgnoreCase);  
        System.out.println(strings); // Output: [apple, banana, cherry]  
    }  
}
```


In this example, we define a list of strings and use the sort() method to sort the list in a case-insensitive manner. Instead of using a lambda expression to define the comparator function, we use a method reference to the compareToIgnoreCase() method of the String class.

在这个案例中，我们定义字符串列表并且用 `sort()` 方法以不区分大小写的方式对列表进行排序，我们没有使用 lambda 表达式来定义比较器函数，而是使用了对 String 类的 compareToIgnoreCase() 方法的引用。

Here’s an example of using a method reference to an instance method of an arbitrary object of a particular type:

下面是一个示例，说明如何使用方法引用来引用特定类型的任意对象的实例方法：

```java
public class MethodReferenceExample {  
    public static void main(String[] args) {  
        List<String> strings = Arrays.asList("apple", "banana", "cherry");  
        strings.forEach(String::toUpperCase);  
        System.out.println(strings); // Output: [apple, banana, cherry]  
    }  
}
```


Here’s an example of using a method reference to a constructor:

下面是一个将方法引用用于构造函数的示例：


```java
public class MethodReferenceExample {  
    public static void main(String[] args) {  
        List<String> strings = Arrays.asList("apple", "banana", "cherry");  
        List<String> copy = strings.stream()  
                                   .map(String::new)  
                                   .collect(Collectors.toList());  
        System.out.println(copy); // Output: [apple, banana, cherry]  
    }  
}
```


In this example, we define a list of strings and use the stream() method to create a stream of the strings. We then use the map() method to create a new stream of strings, each of which is created using the String constructor that takes a single argument. This constructor is referenced using a method reference to the String class constructor. Finally, we collect the new stream into a list and print the result.

在这个例子中，我们定义包含字符串的列表并且使用stream 方法创建一个String 流，接着我们使用map方法创建一个新的String流，每个字符串都是使用String构造函数新创建的，该构造函数只需要一个参数。构造函数使用String类构造函数的方法引用。最后，我们将新的数据流收集到一个列表中并打印结果。

## Use Lambda expressions with default methods in Interfaces 在接口中使用带有默认方法的 Lambda 表达式

Default methods were introduced in Java 8 to allow interfaces to define a default implementation for a method. This allows interfaces to evolve over time without breaking existing implementations.

Default 默认方法是 Java 8 的新特性之一，主要是允许接口定义默认方法并且实现自己的逻辑，这使得接口可以在不破坏现有实现的情况下随着时间的推移而发展。

> Lambda expressions can be used with default methods in interfaces by providing an implementation for the default method in the lambda expression. This allows the lambda expression to be used as an implementation of the interface, even if the interface defines a default method.
>
> **通过在 lambda 表达式中为缺省方法提供实现，lambda 表达式可以与接口中的缺省方法一起使用。** 即使接口定义了缺省方法，也可以将 lambda 表达式用作接口的实现。

Here’s an example of using a lambda expression with a default method in an interface:

下面是一个使用Lambda表达式和使用静态方法接口的例子。
      
```java
@FunctionalInterface  
interface Calculator {  
    int add(int x, int y);  
    default int subtract(int x, int y) {  
        return add(x, -y);  

    }  
}  
```


```java
public class DefaultMethodExample {  
public static void main(String[] args) {  
    Calculator calculator = (x, y) -> x + y;  
    System.out.println(calculator.add(2, 3)); // Output: 5  
    System.out.println(calculator.subtract(2, 3)); // Output: -1  
}  
    }
```


In this example, we define a functional interface called Calculator that has an add() method and a default subtract() method. The subtract() method calls the add() method to perform the subtraction.

上面例子中我们定义`Calculator `接口和静态方法`subtract`和接口方法`add()`,`subtract`方法内部会调用`add()`方法实现减法操作。

We then define a lambda expression that implements the add() method of the Calculator interface. We can use this lambda expression as an implementation of the interface, even though the interface defines a default method.

我们接着使用Lambda表达式定义一个实现了Calculator 接口的`add()`方法的实现类，我们可以使用这个lambda表达式作为接口的实现，尽管接口定义了一个**缺省方法**。

## Use Lambda expressions with the Optional Class 与可选类一起使用 Lambda 表达式

The Optional class was introduced in Java 8 to help prevent null pointer exceptions by providing a container object that may or may not contain a value. Optional provides a set of methods for working with potentially null values in a safer and more concise way.

**Optional** 是Java8 当中引入的类，它提供一个可能包含或者不包含值的容器预防控制针异常，Optional提供了一组方法，用于以更安全、更简洁的方式处理潜在的空值。

> Lambda expressions can be used with the Optional class to define custom behavior for cases where a value is present or absent. Optional provides several methods for working with values, including map(), flatMap(), filter(), ifPresent(), and orElse(). These methods can be used with lambda expressions to define custom behavior for these cases.
> 
> Lambda 是可以和 Optional 类一起使用，在值存在或不存在的情况下定义自定义行为，比如map(), flatMap(), filter(), ifPresent(), and orElse() 等方法，可以和 Lambda一起使用

Here’s an example of using a lambda expression with the map() method of the Optional class:

下面是使用Optional.map() 方法的简单案例。

```java
Optional<String> optional = Optional.of("hello");  
Optional<Integer> result = optional.map(s -> s.length());  
System.out.println(result.get()); // Output: 5
```

In this example, we create an Optional object that contains the string “hello”. We then use the map() method to apply a lambda expression that returns the length of the string. The result of the map() method is an Optional object that contains the result of the lambda expression. We can then use the get() method to retrieve the value from the Optional object and print it to the console.

例子中创建一个内容为“hello”的Optional对象，接着使用 map() 函数应用一个返回字符串长度的lambda表达式。map() 结果是包含Lambda表达式结果的可选对象，我们使用get() 方法从 Optional 对象中获取值。

Here’s an example of using a lambda expression with the filter() method of the Optional class:

下面是使用`Optional.filter()`的另一个案例

```java
Optional<String> optional = Optional.of("hello");  
Optional<String> result = optional.filter(s -> s.length() > 5);  
System.out.println(result.isPresent()); // Output: false
```

In this example, we create an Optional object that contains the string “hello”. We then use the filter() method to apply a lambda expression that filters out strings with length less than or equal to 5. Since “hello” has a length of 5, it does not pass the filter and the resulting Optional object is empty. We can then use the isPresent() method to check whether the Optional object contains a value and print the result to the console.

例子中创建一个内容为“hello”的Optional对象，接着使用 filter() 函数应用一个**过滤掉**字符串长度小于等于5的值lambda表达式，这时候得到的Optional 对象是为空的，使用 `isPresent` 方法便可以检查是否包含一个值，最终打印结果到控制台。

Here’s an example of using a lambda expression with the orElse() method of the Optional class:

最后是一个`orElse()`的栗子。

```java
Optional<String> optional = Optional.empty();  
String result = optional.orElseGet(() -> "default");  
System.out.println(result); // Output: default
```

In this example, we create an empty Optional object and use the orElseGet() method to provide a default value if the Optional object is empty. We pass a lambda expression that returns the default value “default”. Since the Optional object is empty, the orElseGet() method returns the default value, which we then print to the console.

这个例子中我们创建一个空的 **Optional** 对象并且使用`orElseGet()`方法在Optional 内部内容为空的的时候提供一个默认值。我们传递一个 lambda 表达式，返回默认值 "default"。由于 Optional 对象为空，orElseGet() 方法返回默认值，然后我们将其打印到控制台。

_Thanks for your Attention! Happy Learning!_

感谢您的关注！学习愉快！_