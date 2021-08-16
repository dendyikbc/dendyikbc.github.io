---
layout: post
title: 'Java中Optional类使用笔记'
subtitle: 'javaOptional'
date: 2021-08-11
author: Dave
cover: ''
tags: Java
---

## Optional Java

>最近回顾一下，写个笔记。

**最前面**

>Optional 类主要解决的问题是空指针异常NPE（NullPointerException），借鉴自google guava类库的Optional类，用以避免逻辑上的显式null值判断（null的防御性检查）。

```java
//使用Optional前，操作数据，往往显式判断null
public static String getGender(Student student)
{
    if(null == student)
    {
        return "Unkown";
    }
    return student.getGender();
    
}

//使用Optional优化后
public static String getGender(Student student)
{
    return Optional.ofNullable(student).map(u -> u.getGender()).orElse("Unkown");
    
}
```

### Optional对象创建

>创建方法根据是否允许对象值为null进行判断

```java
// 1、创建一个包装对象值为空的Optional对象
Optional<String> optStr = Optional.empty();
// 2、创建包装对象值非空的Optional对象
Optional<String> optStr1 = Optional.of("optional");
// 3、创建包装对象值允许为空的Optional对象
Optional<String> optStr2 = Optional.ofNullable(null);
```

### 基础方法与示例

>一些基础方法的源码见页面底下附录

#### 1.Optional对象的访问

```java
String name = "piccolo";
String name = "piccolo";
Optional<String> opt = Optional.ofNullable(name);
// 1.isPresent() + get()
if(opt.isPresent()){
    System.out.println(opt.get());
}
// 2.ifPresent() + get()
opt.ifPresent(System.out::println);
```

#### 2.附带默认值的返回

- 返回对象
  
>orElse()与orElseGet()当对象为空而返回默认对象时，行为并无差异；`对象非空时，二者均返回非空值，但orElse()仍创建对象，消耗资源，而orElseGet()不创建对象。`

```java
//创建对象操作
public static String createString() {
    System.out.println("Creating New String");
    return new String("Test");
}
```

```java
// 带默认值输出
// 1.null时 二者差异
System.out.println("null时 二者差异");
System.out.println("xxxx.orElse(createString())");
Optional.ofNullable(null).orElse(createString());
System.out.println("xxxx.orElseGet(() -> createString())");
Optional.ofNullable(null).orElseGet(() -> createString());

// 2.非空时 二者差异
System.out.println("非空时 二者差异");
System.out.println("xxxx.orElse(createString())");
Optional.ofNullable("dave").orElse(createString());
System.out.println("xxxx.orElseGet(() -> createString())");
Optional.ofNullable("dave").orElseGet(() -> createString());
/* 输出结果：
null时 二者差异
xxxx.orElse(createString())
Creating New String
xxxx.orElseGet(() -> createString())
Creating New String
非空时 二者差异
xxxx.orElse(createString())
Creating New String
xxxx.orElseGet(() -> createString())
*/
```

- 返回异常
  
>orElseThrow()与orElseGet()相似，入参都是Supplier对象，只不过orElseThrow()的Supplier对象必须返回一个Throwable异常，并在orElseThrow()中将异常抛出。`orElseThrow()方法适用于包装对象值为空时需要抛出特定异常的场景。`

```java
// 自定义异常返回
System.out.println("orElseThrow()自定义异常返回：空值返回 IllegalArgumentException()");

Optional.ofNullable(null).orElseThrow(() -> new IllegalArgumentException());
```


#### 3.数值转换与过滤

>与map()方法不同的是，入参Function函数的返回值类型为Optional<U>类型，而不是U类型，这样flatMap()能将一个二维的Optional对象映射成一个一维的对象。

```java
//User类
private static class User {
    private String id;
    private String name;
    private String position;

    public User(String id, String name, String position) {
        this.id = id;
        this.name = name;
        this.position = position;
    }

    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getPosition() {
        return position;
    }
    // 为了说明flatMap()
    public Optional<String> getPositionOptional() {
        return Optional.ofNullable(position);
    }
}
```

