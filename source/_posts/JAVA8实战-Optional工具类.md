---
title: JAVA8实战 - Optional工具类
subtitle: java一个比较受争议的改进
author: 阿东
url_suffix: optional
tags:
  - java
categories:
  - java
keywords: optional,java
description: JAVA8可以说是JAVA划时代的一个版本
copyright: true
date: 2022-02-20 20:26:21
---

# JAVA8实战 - Optional工具类

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210807203312.png)

[toc]



# 前言

​	没错，这又是一个新的专栏，JAVA8可以说是JAVA划时代的一个版本，几乎是让JAVA焕发了第三春（第二春在JDK5），当然里面的新特性也是十分重要的，虽然Java现在都已经到了10几的版本，但是国内多数使用的版本还是JAVA8，所以这个系列将会围绕Java8的新特性和相关工具做一些总结。希望对大家日常学习和工作中有所帮助。



# 概述：

1. 日常工作学习我们大致是如何规避空指针的。
2. 关于Optional的系统介绍，常见的使用和处理方法
3. Optional的使用场景以及一些小型案例代码
4. 来看看《Effective Java》这个作者如何看待Optional这个工具类>

# 思维导图：
地址：https://www.mubucm.com/doc/2qS40EL1g_B
![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210807214651.png)

<!-- more -->

# 空指针规避

​	在讲述`Optional`之前，我们来看下通常情况下我们是如何防止空指针的。

## 字符串`equals`

​	字符串的操作是最常见的操作，使用字符串的equals方法很有可能抛出空指针异常，比如像下面的代码，如果a变量为Null，则毫无疑问会抛出空指针异常：

```java
a.equals("aaa");
```

​	建议：使用`Objects.equals()`或者使用其他工具类方法替代，或者确保`obj.equals(target)`的obj对象不会为null，比如`"test".equals(target)`。

​	比如我们使用下面的方法保证**equals**的时候一致：

```java
public Tank createTank(String check){
    Tank tank = null;
    if(Objects.equals(check, "my")){
        tank = new MyTank();
    }else if(Objects.equals(check, "mouse")){
        tank = new MouseTank();
    }else if (Objects.equals(check, "big")){
        tank = new BigTank();
    }else {
        throw new UnsupportedOperationException("unsupport");
    }
    return tank;
}
```

​	

## 变量 `==` 操作

​	变量的`==`操作也是用的十分多，通常情况下和null搭配的比较多，我们通常需要注意下面这些事项：

1. 确保比较类型一致，比如最经典的**Integer**和**int**比较在超过127的时候为false的问题。
2. 使用框架工具类的equals() 进行替代
3. 使用`Objects.equals()`方法替代

​	特别强调一下**Integer**的 `==`操作的一些陷阱，特别注意最后一个打印是False，具体的原因有网上很多资料，这里就不啰嗦了：

```java
 public static void main(String[] args) {
        Integer a = 1;
        Integer b = 256;
        System.out.println(a == null);
        System.out.println(a == b);
        System.out.println(a == 1);
        System.out.println(b == 256);
        System.out.println(b == 257);

 }/*运行结果:
    false
    false
    true
    true
    false

    */
```



## 集合元素为null

​	如果在一个List或者Set中存在Null元素，那么遍历的时候也很容易出现空指针的问题，通常情况下我们可以使用`Stream.filter`进行过滤，比如像下面这样，这里使用了**StringUtils::isNotBlank**来判断是否为空字符串并过滤掉所有的空字符串和Null元素:

```java
@Test
public void test2(){
    List<String> list = Arrays.asList("1", null, "2", "", "3");
    System.out.println(list.size());
    List<String> collect = list.stream().filter(StringUtils::isNotBlank).collect(Collectors.toList());
    System.out.println(collect.size());

}/*运行结果
    5
    3

    */
```

最终的建议如下：

+ 元素null多数情况不常见，但是null的集合对象比较常见
+ 可以编写工具类方法对于集合的内容进行null排除，或者使用lambada表达式处理



## map的元素值为null

​	map也是容易出现null的，比如下面这种情况，一旦get()的返回结果为null，就会出现空指针的异常情况：

```java
map.get("user").getName()
```

建议：

1. 使用MapUtils获取元素
2. 每次获取之前需要判断是否为空

