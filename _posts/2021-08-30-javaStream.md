---
layout: post
title: 'Java中Stream使用笔记'
subtitle: 'javaStream'
date: 2021-08-30
author: Dave
cover: ''
tags: Java
---

## Java
>最近开始工作，回顾一下，写个笔记。
本来8月份写完整了的，在u盘上遗失了，21/09/14重构一下本笔记。



**最前面**

Java 8 提供Stream API，使得集合操作开始变得优雅起来。

- 1.Stream创建
- 2.Stream中间操作（中间过程操作数据）
- 3.Stream终止操作（消费掉Stream流，若没有终止操作，则中间操作无法执行）



## stream流创建


```java
//Stream流创建
//#1.Collection的stream()方法
List<Integer> list=Arrays.asList(1,2,3,4,5,6,7,8,9);
//串行流
Stream<Integer> stream1 = list.stream();
//并行流
Stream<Integer> stream1a = list.parallelStream();
//#2.Arrays的静态方法stream()方法
Integer[] arrInteger =new Integer[]{1,2,3,4,5};
Stream<Integer> stream2 = Arrays.stream(arrInteger);
//#3.Stream类的静态方法 of()
Stream<Integer> stream3 = Stream.of(1,2,3,4,5,6);
//#4.迭代创建无限流
Stream<Integer> stream4 = Stream.iterate(0, (x) -> x + 1);
```

除此以外还有IntStream/LongStream/DoubleStream等,分别表示原始 int 流、 原始 long 流 和 原始 double 流。

创建int流都是由IntStream接口类中的静态方法完成的，具体如下：

- of / builder： 可指定int流中包含的具体单个元素；
- range / rangeClosed ： 将指定范围内的元素都添加到int流中，前者不包含最后一个元素，后者包含；
- generate / iterate :  指定生成int流中int元素的生成函数，前者的生成函数没有入参，后者会将前一次调用结果作为下一次调用生成函数的入参；
- concat ：将两个int流合并成一个。

```java
//IntStream of
IntStream intStream=IntStream.of(1,3,4,2);

//包含指定的元素,add方法底层也是调用accept方法，然后返回this
//IntStream builder
intStream=IntStream.builder().add(2020).add(2019).add(2021).build();


//range 左闭右开[start,end)
//从18到21,不包含21
intStream=IntStream.range(18,21);

//rangeClosed [start,end]
//从18到21,包含21
intStream=IntStream.rangeClosed(18,21);


//generate Lambda 无入参
intStream=IntStream.generate(()->{
  Random random=new Random();
  return random.nextInt(100);
}).limit(6);

//iterate Lambda 需要入参
//第一个参数seed就是第一次调用生成函数的入参
intStream=IntStream.iterate(1,x->{
  int a=2*x;
  //System.out.println(x);
  if(a>16){
      return a-20;
  }else{
      return a;
  }
}).limit(6);
```

## `terminal operation`
## stream流终止操作 

### 1.查找与匹配

```java
boolean anyMatch(Predicate<? super T> predicate);//检查是否至少匹配一个元素
boolean allMatch(Predicate<? super T> predicate);//检查是否匹配所有元素
boolean noneMatch(Predicate<? super T> predicate);//检查是否没有匹配的元素
Optional<T> findFirst();//返回第一个元素
Optional<T> findAny();//返回当前流中的任意元素
Optional<T> min(Comparator<? super T> comparator);//返回流中最小值
Optional<T> max(Comparator<? super T> comparator);//返回流中最大值
long count();//返回流中元素的总个数
```

示例

```java
Integer[] arrInteger =new Integer[]{1,2,3,4,5};
//##查找与匹配
//allMatch——检查是否匹配所有元素
boolean res1=Arrays.stream(arrInteger)
        .allMatch(x-> x>0);//检查是否所有元素均大于0
System.out.println(res1);//true

//findFirst——返回第一个元素
Integer res2=Arrays.stream(arrInteger)
        .findFirst()//返回Optional<Integer> 对象，需要get()才为Integer
        .get();
System.out.println(res2);//1

//count——返回流中元素的总个数
long res3=Arrays.stream(arrInteger).count();
System.out.println(res3);//5

//min-返回流中最小值 按自定义规则
Integer min=Arrays.stream(arrInteger).min((x,y)->y-x).get();
System.out.println(min);//5
```