```java
// map
System.out.println("map(byte[])自定义数值转换 ");
System.out.println("String \n" +
        "--> Optional<String> \n" +
        "--> Optional<byte[]> \n" +
        "--> byte[]\n");
System.out.println(
        Optional.ofNullable("piccolo is BIG DEVIL")
                .map(u -> u.getBytes(StandardCharsets.UTF_8))
                .orElse("super".getBytes(StandardCharsets.UTF_8))
);

System.out.println("map(String)自定义数值转换 ");
System.out.println("User Class \n" +
        "--> Optional<testOptional.User> \n" +
        "--> Optional<String> " +"map内参数为String\n"+
        "--> String\n");
System.out.println(
        Optional.ofNullable(new User("12","piccolo","chengdu"))
                .map(u -> u.getId())
                .orElse("defaultID:999")
);

System.out.println("flatMap(Optional<String>)自定义数值转换 ");
System.out.println("User Class \n" +
        "--> Optional<testOptional.User> \n" +
        "--> Optional<String> " +"flatMap内参数为Optional<String>\n"+
        "--> String\n");
System.out.println(
        Optional.ofNullable(new User("12","piccolo","chengdu"))
                .flatMap(u -> u.getPositionOptional())
                .orElse("default:beijing")
);
```

**过滤器**

```java
System.out.println(Optional.ofNullable("piccolo is BIG DEVIL")
        .filter(u -> u.contains("BIG")));
```

#### 附录：

```java
// get
// 值为空会跳NPE，因此往往与isPresent()或ifPresent()配套使用，先判定是否为空
public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}

// isPresent()
// 方法用于判断包装对象的值是否非空
public boolean isPresent() {
    return value != null;
}

// ifPresent()方法接受一个Consumer对象（消费函数），如果包装对象的值非空，运行Consumer对象的accept()方法。
// 做了null值检查，调用无需担心NPE问题
public void ifPresent(Consumer<? super T> consumer) {
if (value != null)
    consumer.accept(value);
}

// filter()方法接受参数为Predicate对象，用于对Optional对象进行过滤，如果符合Predicate的条件，返回Optional对象本身，否则返回一个空的Optional对象。
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent())
        return this;
    else
        return predicate.test(value) ? this : empty();
}

// map()方法的参数为Function（函数式接口）对象，map()方法将Optional中的包装对象用Function函数进行运算，并包装成新的Optional对象（包装对象的类型可能改变）。
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Optional.ofNullable(mapper.apply(value));
    }
}

// 与map()方法不同的是，入参Function函数的返回值类型为Optional<U>类型，而不是U类型，这样flatMap()能将一个二维的Optional对象映射成一个一维的对象。
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Objects.requireNonNull(mapper.apply(value));
    }
}

// orElse()方法，如果包装对象值非空，返回包装对象值，否则返回入参other的值（默认值）。
public T orElse(T other) {
    return value != null ? value : other;
}

// orElse()方法类似，区别在于orElseGet()方法的入参为一个Supplier对象，用Supplier对象的get()方法的返回值作为默认值。
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}

// orElseThrow()与orElseGet()相似，入参都是Supplier对象，只不过orElseThrow()的Supplier对象必须返回一个Throwable异常，并在orElseThrow()中将异常抛出
// orElseThrow()方法适用于包装对象值为空时需要抛出特定异常的场景。
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
    if (value != null) {
        return value;
    } else {
        throw exceptionSupplier.get();
    }
}
```

#### JDK9 Optional New

// todo

- or 

>or() 方法与 orElse() 和 orElseGet() 类似，它们都在对象为空的时候提供了替代情况。or() 的返回值是由 Supplier 参数产生的另一个 Optional 对象。如果对象包含值，则or() 方法内部不执行。

- ifPresentOrElse
  
>ifPresentOrElse() 方法需要两个参数：一个 Consumer 和一个 Runnable。如果对象包含值，会执行 Consumer 的动作，否则运行 Runnable

- stream

>通过把实例转换为 Stream 对象，让你从广大的 Stream API 中受益。如果没有值，它会得到空的 Stream；有值的情况下，Stream 则会包含单一值。