​	第一条建议使用`MapUtils`，代码都比较简单，唯一需要注意的是使用的时候小心**自动装箱**的性能和效率问题：

```java
@Test
public void test1(){
    Map<String, Object> keyVal = new HashMap<>();
    keyVal.put("name","value");
    keyVal.put("yes", new Object());
    keyVal.put("intval1", 1);
    Object val1 = MapUtils.getObject(null, "yes");
    Object val2 = MapUtils.getObject(keyVal, "yes");
    String str1 = MapUtils.getString(keyVal, "name");
    int int1 = MapUtils.getInteger(keyVal, "intval1");
    System.out.println(val1);
    System.out.println(val2);
    System.out.println(str1);
    System.out.println(int1);
}
```



## 类型强转为null

​	还有一种比较常见的情况就是json转换为null，如果传入**空字符串**，会导致转化的结果是一个Null值，所以在转化的地方要么对于字符串做判断是否为空的操作，或者对于转换后的对象进行判空，比如下面的代码就需要对于JSON进行处理：

```java
@Test
public void test3(){
    String str = "{\"name\":\"123\"}";
    String str2 = "{\"email\":\"123\"}";
    String str3 = "";
    User map = JSON.parseObject(str, User.class);
    User email = JSON.parseObject(str2, User.class);
    User user2 = JSONObject.parseObject(str3, User.class);
    System.out.println(Objects.isNull(map));
    System.out.println(Objects.isNull(email));
    System.out.println(Objects.isNull(user2));
}/*
true
true
false
*/
```

​	看了这么多案例，可以发现日常生活中规避空指针是一件非常烦的事情，特别是存在多层嵌套的对象，基本会出现多层的If/else判断，这样会造成代码复杂度增加并且让代码变得十分臃肿，接下来我们就来看下JAVA8是如何使用`Optional`工具来简化这些操作的。



# 什么是Optional？

## 简单介绍

​	Java8之后新增的一个工具类，在包`java.util.Optional<T>`，他的作用类似于一个包装器，负责把我们需要操作的对象包装到一个黑盒中，我们可以通过黑盒安全的操作对象的内容。

## 案例对象：

​	这里简单构建了两个案例对象进行处理：

```java
static class User{
    private String name;
    private int age;

    private Car car;

    public Car getCar() {
        return car;
    }

    // Tip: 兼容序列化
    public Optional<String> getPersonCarName(){
        return Optional.ofNullable(car.getCarName());
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

class Car{
    private String carName;
    private String color;

    public String getCarName() {
        return carName;
    }

    public void setCarName(String carName) {
        this.carName = carName;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }
}
```



## Option的三种初始化方式：

```java
 /**
     * optional 工具类的初始化方法
     * 1. 介绍三种构造方式
     * 2. 主要区别是初始化传入参数是否允许为null
     * Optional.of 不允许为空
     * Optional.ofNullable 允许为空
     * Optional.empty 构建空Optional对象
     */
@Test
public void testInit() {
    // 这种构造方式不能为null，否则会空指针异常
    Optional<Object> notNull = Optional.of(new Object());
    // 允许为空
    Optional<Object> nullAble = Optional.ofNullable(null);
    // 这种方式是返回一个空Optional，等效Optional.ofNullable(null)
    Optional<Object> empty = Optional.empty();
}
```

1. **Optional.of()**：表示创建一个不允许是空值的Optional，如果传入为Null会抛出异常
2. **Optional.ofNullable()**：表示传入的内容允许是空，但是实际上和`Optional.empty()`效果一致。
3. **Optional.empty()**：创建一个空的Optional。



## map - 对象的内容提取和转化：

​	在进入具体介绍之前先看看汇总的测试代码以及相关说明，也方便节省各位的时间：