### 2.迭代/循环

```java
void forEach(Consumer<? super T> action);//内部迭代遍历，串行流时有顺序，并行流下顺序将随机排列，
void forEachOrdered(Consumer<? super T> action);//并行流下顺序仍然被严格保持
```

示例

```java
Integer[] arrInteger =new Integer[]{1,2,3,4,5};
//##循环
Queue<Integer> queue = new ArrayDeque<>();
Arrays.stream(arrInteger).forEach(queue::add);
queue.stream().forEach(System.out::println);
```

### 3.归约

```java
T reduce(T identity, BinaryOperator<T> accumulator);//identity 是起始值, accumulator 二元运算 可以将流中元素反复结合起来，得到一个值。 比如: 求集合中元素的累加总和
Optional<T> reduce(BinaryOperator<T> accumulator);//无起始值，可以将流中元素反复结合起来，得到一个值。
<U> U reduce(U identity,
                BiFunction<U, ? super T, U> accumulator,
                BinaryOperator<U> combiner);//结合上述二者再新增
```

示例

```java
Integer[] arrInteger =new Integer[]{1,2,3,4,5};
//##归约
int sum0=Arrays.stream(arrInteger).reduce((x,y)->x+y).get();
int sum1=Arrays.stream(arrInteger).reduce(1,(x,y)->x+y);
int sum2=Arrays.stream(arrInteger).reduce(2,(x,y)->x+y,(x,y)->x-y);//
System.out.println(sum0);
System.out.println(sum1);
System.out.println(sum2);
//查看中间过程版本
System.out.println("sum");
int sum=Arrays.stream(arrInteger).reduce(1,(x,y)->{
    System.out.println("当前x为："+x+"当前y为："+y);
    return x+y;
});

System.out.println("dif");
int dif=Arrays.stream(arrInteger).reduce(1,(x,y)->{
    System.out.println("当前x为："+x+"当前y为："+y);
    return x-y;
});
/*
sum
当前x为：1当前y为：1
当前x为：2当前y为：2
当前x为：4当前y为：3
当前x为：7当前y为：4
当前x为：11当前y为：5
dif
当前x为：1当前y为：1
当前x为：0当前y为：2
当前x为：-2当前y为：3
当前x为：-5当前y为：4
当前x为：-9当前y为：5
*/
```


### 4.收集与转换

```java
Object[] toArray();//返回Object[]
<A> A[] toArray(IntFunction<A[]> generator);//返回对应类型Arrays

<R> R collect(Supplier<R> supplier,
                BiConsumer<R, ? super T> accumulator,
                BiConsumer<R, R> combiner);

<R, A> R collect(Collector<? super T, A, R> collector);
```

其中，`collect`具备多种转换


```java
//示例：
List<Integer> resList=list.stream().collect(Collectors.toList());

//收集转换 三大语义
toList()  //把流中元素收集到List<T>
toSet()   //把流中元素收集到Set<T>
toCollection()  // 把流中元素收集到创建的集合Collection<T>

//示例：
long count=list.stream().collect(Collectors.counting());
计数：count
平均值：averagingInt、averagingLong、averagingDouble
最值：maxBy、minBy
求和：summingInt、summingLong、summingDouble
统计以上所有：summarizingInt、summarizingLong、summarizingDouble

//Collectors下的多个方法
counting()  //计算流中元素的个数 Long
maxBy() //取最大值
minBy() //取最小值
summingInt() // 对流中元素的整数属性求和 Integer
summingLong()
summingDouble()
averagingInt() // 计算流中元素Integer属性的平均值Double
averagingLong() 
averagingDouble() 
summarizingInt() // 收集流中Integer属性的统计值 IntSummaryStatistics
summarizingLong()//LongSummaryStatistics
summarizingDouble()//DoubleSummaryStatistics

partitioningBy()//分区
groupingBy()//分组

joining()//可以将stream中的元素用特定的连接符（没有的话，则直接连接）连接成一个字符串。
reducing()//类比reduce()
```

