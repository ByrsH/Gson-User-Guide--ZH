# [Gson 用户指南](https://github.com/google/gson/blob/master/UserGuide.md#gson-user-guide)

## 概述

Gson 是一个能够用于转换 Java 对象到JSON表达式的 Java库。它当然也可以用于转换一个 JSON 字符串到等价的 Java 对象。

Gson 能够处理任意的 Java对象，包括你没有源代码的现有对象。


## Gson的目标

-  提供简单的使用方式，例如 toString() 和 构造器（工厂方法）用于Java 和 JSON之间的转换。


- 允许已有的不可改变的对象转换为 JSON，或从JSON转换为对象。 


- 允许自定义表示对象



- 支持任意的复合对象


- 生成紧凑的和易读的 JSON 输出



## Gson 性能和可扩展性

这里有一些度量数据，它们是在一个台式机（双处理器，8GB内存、64位的Ubuntu）上一起运行许多其他事物测试得到的。你可以重新运行这些例子，通过 [PerformanceTest](https://github.com/google/gson/blob/master/gson/src/test/java/com/google/gson/metrics/PerformanceTest.java)类。




-  字符串：反序列化超过25MB的字符串没有任何问题（查看类 PerformanceTest 中 disabled_testStringDeserializationPerformance 方法）


- 大集合：

    序列化一个140万个对象的集合（查看类 PerformanceTest 中 disabled_testLargeCollectionSerialization 方法）

    反序列化一个87000个对象的集合（查看类 PerformanceTest 中 disabled_testLargeCollection Deserialization 方法 ）


- Gson 1.4版本提高了反序列化数组和集合字节的上线限制，从80KB提升到了11MB


注意：运行这些测试例子时去掉 disabled_ 前缀。我们使用这个前缀是为了防止每次运行 JUnit 测试时运行这些例子。
           

## Gson 用户

Gson 最初被创建是用于Google内部的一些项目，现在被用于很多开源项目和公司。


## 使用Gson

主要的类是[Gson](https://github.com/google/gson/blob/master/gson/src/main/java/com/google/gson/Gson.java)，你可以仅仅调用 new Gson() 来创建一个实例。当然也有一个类的 [GsonBuilder](https://github.com/google/gson/blob/master/gson/src/main/java/com/google/gson/GsonBuilder.java)可用的，它可以设置各种各样的属性来构建一个Gson实例，例如版本控制等等。

当调用json操作时，Gson实例是不维持任何状态的。因此，你可以自由的重复使用同一个对象来操作多个json序列化和反序列化。


### 通过Maven使用Gson

对应在Maven2/3下使用Gson，你可以使用Gson可用的版本，通过在Maven文件中添加以下依赖：

```
<dependencies>
    <!--  Gson: Java to Json conversion -->
    <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.8.0</version>
      <scope>compile</scope>
    </dependency>
</dependencies>
```
    
就是这样，现在你的maven项目可以使用Gson了。


### 基本数据类型例子

```
// SerializationGson gson = new Gson();
gson.toJson(1);            // ==> 1
gson.toJson("abcd");       // ==> "abcd"
gson.toJson(new Long(10)); // ==> 10int[] values = { 1 };
gson.toJson(values);       // ==> [1]

// Deserializationint one = gson.fromJson("1", int.class);
Integer one = gson.fromJson("1", Integer.class);
Long one = gson.fromJson("1", Long.class);
Boolean false = gson.fromJson("false", Boolean.class);
String str = gson.fromJson("\"abc\"", String.class);
String[] anotherStr = gson.fromJson("[\"abc\"]", String[].class);
```


### 对象例子

```
class BagOfPrimitives {
  private int value1 = 1;
  private String value2 = "abc";
  private transient int value3 = 3;
  BagOfPrimitives() {
    // no-args constructor
  }
}


// SerializationBagOfPrimitives obj = new BagOfPrimitives();
Gson gson = new Gson();
String json = gson.toJson(obj); 

// ==> json is {"value1":1,"value2":"abc"}
```

注意不能序列化循环引用的对象，这样会造成无限递归。

```
// DeserializationBagOfPrimitives obj2 = gson.fromJson(json, BagOfPrimitives.class);
// ==> obj2 is just like obj
```

#### 对象操作的细节



- 最好（推荐）使用私有字段


- 不需要使用任何注释去表明一个字段包含序列化和反序列化。当前类（和继承父类）的所有的字段都默认包含


-  如果一个字段被标记为 transient，（默认）它是被忽略的，JSON序列化和反序列化时不包含其中


- 这个实现能正确的处理 null


- 当序列化时，输出将会跳过值为null的字段


- 当反序列化时，JSON结果中没有的字段，对象对应的字段将会设置为null


-  如果一个字段是 synthetic 的，那么它将被忽略，JSON序列化和反序列化不包括它


- 在内部类、匿名类和本地类中对应于外部类的字段是忽略的，并且不包括在序列化和反序列化中



### 嵌套类（包括内部类）

Gson 可以很容易的序列化静态嵌套类。

Gson 同样也可以反序列化静态嵌套类。但是，Gson不能自动的反序列化纯的内部类，由于它们的无参构造器需要一个引用包含反序列化时不可用的对象。你可以通过使内部类静态化或者提供一个实例构造器来解决这个问题。下面有一个例子：

```
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

注意：上面的B类（默认）不能使用Gson序列化。

Gson 不能反序列化 {"b":"abc"} 为一个B的实例，由于类B是一个内部类。如果内部类B是静态类那么Gson将能够反序列化这个字符串。另一个解决方案是写一个类B的实例构造器。

```
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
上面的代码是可行的，但是不推荐。


### 数组例子

```
Gson gson = new Gson();
int[] ints = {1, 2, 3, 4, 5};
String[] strings = {"abc", "def", "ghi"};

// Serialization
gson.toJson(ints);     // ==> [1,2,3,4,5]
gson.toJson(strings);  // ==> ["abc", "def", "ghi"]

// Deserializationint[] ints2 = gson.fromJson("[1,2,3,4,5]", int[].class); 
// ==> ints2 will be same as ints
```

我们将支持多维数组和各种各样的混合元素类型。


### 集合例子

```
Gson gson = new Gson();
Collection<Integer> ints = Lists.immutableList(1,2,3,4,5);

// SerializationString json = gson.toJson(ints);  // ==> json is [1,2,3,4,5]

// DeserializationType collectionType = new TypeToken<Collection<Integer>>(){}.getType();
Collection<Integer> ints2 = gson.fromJson(json, collectionType);
// ==> ints2 is same as ints
```

相当可怕的：注意我们怎样定义集合的类型。不幸的是，我们没有办法不使用这个java方法。


#### 集合限制

Gson 能够序列化任意的对象集合，但是不能反序列化它们。因为使用者没有方法指出结果对象的类型，然而，当反序列化时，集合必须有一个明确的、泛型类型。这个很容易明白，当遵循好的java编码实践时很少出现这个问题。


### 序列化和反序列化泛型类型

当你调用 toJson(obj) 时，Gson会调用 obj.getClass() 方法得到类的字段信息从而序列化。类似的，你也可以通过 Myclass.class 方式得到对象信息，在方法 fromJson(json, MyClass.class) 中使用。这种方法对于非泛型类型的对象很适用。但是，如果这个对象是一个泛型类型，那么泛型类型的信息会丢失，因为 Java 类型擦除。这里有一个例子说明了这一点：

```
class Foo<T> {
  T value;
}
Gson gson = new Gson();
Foo<Bar> foo = new Foo<Bar>();
gson.toJson(foo); // May not serialize foo.value correctly

gson.fromJson(json, foo.getClass()); // Fails to deserialize foo.value as Bar
```

以上代码不能解释值为 Bar 类型，因为Gson调用 list.getClass() 方法得到它的类信息，但是这个方法返回一个原始类：Foo.class 。这意味着Gson没有方法知道这是一个类型为 Foo<Bar> 的对象，而不仅仅是普通的Foo。

你可以通过指定当前参数类型为你的泛型类型来解决这个问题， 可以通过使用 TypeToken 类来实现。

```
Type fooType = new TypeToken<Foo<Bar>>() {}.getType();
gson.toJson(foo, fooType);

gson.fromJson(json, fooType);
```

用于得到 fooType 的方式实际上定义了一个匿名的本地内部类，这个类包含 getType() 方法，该方法返回完全参数化类型。


### 序列化和反序列化任意类型对象集合

有时候你会处理包含不同类型的 JSON 数组，例如： ['hello', 5, {name:'GREETINGS', source:'guest'}]。

与其等价的 Collection 包含下面这些：

```
Collection collection = new ArrayList();
collection.add("hello);
collection.add(5);
collection.add(new Event("GREETINGS", "guest"));
```

Event 类的定义在下面：

```
class Event {
  private String name;
  private String source;
  private Event(String name, String source) {
    this.name = name;
    this.source = source;
  }
}
```

你能够使用 Gson 序列化这个集合，不用做其他任何事情，通过 toJson(collection) 就会得到想要的输出。

然而，不能通过 fromJson(json, Collection.class) 方式反序列化这个集合，因为Gson不知道怎样映射这个输入到对应的类型。Gson要求你提供一个集合的泛型版本类型在方法 fromJson() 中。那么，你有三个选择：

1. 使用 Gson的解析器API（低级的流解析器或者DOM解析器 JsonParser）去解析数组元素，然后对每一个数组元素使用 Gson.fromJson() 。这个方式是首选。这里有一个[范例](https://github.com/google/gson/blob/master/extras/src/main/java/com/google/gson/extras/examples/rawcollections/RawCollectionsExample.java)
2.  为 Collection.class 注册一个类型适配器，查看数组的每一个成员，并且映射它们为合适的对象。这种方式的缺点是将会造成Gson反序列化其他的集合时出错。
3.  为 MyCollectionMemberType注册一个类型适配器，并且把 Collection<MyCollectionMemberType> 用于 fromJson() 方法。

只有数组作为一个顶级元素出现，或者你可以改变字段类型使改集合类型为 Collection<MyCollectionMemberType>时，此方法是实用的。


### 内置的序列化器和反序列化器

Gson对于常用的类有内置的序列化器和反序列化器，它的默认表示可能是不适用的。下面列出这些类：



1.  使用字符串像 "https://github.com/google/gson/" 去匹配 java.net.URL
2.  使用字符串像 "/google/gson/" 去匹配 "java.net.URI"


你可以找到这些通用类的源代码，例如 [JodaTime](https://sites.google.com/site/gson/gson-type-adapters-for-common-classes-1)


### 自定义序列化和反序列化

有时候默认的表示不是你想要的。当你处理库类（DateTime, 等）是经常会遇到这种情况。Gson允许你注册你自定义的序列化器和反序列化器。通过定义两个部分来完成：


- Json 序列化： 需要为一个对象定义自定义序列化


- Json 反序列化：需要为一个类型定义自定义序列化


- 实例构造器：如果无参构造器是可用的或者反序列化被注册，那么是不需要的


```
GsonBuilder gson = new GsonBuilder();
gson.registerTypeAdapter(MyType2.class, new MyTypeAdapter());
gson.registerTypeAdapter(MyType.class, new MySerializer());
gson.registerTypeAdapter(MyType.class, new MyDeserializer());
gson.registerTypeAdapter(MyType.class, new MyInstanceCreator());
```

registerTypeAdapter 调用检查类型适配器实现的接口是否超过了这些接口，并且把它们都注册。

#### 写一个序列化器

这里有一个例子，展示了怎样为 JodaTime DateTime 类写一个自定义序列化器。

```
private class DateTimeSerializer implements JsonSerializer<DateTime> {
  public JsonElement serialize(DateTime src, Type typeOfSrc, JsonSerializationContext context) {
    return new JsonPrimitive(src.toString());
  }
}
```

当序列化时，运行的是一个 DateTime 对象，Gson 调用 serialize()。
译注： 先注册自定义的DateTimeSerializer，然后使用生成的gson实例调用 toJSON() 方法， 可以查看Example文件夹下的CustomTest.java 例子。


#### 写一个反序列化器

这里有一个例子，展示了怎样为 JodaTime DateTime 类写一个自定义反序列化器。

```
private class DateTimeDeserializer implements JsonDeserializer<DateTime> {
  public DateTime deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context)
      throws JsonParseException {
    return new DateTime(json.getAsJsonPrimitive().getAsString());
  }
}
```

当反序列化一个格式是 DateTime 对象的JSON字符串时，Gson调用 deserialize。


#### 序列化器和反序列化器的更细节处

通常你想为对于原生类型的所有通用泛型类型注册一个单处理程序。



-  例如，假设你有一个表示/翻译 id 的 Id 类（即一个内部或外部表示）


- Id<T>类型对所有泛型类型有同样的序列化

     本质上是输出id的值


- 反序列化是非常相似的，但是不是一样的

    需要调用 new Id(Class<T>, String)， 它会返回一个 Id<T> 的实例

Gson支持为这个注册一个单处理程序。你也可以为一个特定的泛型类型（表明Id<RequiresSpecialHanding> 需要特定的处理）注册一个特定的处理程序。方法 toJson() 和 fromJson() 包含泛型类型的信息，为了帮助你写一个对于同样的原生类型的通用泛型类型注册一个单处理程序。

### 写一个实例创造器

当反序列化一个对象时，Gson需要创造这个类的默认实例。用于序列化和反序列化的规范的类应该有一个无参的构造函数。


- public和private都可以


通常，当你处理一个没有定义无参构造器的库类时需要实例构造器。

#### 实例构造器例子

```
private class MoneyInstanceCreator implements InstanceCreator<Money> {
  public Money createInstance(Type type) {
    return new Money("1000000", CurrencyCode.USD);
  }
}
```

类型可以是一个通用的泛型类型



- 对于调用构造器，特定泛型类型信息是非常有用的


- 例如，如果Id类存储正在创建Id的类


#### 一个参数化类型的InstanceCreator 

有时候你尝试实例化一个参数化类型。通常，由于实际实例是原生类型，所以这没什么问题。下面是一个例子：

```
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

然而，有时候你需要创建一个基于实际参数化类型的实例。在这中情况下，你可以使用类型参数传递给 createInstance 方法。下面有个例子：

```
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

在上面的例子中，类Id在没有传递参数化类型的实际类型的情况下不能创建实例。我们可以通过使用传递方法参数 type 来解决这个问题。 这个例子中的 type 对象是java参数化的类型，表达式 Id<Foo> 表明实际实例应该绑定为Id<Foo> 类。 由于类Id有一个参数化的类型参数 T，我们使用 getActualTypeArgument() 返回的这个类型的零元素数组，在这种情况下将保存Foo.class。


### 紧凑的 vs 漂亮的格式化输出 JSON 

Gson 默认的 JSON输出是紧凑的JSON格式。这意味着输出的JSON结构中将不会有任何的空格。因此，在输出的JSON中，数组中的字段名和值之间、对象字段和对象之间将不会有空格。同样， "null"字段将被忽略（注意：null值将仍然包含在对象的集合/数组中）。在 Null 对象支持 部分有配置Gson输出所有null值的信息。

如果你想使用漂亮的打印属性，你必须使用GsonBuilder来配置Gson实例。这个 JsonFormatter 通过我们公共的API没有暴露出来，所以客户端不能为JSON输出配置默认打印设置/边距。目前，我们仅仅提供了一个默认的 JsonPrintFormatter , 它默认一行80个字符长度，2个字符缩进和4个字符右边距。

下面是一个例子，展示了怎样使用默认的 JsonPrintFormatter 配置一个Gson实例来代替 JsonCompactFormatter:

```
Gson gson = new GsonBuilder().setPrettyPrinting().create();
String jsonOutput = gson.toJson(someObject);
```

### Null 对象支持

在Gson中对 null 对象字段处理的默认行为是忽略。这个允许为了更紧凑的格式化输出；然而，客户必须为这些字段定义一个默认值为JSON格式转换回Java形式

这里有怎样配置一个Gson实例输出 null ：

```
Gson gson = new GsonBuilder().serializeNulls().create();
```

注意：当用Gson序列化 null 时，它将添加一个 JsonNull 元素到 JsonElement 结构。因此，这个对象能够用于自定义序列化/反序列化。

这里有一个例子：

```
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

输出：

```
{"s":null,"i":5}
null
```

### 版本控制支持

同一个对象的多个版本能够通过使用 @Since注解来维护。这个注解能够被用在类上、字段上和未来版本、方法上。为了利用这个特性，你必须配置你的Gson实例，让它忽略那些比现版本号大的任何字段/对象。如果没有设置Gson实例，那么它将序列化和反序列所有的字段和类，不管什么版本。

```
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
String jsonOutput = gson.toJson(someObject);
System.out.println(jsonOutput);
System.out.println();

gson = new Gson();
jsonOutput = gson.toJson(someObject);
System.out.println(jsonOutput);
```

输出：

```
{"newField":"new","field":"old"}

{"newerField":"newer","newField":"new","field":"old"}
```

### 不包括字段的序列化和反序列化

Gson支持许多机制来排除顶层的类、字段和字段类型。以下是可插拔机制来允许字段和类的排除。如果以下机制不满足你的需要，那么你可以使用自定义的序列化器和反序列化器。

#### java修饰符排除

默认情况下，如果你使用修饰符 transient 标注一个字段，它将被排除。同样，如果一个字段被标注为 static ，那么默认也将被排除。如果你想包括一些 transient 字段，那么按照下面这样做：

```
import java.lang.reflect.Modifier;
Gson gson = new GsonBuilder()
    .excludeFieldsWithModifiers(Modifier.STATIC)
    .create();
```

注意：你可以得到所有 Modifier 常量对于方法 excludeFieldsWithModifiers。例如：

```
Gson gson = new GsonBuilder()
    .excludeFieldsWithModifiers(Modifier.STATIC, Modifier.TRANSIENT, Modifier.VOLATILE)
    .create();
```

#### Gson的 @Expose

这个特性提供了一种方式，你可以标注对象的某些字段来排除它们的序列化和反序列化到JSON。为了使用这个注解，你必须使用 new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create() 来创建Gson。这个Gson实例将排除在类中没有标记 @Expose 注解的字段。

#### 用户自定义排除策略

如果以上的排除字段和类类型的机制不能满足你的需求，那么你可以写你自己的排除策略，并且把它插入Gson中。更多信息可以查看[ExclusionStrategy](http://google.github.io/gson/apidocs/com/google/gson/ExclusionStrategy.html) 文档。

下面的例子展示了如何使用特定的 @Foo 注解来排除字段和排除类的顶级String类型（后者声明的字段类型）。

```
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

输出是：

```
{"longField":1234}
```

#### JSON 字段命名支持

Gson支持一些预定义字段命名策略，使其能够转换成标准的java字段名称（也就是，以小写字母开头的驼峰命名法---sampleFieldNameInJava） 到一个Json字段名称（也就是，sample_field_name_in_java 或者 SampleFieldNameInJava）。关于预定义命名策略可以查看类 [FieldNamingPolicy](http://google.github.io/gson/apidocs/com/google/gson/FieldNamingPolicy.html)。

它也有个基于注释的策略，允许客户在每一个字段上自定义命名。注意，这个基于注释的策略有字段名称检验，如果一个无效的字段名称作为注释值被提供，那么将会抛出运行时异常。

下面是一个例子，展示了如何使用Gson命名策略特性：

```
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

输出：

```
{"custom_naming":"first","SomeOtherField":"second"}
```

如果你需要自定义命名策略（看这个[讨论](http://groups.google.com/group/google-gson/browse_thread/thread/cb441a2d717f6892)）， 你可以使用 [@SerializedName](http://google.github.io/gson/apidocs/com/google/gson/annotations/SerializedName.html) 注释。


#### 共享自定义序列化器和反序列化器之间的状态

有时候你需要共享自定义序列化器/反序列化器之间的状态（看这个[讨论](http://groups.google.com/group/google-gson/browse_thread/thread/2850010691ea09fb)）。你可以使用下面三个策略来完成它：


	1. 存储共享状态到静态字段
	2. 作为内部类声明一个父类型的序列化器/反序列化器，使用这个父类型实例的字段存储共享状态
	3. 使用java的 ThreadLocal


1和2是线程不安全的，3是安全的。

#### 流

此外，Gson的对象模型和数据绑定，你可以使用Gson从一个[流](https://sites.google.com/site/gson/streaming)中读取和写入。你也可以结合流和对象模型访问得到最好的方法。


### Gson设计包含的问题

当设计Gson时我们为面对的issues讨论建立的[Gson设计文档](https://github.com/google/gson/blob/master/GsonDesignDocument.md)。它也包含了Gson和其他能够转换Json的java库的比较。


### 未来对Gson的增强

对于最近提出的增强，或者你有什么新的建议，请参阅部分 [Issues](https://github.com/google/gson/issues) 站点。
