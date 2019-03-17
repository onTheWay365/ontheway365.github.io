---
title: java注解
date: 2019-03-16 15:30:29
tags: [java]
---

### 定义

一种标签，用于解释`Java`代码。

注解在 `Java SE 5.0`开始引入 ，属于 `Java`中的一种类型，同 类`class` 和 接口`interface` 一样 。

<!--more-->

注解（`Annotation`）面向 **编译器 /  APT** 使用 。

`APT（Annotation Processing Tool）` 用于处理带有注解的代码。注解不会自动生效，必须由开发者提供相应的代码来提取并处理 `Annotation` 信息。这些处理提取和处理 `Annotation` 的代码统称为 `APT` 。

### 语法定义

注解的定义要符合以下语法规则：

- 使用@interface定义 
- 自定义注解不能继承其他的注解或者接口
- 属性的类型是受限制的，合法的类型只包含以下的几类： 
  - 基本类型和String  
  - Class,Annotation,Enumeration 
  - 以上类型的数组形式
- 注解的属性声明实际上是方法声明，方法名即为注解的属性名。注解中的方法，不允许使用protect、private修饰符，也无需加上public等修饰符，保持默认即可 。方法不能有参数，不能抛异常，不能是泛型方法
- 单值注解：只有一个属性，且方法名为value()时，则赋值时”value“可以省略

所有的注解接口隐式地拓展自`java.lang.annotation.Annotation`，注解无法被继承。

下面是一个合法的注解声明的例子：

```
public @interface BugReport {

    enum Status {UNCONFIRMED, CONFIRMED, FIXED, NOTABUG}

    boolean showStopper() default false;

    String assignedTo() default "[none]";

    Class<?> testCase() default Void.class;

    Status status() default Status.UNCONFIRMED;

    Reference ref() default @Reference(); // an annotation type
    
    String[] reported();

}
```



### 注解类型

注解类型可分为：元注解、`java`内置注解、自定义注解。

#### 元注解

元注解是一种基本注解，能应用到其他普通注解上，对其他普通注解进行解释说明。

注意：普通注解设置@Target({ElementType.ANNOTATION_TYPE})，也可以应用到普通注解上。

元注解有以下几种：

##### `@Target`

Target 是目标的意思，@Target 指定了注解运用的地方。 

当一个注解被 @Target 注解时，这个注解就被限定了运用的场景。如果不明确指出，该注解可以放在任何地方 。

类比到标签，原本标签是你想张贴到哪个地方就到哪个地方，但是因为 @Target 的存在，它张贴的地方就非常具体了，比如只能张贴到方法上、类上、方法参数上等等。

@Target的取值在`java.lang.annotation.ElementType`中定义 ：

```
public enum ElementType {

    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```

`ElementType.TYPE`:可以给一个类型进行注解，如：类、接口、注解、枚举。

`ElementType.FIELD`:可以给属性进行注解，包括枚举常量。

`ElementType.METHOD`:可以给方法进行注解。

`ElementType.PARAMETER`:  可以给一个方法内的参数进行注解。

`ElementType.CONSTRUCTOR`: 可以给构造方法进行注解。

`ElementType.LOCAL_VARIABLE`: 可以给局部变量进行注解 。

`ElementType.ANNOTATION_TYPE`: 可以给一个注解进行注解，是`ElementType.TYPE`的子集。例如：`@RequestMapping`上的`@Mapping`使用的就是该范围；`@SpringBootApplication`上的`@EnableAutoConfiguration`使用的则是`ElementType.TYPE`。

`ElementType.PACKAGE`: 可以给一个包进行注解。

`ElementType.TYPE_PARAMETER`: `jdk1.8`新增。

`ElementType.TYPE_USE`: `jdk1.8`新增 。

从Java 8开始，对注解进行了增强（Extended Annotations Support） ，注解已经能应用于任何目标。这其中包括new操作符、类型转换、instanceof检查、泛型类型参数，以及implements和throws子句。 ElementType.TYPE_PARAMETER和ElementType.TYPE_USE就是为注解增强而设计的。

ElementType.TYPE_PARAMETER表示该注解能写在类型变量的声明语句中 。

```
@Target(ElementType.TYPE_PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface TypeParameterAnnotation {
    
}

// 如下是该注解的使用例子
public class TypeParameterClass<@TypeParameterAnnotation T> {
    public <@TypeParameterAnnotation T> T foo(T t) {
        return null;
    }    
}
```

ElementType.TYPE_USE是ElementType.TYPE_PARAMETER的超集，表示该注解能写在使用类型的任何语句中（例如声明语句、泛型和强制转换语句中的类型） 。