示例1:基础收集转换

```java
List<Integer> list=Arrays.asList(1,2,3,4,5,6,7,8,9);
//收集转换
Object[] resObject=list.stream().toArray();
Integer[] resInteger= list.stream().toArray(Integer[]::new);
List<Integer> resList=list.stream().collect(Collectors.toList());

Queue<Integer> resQueue =list.stream().collect(Collectors.toCollection(ArrayDeque::new));
```

示例2:Collectors下的多个方法

```java
List<Integer> list= Arrays.asList(8,6,2,1,1,2,2,3,3,4,5,6,9);
//Collectors下的多个方法
long count=list.stream().collect(Collectors.counting());//13
Integer resInt=list.stream().collect(Collectors.summingInt(n->n));//52
Double resAverage=list.stream().collect(Collectors.averagingInt(n->n));//4.0
Integer max=list.stream()
        .collect(Collectors.maxBy((x,y)->x-y))
        .get();//9
//中间过程版本
Integer resInt1=list.stream().collect(Collectors.summingInt(n->{
    System.out.println("当前n：" + n);
    return n;
}));

IntSummaryStatistics summary=list.stream().collect(Collectors.summarizingInt(n->n));
System.out.println(summary);
/*
IntSummaryStatistics{count=13, sum=52, min=1, average=4.000000, max=9}
*/
String str=list.stream().map(String::valueOf).collect(Collectors.joining("-"));
System.out.println(str);
/*
8-6-2-1-1-2-2-3-3-4-5-6-9
*/
```

示例3:分区/分组

```java

public class Main {

    public static void main(String[] args) {
        List<User> list= Arrays.asList(
                new User("dave",25,"China"),
                new User("piccolo",20,"Japan"),
                new User("nora",27,"China"),
                new User("cathy",18,"Korea"),
                new User("mia",18,"Japan"),
                new User("dendy",29,"Wakanda"),
                new User("myra",31,"Korea"),
                new User("carl",22,"Wakanda")
        );

        //分区
        Map<Boolean, List<User>> mapByAge0=list.stream()
                .collect(Collectors.partitioningBy(x->x.getAge()>25));
        System.out.println(mapByAge0);
        //{
        // false=
        // [com.picc0lo.User@5f184fc6, com.picc0lo.User@3feba861, 
        // com.picc0lo.User@5b480cf9, com.picc0lo.User@6f496d9f, 
        // com.picc0lo.User@723279cf], 
        // true=
        // [com.picc0lo.User@10f87f48, com.picc0lo.User@b4c966a, 
        // com.picc0lo.User@2f4d3709]
        // }

        //分组
        Map<String, List<User>> mapByCountry=list.stream()
                .collect(Collectors.groupingBy(User::getCountry));
        System.out.println(mapByCountry);
        //{Japan=[com.picc0lo.User@3feba861, com.picc0lo.User@6f496d9f], 
        // China=[com.picc0lo.User@5f184fc6, com.picc0lo.User@10f87f48], 
        // Wakanda=[com.picc0lo.User@b4c966a, com.picc0lo.User@723279cf], 
        // Korea=[com.picc0lo.User@5b480cf9, com.picc0lo.User@2f4d3709]}
    }
}

class User{
    private String name;
    private int age;

    public User(String name, int age, String country) {
        this.name = name;
        this.age = age;
        this.country = country;
    }

    private String country;

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
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
```
## `intermediate operation`
## stream流中间操作 

### 1.排序

```java
Stream<T> sorted();//默认顺序 String字典序 Integer自然序
Stream<T> sorted(Comparator<? super T> comparator);//自定义比较器顺序
```

