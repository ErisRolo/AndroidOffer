## Java 8

### 一. 语言新特性

#### 1. Lambda表达式和函数式接口

**Lambda表达式（也称为闭包）允许将函数当成参数传递给某个方法，或者把代码本身当作数据处理**，示例如下

```java
//最简单的Lambda表达式可由逗号分隔的参数列表、->符号和语句块组成
Arrays.asList("a", "b", "d").forEach(e -> System.out.println(e));
//上面这个代码中的参数e的类型是由编译器推理得出的，也可以显式指定该参数的类型
Arrays.asList("a", "b", "d").forEach((String e) -> System.out.println(e));
//如果Lambda表达式需要更复杂的语句块，则可以使用花括号将该语句块括起来，类似于Java中的函数体
Arrays.asList("a", "b", "d").forEach(e -> {
    System.out.print(e); 
    System.out.print(e);
});


//Lambda表达式可以引用类成员和局部变量（会将这些变量隐式转换成final的），例如下列两段代码的效果完全相同
String separator = ",";
Arrays.asList("a", "b", "d").forEach((String e) -> System.out.print(e + separator));
final String separator = ",";
Arrays.asList("a", "b", "d").forEach((String e) -> System.out.print(e + separator));


//Lambda表达式有返回值，返回值的类型也由编译器推理得出
//如果Lambda表达式中的语句块只有一行，则可以不用使用return语句，下列两段代码效果相同
Arrays.asList("a", "b", "d").sort((e1, e2) -> e1.compareTo(e2));
Arrays.asList("a", "b", "d").sort((e1, e2) -> {
    int result = e1.compareTo(e2);
    return result; 
});
```

Lambda的设计者们为了让现有的功能与Lambda表达式良好兼容，考虑了很多方 法，于是产生了函数接口这个概念。**函数接口指的是只有一个函数的接口，这样的接口可以隐式转换为Lambda表达式。 **java.lang.Runnable和 java.util.concurrent.Callable是函数式接口的最佳例子。在实践中，函数式接口非常脆弱，只要某个开发者在该接口中添加一个函数，则该接口就不再是函数式接口 进而导致编译失败。为了克服这种代码层面的脆弱性，并显式说明某个接口是函数接口，Java 8提供了一个特殊的注解@FunctionalInterface（Java库中的所有相关接口都已经带有这个注解了），示例如下

```java
@FunctionalInterface
public interface Functional {
    void method(); 
}
```

需要注意的是，**默认方法和静态方法不会破坏函数式接口的定义**，如下

```java
@FunctionalInterface
public interface FunctionalDefaultMethods {
	void method();
    default void defaultMethod() {
    } 
}
```

#### 2. 接口的默认方法和静态方法

Java 8使用两个新概念扩展了接口的含义：默认方法和静态方法。

默认方法使得开发者可以在不破坏二进制兼容性的前提下，往现存接口中添加新的方法，即不强制那些实现了该接口的类也同时实现这个新加的方法。默认方法和抽象方法之间的区别在于抽象方法需要实现，而默认方法不需要。接口提供的默认方法会被接口的实现类继承或者覆写，示例如下

```java
private interface Defaulable {
    // Interfaces now allow default methods, 
    // the implementer may or may not implement (override) them.
    default String notRequired() { 
        return "Default implementation";
    } 
}

private static class DefaultableImpl implements Defaulable {
}

private static class OverridableImpl implements Defaulable {
    @Override public String notRequired() {
        return "Overridden implementation"; 
    } 
}
```

Java 8带来的另一个有趣的特性是在接口中可以定义静态方法，示例如下

```java
private interface DefaulableFactory {
    // Interfaces now allow static methods
    static Defaulable create(Supplier<Defaulable> supplier) {
        return supplier.get(); 
    } 
}
```

下面的代码片段整合了默认方法和静态方法的使用场景

```java
public static void main( String[] args ) {
    Defaulable defaulable = DefaulableFactory.create(DefaultableImpl::new); 
    System.out.println(defaulable.notRequired());
    
    defaulable = DefaulableFactory.create(OverridableImpl::new);
    System.out.println(defaulable.notRequired()); 
}
```

由于JVM上的默认方法的实现在字节码层面提供了支持，因此效率非常高。默认方法允许在不打破现有继承体系的基础上改进接口。该特性在官方库中的应用是：给java.util.Collection接口添加新方法，如`stream()`、`parallelStream()`、`forEach()`和`removeIf()`等等。

#### 3. 方法引用

方法引用使得开发者可以**直接引用现存的方法、Java类的构造方法或者实例对象**。 

方法引用和Lambda表达式配合使用，使得java类的构造方法看起来紧凑而简洁，没有很多复杂的模板代码。

用下面这个例子来区分四种类型的方法引用

```java
public static class Car {
    public static Car create(final Supplier<Car> supplier) {
        return supplier.get(); 
    }
    
    public static void collide(final Car car) { 
        System.out.println("Collided " + car.toString()); 
    }
    
    public void follow(final Car another) {
        System.out.println("Following the " + another.toString( ));
    }
    
    public void repair() {
        System.out.println("Repaired " + this.toString()); 
    } 
}
```