```
public class TestTypeUse {

    @Target(ElementType.TYPE_USE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface TypeUseAnnotation {

    }

    public static @TypeUseAnnotation class TypeUseClass<@TypeUseAnnotation T> extends @TypeUseAnnotation Object {
        public void foo(@TypeUseAnnotation T t) throws @TypeUseAnnotation Exception {

        }
    }

    // 如下注解的使用都是合法的
    @SuppressWarnings({ "rawtypes", "unused", "resource" })
    public static void main(String[] args) throws Exception {
        TypeUseClass<@TypeUseAnnotation String> typeUseClass = new @TypeUseAnnotation TypeUseClass<>();
        typeUseClass.foo("");
        List<@TypeUseAnnotation Comparable> list1 = new ArrayList<>();
        List<? extends Comparable> list2 = new ArrayList<@TypeUseAnnotation Comparable>();
        @TypeUseAnnotation String text = (@TypeUseAnnotation String)new Object();
        java.util. @TypeUseAnnotation Scanner console = new java.util.@TypeUseAnnotation Scanner(System.in);
        
        //note:@TypeUseAnnotation 的位置应该在包名右边，类型左边，以下使用例子是错误的，除非给该注解的 @Target 参数里面加上 ElementType.LOCAL_VARIABLE。
        //@TypeUseAnnotation java.lang.String text;
    }
}
```

类型注解被用来支持在Java的程序中做强类型检查。配合第三方插件工具Checker Framework，可以在编译的时候检测出runtime error，以提高代码质量。这就是类型注解的作用。

```
    import checkers.nullness.quals.*;
    public class TestDemo{
        void sample() {
            @NonNull Object obj = null;
            obj.toString();
    }
```

以上代码可以编译通过，但是运行时会报 NullPointerException 异常 。

为了能在编译期间就自动检查出这类异常，可以通过类型注解结合 Checker Framework 提前排查出错误异常。



##### @Retention

Retention是保留的意思。@Retention用于说明注解的生命周期。

@Retention的取值在`java.lang.annotation.RetentionPolicy`中定义 ：

```
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

RetentionPolicy.SOURCE：在源文件中保留，编译时将被丢弃。

RetentionPolicy.CLASS：随源文件一起编译到class文件中，但不会被加载到 JVM。

RetentionPolicy.RUNTIME：注解保留到程序运行时，会被加载到 JVM 中，可以通过反射获取到。



##### @Documented

它的作用是能够将注解中的元素包含到 Javadoc 中去 。



##### @Inherited

Inherited是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么这个子类就继承了超类的注解（注意：子类重写父类方法，无法获取到注解）。 

```
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface InheritedAnnotation {
    String value() default "";
}

@InheritedAnnotation("person class")
@Component
public class Person {

    @InheritedAnnotation("age filed")
    public int age;

    @InheritedAnnotation("run method")
    public void run(){}  
}

public class ChinesePerson extends Person {

}
```



##### @Repeatable

Repeatable 自然是可重复的意思。@Repeatable 是 Java 1.8 才加进来的，可增加注解的可读性。

java8之前，实现重复注解的方式如下：

```
@Retention(RetentionPolicy.RUNTIME)
public @interface OldRepeatableAnnotation {
    String value();
}

@Retention(RetentionPolicy.RUNTIME)
public @interface OldRepeatableAnnotations {
    OldRepeatableAnnotation[] value();
}

@OldRepeatableAnnotations({
        @OldRepeatableAnnotation("001"),
        @OldRepeatableAnnotation("002"),
})
public class OldProduct {

}

public class OldRepeatableAnnotationTest {
    @Test
    public void test(){
        Class<OldProduct> clazz = OldProduct.class;
        Assert.assertFalse(clazz.isAnnotationPresent(OldRepeatableAnnotation.class));
        Assert.assertTrue(clazz.isAnnotationPresent(OldRepeatableAnnotations.class));
        Assert.assertEquals(2, clazz.getAnnotation(OldRepeatableAnnotations.class).value().length);
    }

}
```

使用@Repeatable实现的方式如下：

```
@Retention(RetentionPolicy.RUNTIME)
//这里声明重复注解的容器类型
@Repeatable(RepeatableAnnotations.class)
public @interface RepeatableAnnotation {
    String value();
}

@Retention(RetentionPolicy.RUNTIME)
public @interface RepeatableAnnotations {
    RepeatableAnnotation[] value();
}

@RepeatableAnnotation("001")
@RepeatableAnnotation("002")
public class Product {

}

public class RepeatableAnnotationTest {