示例

```java
List<Integer> list= Arrays.asList(8,6,2,1,2,9);

list.stream()
        .sorted((x,y)-> y-x)
        .forEach(System.out::println);

//中间过程
list.stream()
        .sorted((x,y)-> {
            System.out.println("当前x："+x+"当前y："+y);
            return y-x;
        })
        .forEach(System.out::println);
/*
当前x：6当前y：8
当前x：2当前y：6
当前x：1当前y：2
当前x：2当前y：1
当前x：2当前y：2
当前x：2当前y：1
当前x：9当前y：2
当前x：9当前y：6
当前x：9当前y：8
9
8
6
2
2
1
*/
```

### 2.筛选/过滤

```java
Stream<T> filter(Predicate<? super T> predicate);//条件筛选
Stream<T> limit(long maxSize);//取前maxSize个元素
Stream<T> skip(long n);//跳过前n个元素
Stream<T> distinct();//去除重复
```

示例

```java
List<Integer> list= Arrays.asList(8,6,8,7,2,1,0,2,9);

list.stream()
        .filter(n->n>2)//8 6 8 7 9
        .limit(4)//8 6 8 7
        .distinct()//8 6 7
        .skip(1)//6 7
        .forEach(System.out::println);
```

### 3.映射

>map映射的中间操作主要体现在数据上的元素映射过程与转换（类型转换或数值转换）;`map 传入Lambda参数带返回值`
>
>peek 做一些输出，外部处理等中间操作，适合用于测试/验证过程。`peek 传入Lambda参数不带返回值`

```java
//接收 Lambda传参，执行中间处理操作
Stream<T> peek(Consumer<? super T> action);
//接收 Lambda传参，将元素转换成其他形式或提取信息;接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

其余语义


  
```java
//接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 IntStream。
IntStream mapToInt(ToIntFunction<? super T> mapper);
//接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 LongStream。
LongStream mapToLong(ToLongFunction<? super T> mapper); 
//接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 DoubleStream。
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);

//接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
//上述flatMap基础上产生新的IntStream
IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper);
//上述flatMap基础上产生新的LongStream
LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper);
//上述flatMap基础上产生新的DoubleStream
DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper);
```


示例

```java
//peek 用作中间输出做测试
Stream.of("one", "two", "three", "four")
        .filter(e -> e.length() > 3)
        .peek(e -> System.out.println("Filtered value: " + e))
        .map(String::toUpperCase)
        .peek(e -> System.out.println("Mapped value: " + e))
        .collect(Collectors.toList());

//.map 测试
int[] arr=new int[]{1,2,3,4,5};
Integer[] arrInteger =new Integer[]{1,2,3,4,5};

//map 做数据类型转换
//int[] Integer[]相互转换
int[] IntegerArr2IntArr = Arrays.stream(arrInteger)//转换为Stream<Integer>
        .mapToInt(Integer::valueOf)//Stream<Integer>调用Integer::valueOf来转成IntStream
        .toArray();

Integer[] IntArr2IntegerArr = Arrays
        .stream(arr)//转换为IntStream
        .boxed()//int 装箱为 Integer
        .toArray(Integer[]::new);

//map 做数据数值操作
Arrays.stream(arrInteger)//转换为Stream<Integer>
        .map(x->x+1)//每个元素进行加1操作
        .forEach(System.out::println);//2 3 4 5 6
```

- [java8 stream流操作的flatMap（流的扁平化）](https://blog.csdn.net/Mark_Chao/article/details/80810030)

flatMap 在集合合并与处理上

```java
//flatMap例子1
//例子来自《java8 实战》
String[] words = new String[]{"Hello","World"};
Arrays.stream(words)//Stream<String>
        .map(word -> word.split(""))//Stream<String[]>
        .distinct()//因为Stream中元素为String[]数组，两个数组不完全一样，所以不重复
        .collect(Collectors.toList())//List<String[]> List 中为String[]数组 所以打印的为两个地址
        .forEach(System.out::print);