```java
 /**
     * optional 如何准确的获取对应的值
     * 1. Optional.map 使用map收集某一个对象的值,
     * 2. Optional.flatMap 根据 Optional 的结果获取参数
     * 插曲：map 和 flatMap 的区别
     * 3. Optional.filter 筛选出符合结果的参数
     */
    @Test
    public void testGetOptionalVal(){
        User user = new User();
        Optional<User> notNull = Optional.ofNullable(user);
        Integer age = notNull.map(User::getAge).orElse(22);
        String name = notNull.map(User::getName).orElse("myname");
        System.out.println(age);
        // Optional.map 收集某一个对象的值
        System.out.println("Optional.map 收集某一个对象的值："+ age);
        System.out.println("Optional.map 收集某一个对象的值："+ name);

        // Optional.flagMap 获取多层Optional迭代对象：
        Optional<String> s = notNull.flatMap(User::getPersonCarName);
        Boolean aBoolean = s.map(String::trim).map(StringUtils::isNotBlank).get();
        System.out.println(aBoolean);
        
        // Optional.map 收集某一个对象的值
        User u1 =new User();
        u1.setName("小王");
        u1.setAge(11);
        User u2 =new User();
        List<User> userLists = new ArrayList<>();
        userLists.add(u1);
        userLists.add(u2);
        Optional<List<User>> notNull2 = Optional.of(userLists);
        // 针对对象集合，使用flagMap获取关联数据
        Optional<User> user1 = notNull2.flatMap(item -> Optional.ofNullable(item.get(0)));
        user1.ifPresent(u-> System.out.println("针对对象集合，使用flagMap获取关联数据 => "+u.getName()));

        // flatMap 的使用场景
        List<String> stringList = new ArrayList<>();
        stringList.add("name1");
        stringList.add("testone");
        stringList.add("other");
        // 使用flatMap处理Optional的返回结果
        Optional<String> stringOptional = Optional.of(u1).flatMap(u -> Optional.ofNullable(u1.getName()));
        System.out.println("flatMap" + stringOptional);

        // 对比：map和flatMap的区别
        // map 方法签名 Function<? super T, ? extends U> mapper
        Optional<String> map = notNull.map(User::getName);
        // flatmap 的方法签名： Function<? super T, Optional<U>> mapper
        Optional<String> stringOptional1 = notNull.flatMap(u -> Optional.ofNullable(u.getName()));
        // 虽然从结果来看两者的结果没有什么区别
        // map => Optional.empty
        //  flatMap => Optional.empty
        // 但是可以明显的看到
        // flatMap Function的返回值为 Optional 类型 Optional<String>
        // map 的 Function返回值为 具体返回类型 String + Optional 的自动封装 Option<String>
        System.out.println("map => " + map);
        System.out.println("flatMap => " + stringOptional1);

        // filter 方法使用
        Optional<String> optional = Optional.of("testNAME");
        String result = optional.filter(str -> str.contains("test")).orElse("not found");
        System.out.println(result);
    }
```

​	上面提到map获取值的方式`map.get("user").getName()`这种方式很容易导致空指针异常。对于Optional，我们可以使用`map()`操作来进行规避，这里可以把Optional想象为一个特殊的集合数据，如果包含一个值，map就会帮我们把流里面的数据进行转化，如果没有值就什么都不做，当然如果我们没有值，可以像案例一样使用`orElse()`在没有值的时候返回这个值作为默认参数，最终的代码效果如下：

```java
Integer age = notNull.map(User::getAge).orElse(22);
String name = notNull.map(User::getName).orElse("myname");
```

​	但是，我们也可以看到map也是存在局限性的，对于单个对象操作十分方便，但是一旦遇到多层**Optional**嵌套。比如Optional处理Optional的处理结果，就需要使用**Optional.flatMap()**的操作。

> 对比：map和flatMap的区别？
> 	从代码里可以看到，map主要针对的是单一对象结果进行处理，比如我们将对象传给方法引用`String::trim`进行`trim()`的操作，这里不需要去思考底层的操作逻辑，只需要知道当遇到Optional需要处理多层的Optional嵌套的时候，就需要使用`Optional.flagMap`。
>
> ​	如果还是很难以理解的话，也可以想象为一个套娃一样套着多层黑盒的操作，我们可以使用flatMap安全的取出属于最底层对象的属性，如果还是不好理解，可以想象为安全的做下面代码的操作：
>
> `map.get("user").get("car").getCarName()`



## 插曲：Optional的序列化问题

​	书中讨论了Optional的序列化问题，书中特别强调：**如果你应用的某些字段需要序列化，使用Optional操作有可能发生失效**，这里给了一个建议如果一定需要序列化的方式处理的话，可以按照下面的方法处理：

```java
public Optional<String> getPersonCarName(){
    return Optional.ofNullable(car.getCarName());
}
```



