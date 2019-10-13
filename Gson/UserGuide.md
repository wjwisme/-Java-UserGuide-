# Gson User Guide

1. [Overview](#TOC-Overview)
2. [Goals for Gson](#TOC-Goals-for-Gson)
3. [Gson Performance and Scalability](#TOC-Gson-Performance-and-Scalability)
4. [Gson Users](#TOC-Gson-Users)
5. [Using Gson](#TOC-Using-Gson)
   * [Using Gson with Gradle/Android](#TOC-Gson-With-Gradle)
   * [Using Gson with Maven](#TOC-Gson-With-Maven)
   * [Primitives Examples](#TOC-Primitives-Examples)
   * [Object Examples](#TOC-Object-Examples)
   * [Finer Points with Objects](#TOC-Finer-Points-with-Objects)
   * [Nested Classes (including Inner Classes)](#TOC-Nested-Classes-including-Inner-Classes-)
   * [Array Examples](#TOC-Array-Examples)
   * [Collections Examples](#TOC-Collections-Examples)
     * [Collections Limitations](#TOC-Collections-Limitations)
   * [Serializing and Deserializing Generic Types](#TOC-Serializing-and-Deserializing-Generic-Types)
   * [Serializing and Deserializing Collection with Objects of Arbitrary Types](#TOC-Serializing-and-Deserializing-Collection-with-Objects-of-Arbitrary-Types)
   * [Built-in Serializers and Deserializers](#TOC-Built-in-Serializers-and-Deserializers)
   * [Custom Serialization and Deserialization](#TOC-Custom-Serialization-and-Deserialization)
     * [Writing a Serializer](#TOC-Writing-a-Serializer)
     * [Writing a Deserializer](#TOC-Writing-a-Deserializer)
   * [Writing an Instance Creator](#TOC-Writing-an-Instance-Creator)
     * [InstanceCreator for a Parameterized Type](#TOC-InstanceCreator-for-a-Parameterized-Type)
   * [Compact Vs. Pretty Printing for JSON Output Format](#TOC-Compact-Vs.-Pretty-Printing-for-JSON-Output-Format)
   * [Null Object Support](#TOC-Null-Object-Support)
   * [Versioning Support](#TOC-Versioning-Support)
   * [Excluding Fields From Serialization and Deserialization](#TOC-Excluding-Fields-From-Serialization-and-Deserialization)
     * [Java Modifier Exclusion](#TOC-Java-Modifier-Exclusion)
     * [Gson's `@Expose`](#TOC-Gson-s-Expose)
     * [User Defined Exclusion Strategies](#TOC-User-Defined-Exclusion-Strategies)
   * [JSON Field Naming Support](#TOC-JSON-Field-Naming-Support)
   * [Sharing State Across Custom Serializers and Deserializers](#TOC-Sharing-State-Across-Custom-Serializers-and-Deserializers)
   * [Streaming](#TOC-Streaming)
6. [Issues in Designing Gson](#TOC-Issues-in-Designing-Gson)
7. [Future Enhancements to Gson](#TOC-Future-Enhancements-to-Gson)

## <a name="TOC-Overview"></a>Overview

Gson is a Java library that can be used to convert Java Objects into their JSON representation. It can also be used to convert a JSON string to an equivalent Java object.

Gson can work with arbitrary Java objects including pre-existing objects that you do not have source code of.

## <a name="TOC-Overview"></a>概况

Gson是一个被用来把Java对象转化为JSON表示的Java库。它也能被用作将JSON串转化成等同的Java对象。

Gson能用来操作任意的Java对象，包括你使用的但没有源代码的对象。

## <a name="TOC-Goals-for-Gson"></a>Goals for Gson

* Provide easy to use mechanisms like `toString()` and constructor (factory method) to convert Java to JSON and vice-versa
* Allow pre-existing unmodifiable objects to be converted to and from JSON
* Allow custom representations for objects
* Support arbitrarily complex objects
* Generate compact and readable JSON output

## <a name="TOC-Goals-for-Gson"></a>Gson的目标

* 提供类似Java中`toString()`方法和构造器（工厂方法模型）这样的易用机制将Java和JSON相互转化
* 允许将已经存在但不能修改的Java对象转化为JSON和从JSON转化回来。
* 允许对象的自定义表示。
* 支持任意复杂对象
* 生成紧凑可读的JSON输出

## <a name="TOC-Gson-Performance-and-Scalability"></a>Gson Performance and Scalability

Here are some metrics that we obtained on a desktop (dual opteron, 8GB RAM, 64-bit Ubuntu) running lots of other things along-with the tests. You can rerun these tests by using the class [`PerformanceTest`](gson/src/test/java/com/google/gson/metrics/PerformanceTest.java).

* Strings: Deserialized strings of over 25MB without any problems (see `disabled_testStringDeserializationPerformance` method in `PerformanceTest`)
* Large collections:
  * Serialized a collection of 1.4 million objects (see `disabled_testLargeCollectionSerialization` method in `PerformanceTest`)
  * Deserialized a collection of 87,000 objects (see `disabled_testLargeCollectionDeserialization` in `PerformanceTest`)
* Gson 1.4 raised the deserialization limit for byte arrays and collection to over 11MB from 80KB.

Note: Delete the `disabled_` prefix to run these tests. We use this prefix to prevent running these tests every time we run JUnit tests.

## <a name="TOC-Gson-Performance-and-Scalability"></a>Gson性能和可伸缩性

下面有我们在桌面系统（双核  AMD *Opteron*(皓龙)处理器, 8GB RAM, 64-bit Ubuntu ）上运行测试获取的一些衡量指标. 你能通过测试类 [`PerformanceTest`](gson/src/test/java/com/google/gson/metrics/PerformanceTest.java)重现测试。

* Strings: 反序列化超过 25MB 的字符串没有任何问题 (see `disabled_testStringDeserializationPerformance` method in `PerformanceTest`)
* 大集合:
  * 序列化包含140万的对象的集合 (see `disabled_testLargeCollectionSerialization` method in `PerformanceTest`)
  * 反序列化包含87000对象的集合 (see `disabled_testLargeCollectionDeserialization` in `PerformanceTest`)
* Gson 1.4 将字节数组和集合的反序列化限制从80K提高到了超过11MB。

注意: 删除 `disabled_` 前缀后再运行测试. 我们使用这个前缀来阻止每一次运行JUnit测试都跑这些测试。

## <a name="TOC-Gson-Users"></a>Gson Users

Gson was originally created for use inside Google where it is currently used in a number of projects. It is now used by a number of public projects and companies.

## <a name="TOC-Gson-Users"></a>Gson 使用者

Gson 刚开始是为了Google内部创建，它当前被用在谷歌大量的项目中。现在它也被很多开源项目和机构使用。

## <a name="TOC-Using-Gson"></a>Using Gson

The primary class to use is [`Gson`](gson/src/main/java/com/google/gson/Gson.java) which you can just create by calling `new Gson()`. There is also a class [`GsonBuilder`](gson/src/main/java/com/google/gson/GsonBuilder.java) available that can be used to create a Gson instance with various settings like version control and so on.

The Gson instance does not maintain any state while invoking Json operations. So, you are free to reuse the same object for multiple Json serialization and deserialization operations.

## <a name="TOC-Using-Gson"></a>使用 Gson

要使用Gson，第一个需要的类是 [`Gson`](gson/src/main/java/com/google/gson/Gson.java) ，你能通过 `new Gson()`创建一个Gson实例. 也可以使用 [`GsonBuilder`](gson/src/main/java/com/google/gson/GsonBuilder.java) 类去创建 Gson 实例，使用[`GsonBuilder`](gson/src/main/java/com/google/gson/GsonBuilder.java) 可以同时设置如版本等变量.

当使用Gson实例调用Json操作时不维持任何状态. 所以你可以使用相同的Gson实例对象用作多次序列化和反序列化操作.

## <a name="TOC-Gson-With-Gradle"></a>Using Gson with Gradle/Android
```
dependencies {
    implementation 'com.google.code.gson:gson:2.8.6'
}
```


## <a name="TOC-Gson-With-Gradle"></a>Gradle/Android 使用Gson

```
dependencies {
    implementation 'com.google.code.gson:gson:2.8.6'
}
```



## <a name="TOC-Gson-With-Maven"></a>Using Gson with Maven

To use Gson with Maven2/3, you can use the Gson version available in Maven Central by adding the following dependency:

```xml
<dependencies>
    <!--  Gson: Java to Json conversion -->
    <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.8.6</version>
      <scope>compile</scope>
    </dependency>
</dependencies>
```

That is it, now your maven project is Gson enabled. 





## <a name="TOC-Gson-With-Maven"></a>Maven 使用Gson

在Maven2/3中使用Gson, 你能通过在pom.xml添加下面依赖使用Maven中央仓库中的可用的Gson版本:

```xml
<dependencies>
    <!--  Gson: Java to Json conversion -->
    <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.8.6</version>
      <scope>compile</scope>
    </dependency>
</dependencies>
```

好了，现在你的Maven项目可以使用Gson了. 

### <a name="TOC-Primitives-Examples"></a>简单例子

```java
// Serialization
Gson gson = new Gson();
gson.toJson(1);            // ==> 1
gson.toJson("abcd");       // ==> "abcd"
gson.toJson(new Long(10)); // ==> 10
int[] values = { 1 };
gson.toJson(values);       // ==> [1]

// Deserialization
int one = gson.fromJson("1", int.class);
Integer one = gson.fromJson("1", Integer.class);
Long one = gson.fromJson("1", Long.class);
Boolean false = gson.fromJson("false", Boolean.class);
String str = gson.fromJson("\"abc\"", String.class);
String[] anotherStr = gson.fromJson("[\"abc\"]", String[].class);
```

### <a name="TOC-Object-Examples"></a>对象使用例子

```java
class BagOfPrimitives {
  private int value1 = 1;
  private String value2 = "abc";
  private transient int value3 = 3;
  BagOfPrimitives() {
    // no-args constructor
  }
}

// Serialization
BagOfPrimitives obj = new BagOfPrimitives();
Gson gson = new Gson();
String json = gson.toJson(obj);  

// ==> json is {"value1":1,"value2":"abc"}
```

注意，你不能使用循环索引序列化对象，因为这将导致陷入无限循环.

```java
// Deserialization
BagOfPrimitives obj2 = gson.fromJson(json, BagOfPrimitives.class);
// ==> obj2 is just like obj
```

#### <a name="TOC-Finer-Points-with-Objects"></a>**Finer Points with Objects**

* It is perfectly fine (and recommended) to use private fields.
* There is no need to use any annotations to indicate a field is to be included for serialization and deserialization. All fields in the current class (and from all super classes) are included by default.
* If a field is marked transient, (by default) it is ignored and not included in the JSON serialization or deserialization.
* This implementation handles nulls correctly.
  * While serializing, a null field is omitted from the output.
  * While deserializing, a missing entry in JSON results in setting the corresponding field in the object to its default value: null for object types, zero for numeric types, and false for booleans.
* If a field is _synthetic_, it is ignored and not included in JSON serialization or deserialization.
* Fields corresponding to the outer classes in inner classes, anonymous classes, and local classes are ignored and not included in serialization or deserialization.



#### <a name="TOC-Finer-Points-with-Objects"></a>**对象使用细则**

* 推荐使用私有属性（private field）.
* 不必使用任何注释表示该属性将会被序列化或反序列化。当前类的所有属性（和从父类继承的属性）默认会被序列化和反序列化.
* 如果属性被 transient 修饰, 默认不被序列化和反序列化.
* 以下能正确处理null.
  * 当序列化的时候, null属性被从输出里忽略.
  * 当反序列化的时候, JSON串中缺少的属性会被反序列化为默认值: 对象类型为null, 数字类型为0, boolean类型为false.
* 如果属性被修饰为 _synthetic_（编译器添加的）, 该属性将会被忽视，不会被序列化和反序列化.
* Fields corresponding to the outer classes in inner classes, anonymous classes, and local classes are ignored and not included in serialization or deserialization.

### <a name="TOC-Nested-Classes-including-Inner-Classes-"></a>Nested Classes (including Inner Classes)

Gson can serialize static nested classes quite easily.

Gson can also deserialize static nested classes. However, Gson can **not** automatically deserialize the **pure inner classes since their no-args constructor also need a reference to the containing Object** which is not available at the time of deserialization. You can address this problem by either making the inner class static or by providing a custom InstanceCreator for it. Here is an example:

```java
public class A { 
  public String a; 

  class B { 

    public String b; 

    public B() {
      // No args constructor for B
    }
  } 
}
```

**NOTE**: The above class B can not (by default) be serialized with Gson.

Gson can not deserialize `{"b":"abc"}` into an instance of B since the class B is an inner class. If it was defined as static class B then Gson would have been able to deserialize the string. Another solution is to write a custom instance creator for B. 

```java
public class InstanceCreatorForB implements InstanceCreator<A.B> {
  private final A a;
  public InstanceCreatorForB(A a)  {
    this.a = a;
  }
  public A.B createInstance(Type type) {
    return a.new B();
  }
}
```

The above is possible, but not recommended.







### <a name="TOC-Nested-Classes-including-Inner-Classes-"></a>嵌套类 (包括内部类)

Gson 可以很轻松的序列化静态嵌套类.

Gson 也能反序列化静态嵌套类. 然而, Gson 不能自动反序列化 the **pure inner classes since their no-args constructor also need a reference to the containing Object** which is not available at the time of deserialization. You can address this problem by either making the inner class static or by providing a custom InstanceCreator for it. 下面有一个例子:

```java
public class A { 
  public String a; 

  class B { 

    public String b; 

    public B() {
      // No args constructor for B
    }
  } 
}
```

**注意**: 上面的B类默认不能被Gson反序列化.

Gson 不能反序列化 `{"b":"abc"}` 成为一个 B 实例，因为B类是一个内部类. 如果 B 被定义为静态类，那么Gson将会可以反序列化这段JSON串. 另一种解决办法是为B类专门写一个creator. 

```java
public class InstanceCreatorForB implements InstanceCreator<A.B> {
  private final A a;
  public InstanceCreatorForB(A a)  {
    this.a = a;
  }
  public A.B createInstance(Type type) {
    return a.new B();
  }
}
```

上面可以用，但不推荐使用.

### <a name="TOC-Array-Examples"></a>数组示例

```java
Gson gson = new Gson();
int[] ints = {1, 2, 3, 4, 5};
String[] strings = {"abc", "def", "ghi"};

// Serialization
gson.toJson(ints);     // ==> [1,2,3,4,5]
gson.toJson(strings);  // ==> ["abc", "def", "ghi"]

// Deserialization
int[] ints2 = gson.fromJson("[1,2,3,4,5]", int[].class); 
// ==> ints2 will be same as ints
```

我们也支持任意复杂类型的多维数组.

### <a name="TOC-Collections-Examples"></a>集合示例

```java
Gson gson = new Gson();
Collection<Integer> ints = Lists.immutableList(1,2,3,4,5);

// Serialization
String json = gson.toJson(ints);  // ==> json is [1,2,3,4,5]

// Deserialization
Type collectionType = new TypeToken<Collection<Integer>>(){}.getType();
Collection<Integer> ints2 = gson.fromJson(json, collectionType);
// ==> ints2 is same as ints
```

很可怕啊: 请记住我们怎样在反序列化的时候定义集合类型.
不幸的是，在Java中，上面的问题暂时还没有解决办法.

#### <a name="TOC-Collections-Limitations"></a>Collections Limitations

Gson can serialize collection of arbitrary objects but can not deserialize from it, because there is no way for the user to indicate the type of the resulting object. Instead, while deserializing, the Collection must be of a specific, generic type.
This makes sense, and is rarely a problem when following good Java coding practices.

#### <a name="TOC-Collections-Limitations"></a>集合限制

Gson 能序列化任意类型的集合但是不能从它反序列化, 因为没有办法让使用者去标识结果对象类型. 因此，在反序列化的时候，集合必须是一个指定的泛型.
这很有意义, 并且当我们遵守良好的Java编码规范时很少出问题.

### <a name="TOC-Serializing-and-Deserializing-Generic-Types"></a>Serializing and Deserializing Generic Types

When you call `toJson(obj)`, Gson calls `obj.getClass()` to get information on the fields to serialize. Similarly, you can typically pass `MyClass.class` object in the `fromJson(json, MyClass.class)` method. This works fine if the object is a non-generic type. However, if the object is of a generic type, then the Generic type information is lost because of Java Type Erasure. Here is an example illustrating the point:

```java
class Foo<T> {
  T value;
}
Gson gson = new Gson();
Foo<Bar> foo = new Foo<Bar>();
gson.toJson(foo); // May not serialize foo.value correctly

gson.fromJson(json, foo.getClass()); // Fails to deserialize foo.value as Bar
```

The above code fails to interpret value as type Bar because Gson invokes `foo.getClass()` to get its class information, but this method returns a raw class, `Foo.class`. This means that Gson has no way of knowing that this is an object of type `Foo<Bar>`, and not just plain `Foo`.

You can solve this problem by specifying the correct parameterized type for your generic type. You can do this by using the [`TypeToken`](https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/reflect/TypeToken.html) class.

```java
Type fooType = new TypeToken<Foo<Bar>>() {}.getType();
gson.toJson(foo, fooType);

gson.fromJson(json, fooType);
```
The idiom used to get `fooType` actually defines an anonymous local inner class containing a method `getType()` that returns the fully parameterized type.





### <a name="TOC-Serializing-and-Deserializing-Generic-Types"></a>序列化和反序列化泛型

当你调用`toJson(obj)`时, Gson 调用 `obj.getClass()` 去获取属性的序列化信息.  你可以在`fromJson(json, MyClass.class)` 方法中传递`MyClass.class`对象. 如果对象不是一个泛型类型，这将会工作很好的. 然而, 如果该对象是一个泛型, 那么泛型类型会因为Java类型不确定而丢失. 下面通过一个例子来说明这个知识点:

```java
class Foo<T> {
  T value;
}
Gson gson = new Gson();
Foo<Bar> foo = new Foo<Bar>();
gson.toJson(foo); // May not serialize foo.value correctly

gson.fromJson(json, foo.getClass()); // Fails to deserialize foo.value as Bar
```

上面的代码解释Bar类型失败，因为 Gson 通过调用`foo.getClass()` 去获取类信息, 但是这个方法返回一个原始class字节码（译者注:不包含泛型信息）, `Foo.class`. 这意味着 Gson 没办法知道这是一个 `Foo<Bar>`类型的对象, 没办法知道该类不仅仅是一个普通的 `Foo`.

你可以通过指定正确的参数类型为你的泛型类来解决这个问题. 你能通过 [`TypeToken`](https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/reflect/TypeToken.html) 类来做这件事.

```java
Type fooType = new TypeToken<Foo<Bar>>() {}.getType();
gson.toJson(foo, fooType);

gson.fromJson(json, fooType);
```

被用来获取 `fooType`的语句实际上定义了一个匿名内部类，该匿名内部类包含一个 返回完整参数类型的返回值的方法 `getType()` .

### <a name="TOC-Serializing-and-Deserializing-Collection-with-Objects-of-Arbitrary-Types"></a>Serializing and Deserializing Collection with Objects of Arbitrary Types

Sometimes you are dealing with JSON array that contains mixed types. For example:
`['hello',5,{name:'GREETINGS',source:'guest'}]`

The equivalent `Collection` containing this is:

```java
Collection collection = new ArrayList();
collection.add("hello");
collection.add(5);
collection.add(new Event("GREETINGS", "guest"));
```

where the `Event` class is defined as:

```java
class Event {
  private String name;
  private String source;
  private Event(String name, String source) {
    this.name = name;
    this.source = source;
  }
}
```

You can serialize the collection with Gson without doing anything specific: `toJson(collection)` would write out the desired output.

However, deserialization with `fromJson(json, Collection.class)` will not work since Gson has no way of knowing how to map the input to the types. Gson requires that you provide a genericised version of collection type in `fromJson()`. So, you have three options:

1. Use Gson's parser API (low-level streaming parser or the DOM parser JsonParser) to parse the array elements and then use `Gson.fromJson()` on each of the array elements.This is the preferred approach. [Here is an example](extras/src/main/java/com/google/gson/extras/examples/rawcollections/RawCollectionsExample.java) that demonstrates how to do this.

2. Register a type adapter for `Collection.class` that looks at each of the array members and maps them to appropriate objects. The disadvantage of this approach is that it will screw up deserialization of other collection types in Gson.

3. Register a type adapter for `MyCollectionMemberType` and use `fromJson()` with `Collection<MyCollectionMemberType>`.

This approach is practical only if the array appears as a top-level element or if you can change the field type holding the collection to be of type `Collection<MyCollectionMemberType>`.



### <a name="TOC-Serializing-and-Deserializing-Collection-with-Objects-of-Arbitrary-Types"></a>序列化和反序列化包含任意类型对象的集合

有时候，你需要处理包含多种混合类型的JSON数组. 举个例子:
`['hello',5,{name:'GREETINGS',source:'guest'}]`

这等同于 `Collection` 包含如下所示:

```java
Collection collection = new ArrayList();
collection.add("hello");
collection.add(5);
collection.add(new Event("GREETINGS", "guest"));
```

 `Event` 类定义:

```java
class Event {
  private String name;
  private String source;
  private Event(String name, String source) {
    this.name = name;
    this.source = source;
  }
}
```

你可以使用 Gson 不需要做其他额外操作来序列化集合 : 使用`toJson(collection)` 你就能获得期望的输出.

然而, 使用 `fromJson(json, Collection.class)` 反序列化就没有这么便捷了 因为Gson 没有办法知道怎样将JSON串的输入映射成为具体类型. Gson 需要你在`fromJson()`方法中提供一个集合类型的泛化版本. 所以，你有以下三个选择:

1. 使用 Gson's 解析器 API (低级的流处理解析器或者 DOM 解析器 JsonParser) 去解析数组元素 然后使用 `Gson.fromJson()` 遍历处理每个数组元素.这是首选方法. [示例](extras/src/main/java/com/google/gson/extras/examples/rawcollections/RawCollectionsExample.java) 展示如何去做.

2.  为 `Collection.class` 注册一个类型适配器，遍历每个数组成员并且把他们映射成对象. 这种方法的缺陷是它将会破坏Gson中的其他的集合类型.

3. 为 `MyCollectionMemberType` 注册一个类型的适配器并且 将 `Collection<MyCollectionMemberType>`与 `fromJson()` 一起使用.

仅仅当数组元素为顶级（top-level）元素时这个方法才适用或者你可以将保存这个集合属性类型修改为 `Collection<MyCollectionMemberType>`.

### <a name="TOC-Built-in-Serializers-and-Deserializers"></a>Built-in Serializers and Deserializers

Gson has built-in serializers and deserializers for commonly used classes whose default representation may be inappropriate.
Here is a list of such classes:

1. `java.net.URL` to match it with strings like `"https://github.com/google/gson/"`
2. `java.net.URI` to match it with strings like `"/google/gson/"`

You can also find source code for some commonly used classes such as JodaTime at [this page](https://sites.google.com/site/gson/gson-type-adapters-for-common-classes-1).



### <a name="TOC-Built-in-Serializers-and-Deserializers"></a>内置的序列化器和反序列化器

Gson has built-in serializers and deserializers for commonly used classes whose default representation may be inappropriate.
Here is a list of such classes:

1. `java.net.URL` to match it with strings like `"https://github.com/google/gson/"`
2. `java.net.URI` to match it with strings like `"/google/gson/"`

You can also find source code for some commonly used classes such as JodaTime at [this page](https://sites.google.com/site/gson/gson-type-adapters-for-common-classes-1).

### <a name="TOC-Custom-Serialization-and-Deserialization"></a>Custom Serialization and Deserialization

Sometimes default representation is not what you want. This is often the case when dealing with library classes (DateTime, etc).
Gson allows you to register your own custom serializers and deserializers. This is done by defining two parts:

* Json Serializers: Need to define custom serialization for an object
* Json Deserializers: Needed to define custom deserialization for a type

* Instance Creators: Not needed if no-args constructor is available or a deserializer is registered

```java
GsonBuilder gson = new GsonBuilder();
gson.registerTypeAdapter(MyType2.class, new MyTypeAdapter());
gson.registerTypeAdapter(MyType.class, new MySerializer());
gson.registerTypeAdapter(MyType.class, new MyDeserializer());
gson.registerTypeAdapter(MyType.class, new MyInstanceCreator());
```

`registerTypeAdapter` call checks if the type adapter implements more than one of these interfaces and register it for all of them.

#### <a name="TOC-Writing-a-Serializer"></a>Writing a Serializer

Here is an example of how to write a custom serializer for JodaTime `DateTime` class.

```java
private class DateTimeSerializer implements JsonSerializer<DateTime> {
  public JsonElement serialize(DateTime src, Type typeOfSrc, JsonSerializationContext context) {
    return new JsonPrimitive(src.toString());
  }
}
```

Gson calls `serialize()` when it runs into a `DateTime` object during serialization.

#### <a name="TOC-Writing-a-Deserializer"></a>Writing a Deserializer

Here is an example of how to write a custom deserializer for JodaTime DateTime class.

```java
private class DateTimeDeserializer implements JsonDeserializer<DateTime> {
  public DateTime deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context)
      throws JsonParseException {
    return new DateTime(json.getAsJsonPrimitive().getAsString());
  }
}
```

Gson calls `deserialize` when it needs to deserialize a JSON string fragment into a DateTime object

**Finer points with Serializers and Deserializers**

Often you want to register a single handler for all generic types corresponding to a raw type

* For example, suppose you have an `Id` class for id representation/translation (i.e. an internal vs. external representation).
* `Id<T>` type that has same serialization for all generic types
  * Essentially write out the id value
* Deserialization is very similar but not exactly the same
  * Need to call `new Id(Class<T>, String)` which returns an instance of `Id<T>`

Gson supports registering a single handler for this. You can also register a specific handler for a specific generic type (say `Id<RequiresSpecialHandling>` needed special handling).
The `Type` parameter for the `toJson()` and `fromJson()` contains the generic type information to help you write a single handler for all generic types corresponding to the same raw type.

### <a name="TOC-Writing-an-Instance-Creator"></a>Writing an Instance Creator

While deserializing an Object, Gson needs to create a default instance of the class.
Well-behaved classes that are meant for serialization and deserialization should have a no-argument constructor.

* Doesn't matter whether public or private

Typically, Instance Creators are needed when you are dealing with a library class that does NOT define a no-argument constructor

**Instance Creator Example**

```java
private class MoneyInstanceCreator implements InstanceCreator<Money> {
  public Money createInstance(Type type) {
    return new Money("1000000", CurrencyCode.USD);
  }
}
```

Type could be of a corresponding generic type

* Very useful to invoke constructors which need specific generic type information
* For example, if the `Id` class stores the class for which the Id is being created

#### <a name="TOC-InstanceCreator-for-a-Parameterized-Type"></a>InstanceCreator for a Parameterized Type

Sometimes the type that you are trying to instantiate is a parameterized type. Generally, this is not a problem since the actual instance is of raw type. Here is an example:

```java
class MyList<T> extends ArrayList<T> {
}

class MyListInstanceCreator implements InstanceCreator<MyList<?>> {
    @SuppressWarnings("unchecked")
  public MyList<?> createInstance(Type type) {
    // No need to use a parameterized list since the actual instance will have the raw type anyway.
    return new MyList();
  }
}
```

However, sometimes you do need to create instance based on the actual parameterized type. In this case, you can use the type parameter being passed to the `createInstance` method. Here is an example:

```java
public class Id<T> {
  private final Class<T> classOfId;
  private final long value;
  public Id(Class<T> classOfId, long value) {
    this.classOfId = classOfId;
    this.value = value;
  }
}

class IdInstanceCreator implements InstanceCreator<Id<?>> {
  public Id<?> createInstance(Type type) {
    Type[] typeParameters = ((ParameterizedType)type).getActualTypeArguments();
    Type idType = typeParameters[0]; // Id has only one parameterized type T
    return Id.get((Class)idType, 0L);
  }
}
```

In the above example, an instance of the Id class can not be created without actually passing in the actual type for the parameterized type. We solve this problem by using the passed method parameter, `type`. The `type` object in this case is the Java parameterized type representation of `Id<Foo>` where the actual instance should be bound to `Id<Foo>`. Since `Id` class has just one parameterized type parameter, `T`, we use the zeroth element of the type array returned by `getActualTypeArgument()` which will hold `Foo.class` in this case.

### <a name="TOC-Compact-Vs.-Pretty-Printing-for-JSON-Output-Format"></a>Compact Vs. Pretty Printing for JSON Output Format

The default JSON output that is provided by Gson is a compact JSON format. This means that there will not be any whitespace in the output JSON structure. Therefore, there will be no whitespace between field names and its value, object fields, and objects within arrays in the JSON output. As well, "null" fields will be ignored in the output (NOTE: null values will still be included in collections/arrays of objects). See the [Null Object Support](#TOC-Null-Object-Support) section for information on configure Gson to output all null values.

If you would like to use the Pretty Print feature, you must configure your `Gson` instance using the `GsonBuilder`. The `JsonFormatter` is not exposed through our public API, so the client is unable to configure the default print settings/margins for the JSON output. For now, we only provide a default `JsonPrintFormatter` that has default line length of 80 character, 2 character indentation, and 4 character right margin.

The following is an example shows how to configure a `Gson` instance to use the default `JsonPrintFormatter` instead of the `JsonCompactFormatter`:
```
Gson gson = new GsonBuilder().setPrettyPrinting().create();
String jsonOutput = gson.toJson(someObject);
```
### <a name="TOC-Null-Object-Support"></a>Null Object Support

The default behaviour that is implemented in Gson is that `null` object fields are ignored. This allows for a more compact output format; however, the client must define a default value for these fields as the JSON format is converted back into its Java form.

Here's how you would configure a `Gson` instance to output null:

```java
Gson gson = new GsonBuilder().serializeNulls().create();
```

NOTE: when serializing `null`s with Gson, it will add a `JsonNull` element to the `JsonElement` structure. Therefore, this object can be used in custom serialization/deserialization.

Here's an example:

```java
public class Foo {
  private final String s;
  private final int i;

  public Foo() {
    this(null, 5);
  }

  public Foo(String s, int i) {
    this.s = s;
    this.i = i;
  }
}

Gson gson = new GsonBuilder().serializeNulls().create();
Foo foo = new Foo();
String json = gson.toJson(foo);
System.out.println(json);

json = gson.toJson(null);
System.out.println(json);
```

The output is:

```
{"s":null,"i":5}
null
```

### <a name="TOC-Versioning-Support"></a>Versioning Support

Multiple versions of the same object can be maintained by using [@Since](gson/src/main/java/com/google/gson/annotations/Since.java) annotation. This annotation can be used on Classes, Fields and, in a future release, Methods. In order to leverage this feature, you must configure your `Gson` instance to ignore any field/object that is greater than some version number. If no version is set on the `Gson` instance then it will serialize and deserialize all fields and classes regardless of the version.

```java
public class VersionedClass {
  @Since(1.1) private final String newerField;
  @Since(1.0) private final String newField;
  private final String field;

  public VersionedClass() {
    this.newerField = "newer";
    this.newField = "new";
    this.field = "old";
  }
}

VersionedClass versionedObject = new VersionedClass();
Gson gson = new GsonBuilder().setVersion(1.0).create();
String jsonOutput = gson.toJson(versionedObject);
System.out.println(jsonOutput);
System.out.println();

gson = new Gson();
jsonOutput = gson.toJson(versionedObject);
System.out.println(jsonOutput);
```

The output is:

```
{"newField":"new","field":"old"}

{"newerField":"newer","newField":"new","field":"old"}
```

### <a name="TOC-Excluding-Fields-From-Serialization-and-Deserialization"></a>Excluding Fields From Serialization and Deserialization

Gson supports numerous mechanisms for excluding top-level classes, fields and field types. Below are pluggable mechanisms that allow field and class exclusion. If none of the below mechanisms satisfy your needs then you can always use [custom serializers and deserializers](#TOC-Custom-Serialization-and-Deserialization).

#### <a name="TOC-Java-Modifier-Exclusion"></a>Java Modifier Exclusion

By default, if you mark a field as `transient`, it will be excluded. As well, if a field is marked as `static` then by default it will be excluded. If you want to include some transient fields then you can do the following:

```java
import java.lang.reflect.Modifier;
Gson gson = new GsonBuilder()
    .excludeFieldsWithModifiers(Modifier.STATIC)
    .create();
```

NOTE: you can give any number of the `Modifier` constants to the `excludeFieldsWithModifiers` method. For example:

```java
Gson gson = new GsonBuilder()
    .excludeFieldsWithModifiers(Modifier.STATIC, Modifier.TRANSIENT, Modifier.VOLATILE)
    .create();
```

#### <a name="TOC-Gson-s-Expose"></a>Gson's `@Expose`

This feature provides a way where you can mark certain fields of your objects to be excluded for consideration for serialization and deserialization to JSON. To use this annotation, you must create Gson by using `new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()`. The Gson instance created will exclude all fields in a class that are not marked with `@Expose` annotation.

#### <a name="TOC-User-Defined-Exclusion-Strategies"></a>User Defined Exclusion Strategies

If the above mechanisms for excluding fields and class type do not work for you then you can always write your own exclusion strategy and plug it into Gson. See the [`ExclusionStrategy`](https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/ExclusionStrategy.html) JavaDoc for more information.

The following example shows how to exclude fields marked with a specific `@Foo` annotation and excludes top-level types (or declared field type) of class `String`.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD})
public @interface Foo {
  // Field tag only annotation
}

public class SampleObjectForTest {
  @Foo private final int annotatedField;
  private final String stringField;
  private final long longField;
  private final Class<?> clazzField;

  public SampleObjectForTest() {
    annotatedField = 5;
    stringField = "someDefaultValue";
    longField = 1234;
  }
}

public class MyExclusionStrategy implements ExclusionStrategy {
  private final Class<?> typeToSkip;

  private MyExclusionStrategy(Class<?> typeToSkip) {
    this.typeToSkip = typeToSkip;
  }

  public boolean shouldSkipClass(Class<?> clazz) {
    return (clazz == typeToSkip);
  }

  public boolean shouldSkipField(FieldAttributes f) {
    return f.getAnnotation(Foo.class) != null;
  }
}

public static void main(String[] args) {
  Gson gson = new GsonBuilder()
      .setExclusionStrategies(new MyExclusionStrategy(String.class))
      .serializeNulls()
      .create();
  SampleObjectForTest src = new SampleObjectForTest();
  String json = gson.toJson(src);
  System.out.println(json);
}
```

The output is:

```
{"longField":1234}
```

### <a name="TOC-JSON-Field-Naming-Support"></a>JSON Field Naming Support

Gson supports some pre-defined field naming policies to convert the standard Java field names (i.e., camel cased names starting with lower case --- `sampleFieldNameInJava`) to a Json field name (i.e., `sample_field_name_in_java` or `SampleFieldNameInJava`). See the [FieldNamingPolicy](https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/FieldNamingPolicy.html) class for information on the pre-defined naming policies.

It also has an annotation based strategy to allows clients to define custom names on a per field basis. Note, that the annotation based strategy has field name validation which will raise "Runtime" exceptions if an invalid field name is provided as the annotation value.

The following is an example of how to use both Gson naming policy features:

```java
private class SomeObject {
  @SerializedName("custom_naming") private final String someField;
  private final String someOtherField;

  public SomeObject(String a, String b) {
    this.someField = a;
    this.someOtherField = b;
  }
}

SomeObject someObject = new SomeObject("first", "second");
Gson gson = new GsonBuilder().setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE).create();
String jsonRepresentation = gson.toJson(someObject);
System.out.println(jsonRepresentation);
```

The output is:

```
{"custom_naming":"first","SomeOtherField":"second"}
```

If you have a need for custom naming policy ([see this discussion](https://groups.google.com/group/google-gson/browse_thread/thread/cb441a2d717f6892)), you can use the [@SerializedName](https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/annotations/SerializedName.html) annotation.

### <a name="TOC-Sharing-State-Across-Custom-Serializers-and-Deserializers"></a>Sharing State Across Custom Serializers and Deserializers

Sometimes you need to share state across custom serializers/deserializers ([see this discussion](https://groups.google.com/group/google-gson/browse_thread/thread/2850010691ea09fb)). You can use the following three strategies to accomplish this:

1. Store shared state in static fields
2. Declare the serializer/deserializer as inner classes of a parent type, and use the instance fields of parent type to store shared state
3. Use Java `ThreadLocal`

1 and 2 are not thread-safe options, but 3 is.

### <a name="TOC-Streaming"></a>Streaming

In addition Gson's object model and data binding, you can use Gson to read from and write to a [stream](https://sites.google.com/site/gson/streaming). You can also combine streaming and object model access to get the best of both approaches.

## <a name="TOC-Issues-in-Designing-Gson"></a>Issues in Designing Gson

See the [Gson design document](https://github.com/google/gson/blob/master/GsonDesignDocument.md "Gson design document") for a discussion of issues we faced while designing Gson. It also include a comparison of Gson with other Java libraries that can be used for Json conversion.

## <a name="TOC-Future-Enhancements-to-Gson"></a>Future Enhancements to Gson

For the latest list of proposed enhancements or if you'd like to suggest new ones, see the [Issues section](https://github.com/google/gson/issues) under the project website.