/*
[Ljava.lang.String;@27d6c5e0[Ljava.lang.String;@4f3f5b241
*/
Arrays.stream(words)//Stream<String>
        .map(word -> word.split(""))//Stream<String[]>  Stream中元素为String[]数组
        .flatMap(Arrays::stream)//Stream<String>   拍平为一个分割好的Stream流
        .distinct()//去重可靠 
        .collect(Collectors.toList())//List<String>
        .forEach(System.out::print);
/*
HeloWrd1
*/
```

```java
//flatMap例子2:来自网络
//flatMap的复杂操作，实现List<Data1>和List<Data2>根据Id进行连接，将连接结果输出为一个List<OutputData>：
@Data
@AllArgsConstructor
public class Data1 {
    private int id;
    private String name;
    private int amount;
}
 
@Data
@AllArgsConstructor
public class Data2 {
    private int id;
    private String name;
    private String type;
}
 
@Data
@AllArgsConstructor
public class OutputData {
    private int id;
    private String name;
    private String type;
    private int amount;
}
 
 
@Test
public void intersectByKeyTest(){
    List<Data2> listOfData2 = new ArrayList<Data2>();
 
    listOfData2.add(new Data2(10501, "JOE"  , "Type1"));
    listOfData2.add(new Data2(10603, "SAL"  , "Type5"));
    listOfData2.add(new Data2(40514, "PETER", "Type4"));
    listOfData2.add(new Data2(59562, "JIM"  , "Type2"));
    listOfData2.add(new Data2(29415, "BOB"  , "Type1"));
    listOfData2.add(new Data2(61812, "JOE"  , "Type9"));
    listOfData2.add(new Data2(98432, "JOE"  , "Type7"));
    listOfData2.add(new Data2(62556, "JEFF" , "Type1"));
    listOfData2.add(new Data2(10599, "TOM"  , "Type4"));
 
 
    List<Data1> listOfData1 = new ArrayList<Data1>();
 
    listOfData1.add(new Data1(10501, "JOE"    ,3000000));
    listOfData1.add(new Data1(10603, "SAL"    ,6225000));
    listOfData1.add(new Data1(40514, "PETER"  ,2005000));
    listOfData1.add(new Data1(59562, "JIM"    ,3000000));
    listOfData1.add(new Data1(29415, "BOB"    ,3000000));

    List<OutputData> result = listOfData1.stream()
            .flatMap(x -> listOfData2.stream()
                    .filter(y -> x.getId() == y.getId())
                    .map(y -> new OutputData(y.getId(), x.getName(), y.getType(), x.getAmount())))
            .collect(Collectors.toList());
    System.out.println(result);
 
    /*difference by key*/
    List<Data1> data1IntersectResult = listOfData1.stream().filter(data1 -> listOfData2.stream().map(Data2::getId).collect(Collectors.toList()).contains(data1.getId())).collect(Collectors.toList());
    System.out.println(data1IntersectResult);
}
```


## stream流一些常用用法归类

### 1.序列求和 集合求和

```java
List<Integer> list= Arrays.asList(8,6,2,1,1,2,2,3,3,4,5,6,9);
int sum1=list.stream().reduce(0,(x,y)->x+y);
int sum2=list.stream().collect(Collectors.summingInt(n->n));
int sum3=list.stream().collect(Collectors.reducing(0,(x,y)->x+y));

int sum4=list.stream().mapToInt(Integer::valueOf).sum();//可Double Long型，更推荐此法

System.out.println("sum1:"+sum1);
System.out.println("sum2:"+sum1);
System.out.println("sum3:"+sum1);
System.out.println("sum4:"+sum1);
/*
sum1:52
sum2:52
sum3:52
sum4:52
*/

//BigDecimal 类