## 解引用Optional对象方法

​	下面整合了关于Optional大部分常见操作。

```java
/**
     * optional 校验对象属性等是否存在
     * 1. Optional.isPresent 校验对象是否存在，存在返回true
     * 2. Optional.orElse 如果为空返回默认值，不为空不做处理
     * 3. Optional.get 对象必须存在
     * 4. Optional.orElseGet 通过方法提供值
     * 5. Optional.orElseThrow 如果获取为null，抛出指定异常
     * 6. Optional.isPresent 使用ifPresent()来进行对象操作，存在则操作，否则不操作
     * 7. Optional.filter 操作，可以过滤出符合条件的内容
*/
@Test
public void testOptionalValExists() {
    // 对象属性是否存在
    Optional<Object> notNull = Optional.of(new Integer(4));
    boolean present = notNull.isPresent();
    System.out.println("notNull 值是否不为空 " + present);
    Optional<Object> nullAble = Optional.ofNullable("sss");
    System.out.println("nullAble 是否不为空 "+ nullAble.isPresent());

    // Optional.orElse - 如果值存在，返回它，否则返回默认值
    Optional<Object> integerNull = Optional.ofNullable(null);
    Object o = integerNull.orElse("22");
    System.out.println("o 否则返回默认值 " + o);

    //Optional.get - 获取值，值需要存在
    Optional<Object> integerNull2 = Optional.ofNullable(null);
    // 抛出异常 java.util.NoSuchElementException: No value present
    // 来源：throw new NoSuchElementException("No value present");
    // Object o1 = integerNull2.get();
    Optional<Object> integerNull3 = Optional.ofNullable(12321);
    System.out.println("Optional.get 必须存在"+ integerNull3.get());

    //通过方法提供值
    Optional<Object> integerNull4 = Optional.ofNullable(12321);
    Object o1 = integerNull4.orElseGet(() -> String.valueOf(22));
    System.out.println("Optional.orElseGet 通过方法提供值" + o1);

    // 如果获取为null，抛出指定异常
    Optional<Object> integerNull5 = Optional.ofNullable(null);
    // java.lang.RuntimeException: 当前运行代码有误 如果需要抛出异常，请放开下面的代码
    //        Object orElseThrow = integerNull5.orElseThrow(() -> new RuntimeException("当前运行代码有误"));
    //        System.out.println("Optional.orElseThrow 自定义异常" + orElseThrow);

    // Optional.isPresent 使用ifPresent()来进行对象操作，存在则操作，否则不操作
    integerNull5.ifPresent((item) -> {
        System.err.println("Optional.isPresent 如果存在对象，执行如下操作");
    });


    // filter 方法使用
    Optional<String> optional = Optional.of("testNAME");
    String result = optional.filter(str -> str.contains("test")).orElse("not found");
    System.out.println("Optional.filter 过滤出符合条件的对象: " + result);

}/*运行结果：
        notNull 值是否不为空 true
        nullAble 是否不为空 true
        o 否则返回默认值 22
        Optional.get 必须存在12321
        Optional.orElseGet 通过方法提供值12321
        Optional.orElseThrow 自定义异常 当前运行代码有误
        java.lang.RuntimeException: 当前运行代码有误
        Optional.isPresent 如果存在对象，执行如下操作
        Optional.filter 过滤出符合条件的对象: testNAME
    */
```

# Optional的使用场景

​	<s>从个人角度来说其实使用不是很方便</s>，下面说下Optional的工具使用场景：

## 1. 封装可能为Null的值

​	还是和所说的Map类似，当我们使用下面的操作获取值的时候，就很容易引发null：

```java
User user = (User)map.get("user");
```

​	于是，Optional的用处就派上了：

```java
Optional<Object> value = Optional.ofNullable(map.get("user"));
```

## 2. 异常和optional的对比

​	通常情况下我们会使用捕获异常的方式进行异常的处理，下面是一个常见的字符串转Int的方法，一般情况下我们都会用`try/catch`防止空指针或者转化异常，除非我们可以保证数据的准确性：

```java
String str = "s";
try{
    Integer.parseInt(str);
}catch(Exception e){
    // ....
}
```