    @Test
    public void test() {
        //两种方式获取重复注解
        //1、通过重复注解容器类型获取 getAnnotation
        //2、通过重复注解类型获取 getAnnotationsByType
        Class<Product> clazz = Product.class;
        Assert.assertFalse(clazz.isAnnotationPresent(RepeatableAnnotation.class));
        Assert.assertTrue(clazz.isAnnotationPresent(RepeatableAnnotations.class));

        Assert.assertEquals(2, clazz.getAnnotation(RepeatableAnnotations.class).value().length);
        Assert.assertEquals(2, clazz.getAnnotationsByType(RepeatableAnnotation.class).length);

    }
}
```

重复注解可以看做是一种新的语法糖，在编译器层面，其实与原来的实现无异。同时提供了一个新的方法getAnnotationsByType用于获取重复注解。



##### @Native

@Native 作用在域上，用来表示域中的常量可能来自于本地代码。



#### java内置注解

Java有多种内置注解，其中最常用的是 @Override、@Deprecated、@SuppressWarnings、@SafeVarargs、@FunctionalInterface。 

##### @Override

@Override 注解表示覆盖，应用在方法上。 在覆盖父类方法时加上 @Override 注解，这是一个良好的编程习惯，有助于减少错误的发生。 



##### @Deprecated

@Deprecated 注解也应用在方法上，表示当前方法已经废弃。对弃用的方法加上 @Deprecated 注解，并且避免使用弃用的方法，也是编程的良好习惯。 



##### @SuppressWarnings

@SuppressWarnings注解的作用是告知编译器，忽略他们产生的特殊警告。



##### @SafeVarargs

表示参数安全的注解，它的存在会阻止编译器产生 unchecked 这样的警告 。

加上这个注解并不是真正的安全，参数安全还是需要开发者在代码实现中保证。



##### @FunctionalInterface

函数式接口，用于lambada。作用与@Override类似。



### 注解解析

注解的解析需要使用Java反射机制，注意要将Retention设置为RetentionPolicy.RUNTIME，这样才能在程序运行中获取注解的相关信息。

jdk中是通过java.lang.reflect.AnnotatedElement接口实现对注解解析的 。AnnotatedElement代表了jvm中一个正在运行的被注解元素，这个接口允许通过反射的方式读取注解。

```
public interface AnnotatedElement {

    boolean isAnnotationPresent(Class<? extends Annotation> annotationClass);

    <T extends Annotation> T getAnnotation(Class<T> annotationClass);

    Annotation[] getAnnotations();

    <T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass);

    <T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass);

    <T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass);

    Annotation[] getDeclaredAnnotations();
}
```

Class、Method、Field实现了AnnotatedElement接口。

使用步骤如下：

```
<-- 步骤1：判断该类是否应用了某个注解 -->
// 手段：采用 Class.isAnnotationPresent() 
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}

<-- 步骤2：获取 注解对象（Annotation）-->
// 手段1：采用getAnnotation()  ；返回指定类型的注解
public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}
// 手段2：采用getAnnotations() ；返回该元素上的所有注解
public Annotation[] getAnnotations() {}
```

举例：

```
@Retention(RetentionPolicy.RUNTIME)
public @interface Info {
    int id() default 0;
    String name();
}

@Retention(RetentionPolicy.RUNTIME)
public @interface Info2 {
    int id() default 0;
    String name();
}

@Info(id = 1, name = "order class")
public class Order {

    @Info(id = 2, name = "amount field")
    private int amount;

    @Info(id = 3, name = "create method:info")
    @Info2(id = 4, name = "create method:info2")
    public void create(){

    }

    public static void main(String[] args) throws Exception {

        Class<Order> clazz = Order.class;
        //判断是否存在指定的注解
        boolean hasInfoAnnotation = clazz.isAnnotationPresent(Info.class);
        //如果存在注解，则获取类上的注解
        if (hasInfoAnnotation) {
            Info annotation = clazz.getAnnotation(Info.class);
            System.out.println("Order类上的注解");
            System.out.println(annotation.id());
            System.out.println(annotation.name());
        }

        //获取成员变量上的注解
        Field field = clazz.getDeclaredField("amount");
        field.setAccessible(true);
        boolean annotationPresent = field.isAnnotationPresent(Info.class);
        if (annotationPresent) {
            Info annotation = field.getAnnotation(Info.class);
            System.out.println("成员变量上的注解");
            System.out.println(annotation.id());
            System.out.println(annotation.name());
        }

        //获取方法上的注解
        Method method = clazz.getDeclaredMethod("create");
        if (method == null) {
            return;
        }
        // 因为方法上有2个注解，所以采用getAnnotations()获得所有类型的注解
        Annotation[] annotations = method.getAnnotations();
        System.out.println("方法上的注解");
        for (Annotation annotation : annotations) {
            System.out.println(annotation.annotationType().getSimpleName());
        }
    }

}
```



### 参考

[秒懂，Java 注解你可以这样学](https://blog.csdn.net/briblue/article/details/73824058/)

[Java注解解析](http://patchouli-know.com/2016/11/22/java-annotation/)

[Java元注解和注解的解析](https://www.jianshu.com/p/e9329c8a59c2)