List<BigDecimal> list= Arrays.asList(new BigDecimal(8.3),new BigDecimal(9.6));
// create MathContext object with 4 precision
MathContext mc = new MathContext(4);
BigDecimal sumB1 =list.stream()
        .map(x->x.add(new BigDecimal(1)))
        .reduce(BigDecimal.ZERO,(x,y)-> x.add(y));

BigDecimal sumB2 =list.stream()
        .map(x->x.add(new BigDecimal(1)))
        .reduce(BigDecimal.ZERO,BigDecimal::add);

BigDecimal sumB3 =list.stream()
        .map(x->x.add(new BigDecimal(1)))
        .reduce(BigDecimal.ZERO,(x,y)->{
            return x.add(y,mc);
        });
System.out.println("sumB1:"+sumB1);
System.out.println("sumB2:"+sumB2);
System.out.println("sumB3:"+sumB3);
/*
sumB1:19.9000000000000003552713678800500929355621337890625
sumB2:19.9000000000000003552713678800500929355621337890625
sumB3:19.90
*/

//范式
//以User类的GetAge为例

//BigDecimal:
BigDecimal sumB =list.stream()
        .map(User::getAge)
        .reduce(BigDecimal.ZERO,BigDecimal::add);

//int/double/long:
double sum = list.stream()
        .mapToDouble(User::getAge)
        .sum();
```

### 2.集合操作

- 典型示例1
  
```java
//典型示例1：去重+过滤+循环添加
Integer[] arrInteger =new Integer[]{1,2,3,4,5,6,7,8,9,10,1,2,3,4,5};
Queue<Integer> queue = new ArrayDeque<>();

Arrays.stream(arrInteger)
        .distinct()
        .filter(x->x>3)
        .forEach(queue::offer);
```

- 典型示例2
  
```java
//典型示例2：交集/差集/并集/合并

List<Integer> list1= Arrays.asList(1,2,2,3,11,8,7,9,13);
List<Integer> list2= Arrays.asList(1,2,2,3,3,4,4,5,5,6,9);

//交集
List<Integer> intersectionList=list1.stream()
        .filter(list2::contains)
        .collect(Collectors.toList());
//1 2 2 3 9

//差集 list1中去除list2中存在的元素
List<Integer> differenceList=list1.stream()
        .filter(x->!list2.contains(x))
        .collect(Collectors.toList());
//11 8 7 13

//合并 list1 与 list2 完整合并
List<Integer> mergeList= Stream.of(list1,list2)
        .flatMap(Collection::stream)
        .collect(Collectors.toList());

//并集= 合并集-交集
//第一类：求并集整个过程考虑去除重复 简单
List<Integer> noRepeatUnionList=Stream.of(list1,list2)
        .flatMap(Collection::stream)
        .distinct()
        .collect(Collectors.toList());
//1,2,3,11,8,7,9,13,4,5,6 size=11


//第二类：求并集整个过程考虑均不去除重复
List<Integer> orignUnionList=Stream.of(differenceList,list2)
        .flatMap(Collection::stream)
        .collect(Collectors.toList());
//11,8,7,13,1,2,2,3,3,4,4,5,5,6,9 size=15

//第三类：求并集后需要考虑去除重复 
//与第一类殊途同归
List<Integer> orignUnionListDistinct=Stream.of(differenceList,list2)
        .flatMap(Collection::stream)
        .distinct()
        .collect(Collectors.toList());
//11,8,7,13,1,2,3,3,4,5,6,9 size=11

```

## IntStream/LongStream/DoubleStream的补充

>除上述操作外，这三类原始类型流，还具备多种方法。

```java
//常用
//终止操作
int sum();//求和
OptionalDouble average();//求平均
IntSummaryStatistics summaryStatistics();//求统计
//中间操作
Stream<Integer> boxed();//装箱为包装类
IntStream sequential();//返回顺序流
IntStream concat(IntStream a, IntStream b);

//举例：
Integer[] arrInteger=IntStream.range(18,21)//IntStream
        .boxed()//装箱为Stream<Integer>
        .toArray(Integer[]::new);//转为数组
```