​	使用**Optional**之后可以进行如下的改造，我们将得到一个安全的Optional进行操作，而不是一个可能存在隐式的**NullPointException**问题代码：

```java
public static Optional<Integer> str2Int(String str) {
    try {
        return Optional.of(Integer.parseInt(str));
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}
```

## 3. 建议将Optional封装到一个工具类当中：

​	比如封装成下面这种的简单方法：

```java
private static Optional<Integer> str2Int(String str) {
    try {
        return Optional.of(Integer.parseInt(str));
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}
```

​	

## 4. 实战：读取Properties值

​	这里直接把书里的案例拿来用了，这个方法主要作用是读取一个配置文件的`int`值，当读取不到内容的时候，自动给默认值0。

```java
private static Optional<Integer> str2Int(String str) {
    try {
        return Optional.of(Integer.parseInt(str));
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}

public static int read(Properties properties, String name) {
    return Optional.ofNullable(properties.getProperty(name))
        .flatMap(OptionalTest::str2Int)
        .filter(i -> i > 0)
        .orElse(0);
}
```

## 5. 其他测试：

​	下面是个人学习的时候一些简单案例尝试：

```java
/**
     * 实际使用场景
     * 1. 我们要将一个对象的名称全部统一为大写，防止空指针. 但是实际使用来看还是遇到了不少的问题
     */
    @Test
    public void actualUse() {
        User user = new User();
        // java.lang.NullPointerException 如果编程习惯不好，这种工具类其实并不能解决问题
//        Optional.ofNullable(user).ifPresent(u->{
//            String toLowerCase = u.getName().toLowerCase();
//            u.setName(toLowerCase);
//        });
        // 正确的使用方式应该是如下的形式：
        // 下面的语句放开注释运行打印结果为： SSS
//        user.setName("sss");
        // 如果为null会抛出 java.lang.RuntimeException
//        String s = Optional.ofNullable(user).map(User::getName).map(String::toUpperCase).orElseThrow(RuntimeException::new);
//        System.out.println(s);
        // 我们也可以用另一种方式
        String s2 = Optional.ofNullable(user).map(User::getName).map(String::toUpperCase).orElse("默认值");
        System.out.println(s2);

        // 我们要对一个前端传入的值进行split或者substring的时候
        // 案例数据除开分隔符有差异之外无任何差异
        String tags1 = "标签1,标签2,标签3,标签4";
        String tags2 = "标签1，标签2，标签3，标签4";
        String[] strings1 = Optional.of(tags1).map(tg -> tags1.split(",")).get();
        System.out.println(strings1[0]);
        String[] strings2 = Optional.of(tags2).map(tg -> tags2.split(",")).get();
        String[] strings3 = Optional.ofNullable(tags2).map(StringUtils::isNotBlank).map(tg -> tags2.split(",")).orElse(new String[]{"ss"});
        System.out.println(strings2[0]);
        System.out.println(Arrays.toString(strings3));
        // 运行结果，如果此时有值，基本无问题
        // 标签1
        // 标签1，标签2，标签3，标签4

        // 如果上面的案例当中，传入的为null会如何?
        // 所以我们需要修改上面的格式，确保不管tag的值是否存在，都可以只关心我们具体需要操作的数据
        // 如果为null，则没有任何结果处理。我们可以使用map进行各种操作
//        String tags3 = "null,222";
        String tags3 = "a,b|C|d";
        Optional.ofNullable(tags3).map(tg -> tags3.split(",")).map(tg -> {
            for (int i = 0; i < tg.length; i++) {
                tg[i] = tg[i].toUpperCase();
            }
            return tg;
        }).ifPresent(item -> {
            for (String s : item) {
                System.out.println(s);
            }
        });

        // JSON解析的数据失败或者传入的格式不对导致的NULL
        System.out.print("\nJSON解析的数据失败或者传入的格式不对导致的NULL");
        Object parse = JSON.parseObject("{name:\"13\"}", User.class);
        System.out.println(parse);
        Optional.ofNullable("{age:\"1\"}")
                .map(obj -> JSON.parseObject(obj, User.class))
                .ifPresent(System.out::println);
    }/*运行结果
    默认值
    标签1
    标签1，标签2，标签3，标签4
    A
    B|C|D

    JSON解析的数据失败或者传入的格式不对导致的NULLcom.xxx.interview.jdk8.OptionalTest$User@548b7f67
    com.xxx.interview.jdk8.OptionalTest$User@1810399e
    */
```