第一种方法引用的类型是**构造器引用**，语法是**Class::new**，**这个构造器没有参数**， 如下

```java
final Car car = Car.create(Car::new);
final List<Car> cars = Arrays.asList(car);
```

第二种方法引用的类型是**静态方法引用**，语法是**Class::static_method**，**这个方法接受一个Car类型的参数**，如下

```java
cars.forEach(Car::collide);
```

第三种方法引用的类型是**某个类的成员方法的引用**，语法是**Class::method**，**这个方法没有定义入参**，如下

```java
cars.forEach(Car::repair);
```

第四种方法引用的类型是**某个实例对象的成员方法的引用**，语法是**instance::method**，这**个方法接受一个Car类型的参数**，如下

```java
final Car police = Car.create(Car::new);
cars.forEach(police::follow);
```

#### 4. 重复注解

Java 8引入了重复注解的概念，允许在同一个地方多次使用同一个注解，在Java 8中使用**@Repeatable**注解定义重复注解，示例如下

```java
public class RepeatingAnnotations {
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Filters {
        Filter[] value(); 
    }
    
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Repeatable(Filters.class)
    public @interface Filter { 
        String value(); 
    };
    
    @Filter("filter1")
    @Filter("filter2") 
    public interface Filterable {
    }
    
    public static void main(String[] args) { 
        for(Filter filter: Filterable.class.getAnnotationsByType(Filter.class)) { 
            System.out.println(filter.value());
        } 
    } 
}
```

这里的Filter类使用@Repeatable(Filters.class)注解修饰， 而Filters是存放Filter注解的容器，编译器尽量对开发者屏蔽这些细节。这样，Filterable接口可以用两个Filter注解注释（这里并没有提到任何关于Filters的信息）。 

另外，反射API提供了一个新的方法`getAnnotationsByType()`，可以返回某个类型的重复注解，例如`Filterable.class.getAnnoation(Filters.class)`将返回两个Filter实例`filter1`和`filter2`。

#### 5. 更好的类型推断

Java 8编译器在类型推断方面有很大的提升，在很多场景下编译器可以推导出某个参数的数据类型，从而使得代码更为简洁。示例如下

```java
public class Value<T> {
    public static<T> T defaultValue() {
        return null; 
    }
    public T getOrDefault(T value, T defaultValue) {
        return (value != null) ? value : defaultValue;
    } 
}

public class TypeInference {
    public static void main(String[] args) { 
        final Value<String> value = new Value<>(); //不需要显式指出
        value.getOrDefault( "22", Value.defaultValue());
    } 
}
```

#### 6. 拓宽注解的应用场景

Java 8拓宽了注解的应用场景，注解几乎可以使用在任何元素上：局部变量、接口类型、超类和接口实现类，甚至可以用在函数的异常定义上。



### 二. 编译器新特性

为了在运行时获得Java程序中方法的**参数名称**，老一辈的Java程序员必须使用不同方法，例如Paranamerliberary。Java 8终于将这个特性规范化，在语言层面（使用反射API和`Parameter.getName()`方法）和字节码层面（使用新的javac编译器以及-parameters参数）提供支持。



### 三. 官方库的新特性

Java 8增加了很多新的工具类，并扩展了现存的工具类，以支持现代的并发编程、函数式编程等。

Java应用中最常见的bug就是空值异常。在Java 8之前，Google Guava引入了Optionals类来解决NullPointerException，从而避免源码被各种null检查污染，以便开发者写出更加整洁的代码。Java 8也将**Optional**加入了官方库。

新增的**Stream API(java.util.stream)**将生成环境的函数式编程引入了Java库中。这是目前为止最大的一次对Java库的完善，以便开发者能够写出更加有效、更加简洁和紧凑的代码。

Java 8引入了新的**Date-Time API(JSR 310)**来改进时间、日期的处理。时间和日期的管理一直是最令Java开发者痛苦的问题。**java.util.Date**和后来的 **java.util.Calendar**一直没有解决这个问题。

Java 8提供了新的**Nashorn JavaScript**引擎，允许在JVM上开发和运行JS应用。Nashorn JavaScript引擎是javax.script.ScriptEngine的另一个实现版本，这类Script引擎遵循相同的规则，允许Java和JavaScript交互使用。

**对Base64编码的支持**已经被加入到Java 8官方库中，这样不需要使用第三方库就可以进行Base64编码。

Java8版本新增了很多新的方法，用于**支持并行数组处理**。最重要的方法 是`parallelSort()`，可以显著加快多核机器上的数组排序。

对于并发性，基于新增的lambda表达式和stream特性，Java 8为**java.util.concurrent.ConcurrentHashMap**类添加了新的方法来支持聚焦操作； 另外，也为**java.util.concurrentForkJoinPool**类添加了新的方法来支持通用线程池操作。Java 8还添加了新的**java.util.concurrent.locks.StampedLock**类，用于支持基于容量的锁，该锁有三个模型用于支持读写操作（可以把这个锁当做是**java.util.concurrent.locks.ReadWriteLock**的替代者）。