# 《Effective java》第55条建议

> ​	感兴趣的可以直接跳转下面这些链接（在线阅读网站貌似点不进去，所以只有在线PDF网址了...）
>
> [itmyhome.com](http://itmyhome.com/effective-java/effective-java.pdf)
>
> ​	以防万一这里再补一个百度链接（如果公众号无法点击，请阅读原文获取）
>
> 链接：https://pan.baidu.com/s/1kQ8EHTn7N6kErH0EUBIDUg 
> 提取码：vkv3 

​	既然提到了Optional的用法，这里也一并谈谈关于《Effective java》是如何看待这一个工具类的，这一条的标题是：**谨慎返回Optional**。

​	从个人的角度来看，Optional的本质作用是：**提供了结果的可扩展性以及提供给调用方更多的可操作性**，比如调用方可以使用此来按照以前的判断null方式处理`if(obj.isPresent())`，或者使用新式的Lambda操作，比如像下面这样，如果存在值则进行打印的操作，否则什么事情都不会发生：

```java
Optional.ofNullable("{age:\"1\"}")
                .map(obj -> JSON.parseObject(obj, User.class))
                .ifPresent(System.out::println);
```

​	还有一点需要注意的是`get()`这个方法，如果不能确保值确实存在，建议谨慎或者避开这个方法，因为一旦为null此方法会抛出一个空指针异常。

​	另外，在之前讲述的方法：`orElseThrow`这个方法传入的是一个**异常工厂**而不是真正的异常。

​	后面主要提到的是一些Java9的操作，由于本文只涉及Java8的版本，所以更高版本的内容可以从《Effective Java》这本书里面看到。

​	下面是作者关于optional的一些个人建议以及编程禁忌：

## 几点警告

### 1. 永远不要使用Optional返回Null

​	首先，该书作者也是提到了Optional在日常的编码工作当中如何使用它来规避一些可能存在的null对象操作，同时提出一个重要的禁忌：**永远不要用Optional的返回值返回null**，比如我们将上面练习当中的代码改为下面的方式：

```java
private static Optional<Integer> str2Int(String str) {
    try {
        return Optional.of(Integer.parseInt(str));
    } catch (NumberFormatException e) {
        // 千万不要这么做
        return null;
    }
}
```

### 2. 不要使用包装基本类型的Optional

​	设计Optional的设计师在考虑的时候，为基础类型也设置了专属的Optional类，然而作者认为这三个类的设计**很垃圾**，并且建议**永远不要返回基本包装类型**，这里验证了一下，发现少了确实少了不少方法，比如：`ofNullable`这个方法，这会直接导致你在编写代码的时候增加不必要的判断，并且无法互相兼容，我想这也是作者不推荐的原因之一吧。

+ OptionalDouble
+ OptionalInt
+ OptionalLong

### 3. 不要把optional作为映射或者键值

​	这也很好理解，比如像下面这样：

```java
Map<Optional<String>, Optional<Object>> map = // .....
```

​	这并不会让你少些代码，反而会增加代码的理解难度和程序的复杂度，所以不建议把Optional用作任何的键值对或者集合的元素当中。



## 什么时候应该使用？

 	一句话：**如果无法返回结果并且返回结果的客户端必须处理的时候，就应该声明Optional\<T\>。**



## 哪些情况不适用？

​	其实上面的警告也提到了一部分内容：

1. **非常注重性能的场合**：因为Optional的封装以及类似“流”的操作需要额外的内存开销，所以不适合一些十分注重性能的情况。
2. **需要使用键值对或者集合元素的场合**：原因在上文说了，这里不再赘述。
3. **包装基本类型的Optional**：设计缺陷，对比源代码就知道了。



# 总结：

​	总之**Optional**这个工具还是具备一定的实用价值的，这里非常喜欢《Effective Java》作者对于这个工具类的一些建议，可以说是一针见血，果然大神的眼光和经验是非常独到的。



# 写到最后

​	写稿不易，求赞，求收藏。

​	最后推荐一下个人的微信公众号：**“懒时小窝**”。有什么问题可以通过公众号私信和我交流，当然评论的问题看到的也会第一时间解答。

