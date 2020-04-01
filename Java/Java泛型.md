## Java泛型

### 一. 简介

语法糖：也称糖衣语法，是由英国计算机学家Peter.J.Landin发明的一个术语，**指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用**。Java中最常用的语法糖主要有泛型、变长参数、条件编译、自动拆装箱、内部类等。虚拟机并不支持这些语法，它们在编译阶段就被还原回了简单的基础语法结构，这个过程成为解语法糖。

引入泛型的目的：Java泛型就是一种语法糖，是对Java语言类型系统的一种扩展，有点类似于C++的模板，可以把类型参数看作是使用参数化类型时指定的类型的一个占位符，**通过泛型使得在编译阶段完成一些类型转换的工作，避免在运行时强制类型转换而出现`ClassCastException`，即类型转换异常**。

泛型的好处：① 类型安全；

​                       ② 消除了代码中许多的强制类型转换，增强了代码的可读性；

​                       ③ 为较大的优化带来了可能



### 二. 使用

#### 1. 泛型类和泛型接口

泛型的实质：允许在定义接口、类时声明类型形参，类型形参在整个接口、类体内可当成类型使用，几乎所有可使用普通类型的地方都可以使用这种类型形参。

以泛型类的使用为例，泛型接口的使用与泛型类基本相同，示例如下

```java
//定义一个容器类，存放键值对key-value，键值对的类型不确定，可以使用泛型来定义，分别指定为K和V
public class Container<K, V> {
    private K key;
    private V value;
    public Container(K k, V v) {
        key = k;
        value = v;
    }
    public K getkey() {
        return key; 
    }
    public V getValue() {
        return value; 
    }
    public void setKey() {
        this.key = key; 
    }
    public void setValue() {
        this.value = value; 
    } 
}

//使用时，只需指定K,V的具体类型即可，从而创建出逻辑上不同的Container实例，用来存放不同的数据类型
public static void main(String[] args) {
    Container<String,String> c1=new Container<String ,String>( "name","hello");
    Container<String,Integer> c2=new Container<String,Integer>( "age",22);
    Container<Double,Double> c3=new Container<Double,Double>(1.1,1.3);
    System.out.println(c1.getKey() + " : " + c1.getValue());
    System.out.println(c2.getKey() + " : " + c2.getValue());
    System.out.println(c3.getKey() + " : " + c3.getValue());
}
```

在JDK1.7增加了泛型的“菱形”语法：Java允许在构造器后不需要带完整的泛型信息，只需给出一对尖括号<>即可，Java可以推断尖括号里应该是什么泛型信息，如下

```java
Container<String,String> c1=new Container<>("name","hello");
Container<String,Integer> c2=new Container<>("age",22);
```

当创建了带泛型声明的接口、父类之后，可以为该接口创建实现类，或者从该父类派生子类，需要注意的是，使用这些接口、父类派生子类时不能再包含类型形参，需要传入具体的类型，如下

```java
//错误的方式
public class A extends Container<K, V>{}

//正确的方式
public class A extends Container<Integer, String>{}
//也可以不指定具体的类型，此时系统会把K,V形参当成Object类型处理
public class A extends Container{}
```

#### 2. 泛型方法

在泛型类、泛型接口的方法中，可以把泛型中声明的类型形参当成普通类型使用，但在另外一些情况下，在类、接口中没有使用泛型时，定义方法时想定义类型形参，就会使用泛型方法，如下

```java
public class Main{
    public static <T> void out(T t){
        System.out.println(t);
    }
    public static void main(String[] args){
        out("hansheng");
        out(123);
    } 
}
```

所谓**泛型方法，就是在声明方法时定义一个或多个类型形参**，用法格式如下

```java
修饰符<T, S> 返回值类型 方法名（形参列表）｛
	方法体 
｝
```

注意：方法声明中定义的形参只能在该方法里使用，而接口、类声明中定义的类型形参则可以在整个接口、类中使用，如下

```java
class Demo{
    public <T> T fun(T t){
        //可以接收任意类型的数据
        return t; //直接把参数返回 
    } 
};
public class GenericsDemo{
    public static void main(String args[]){
        Demo d = new Demo(); //实例化Demo对象
        String str = d.fun("汤姆"); //传递字符串，编译器自动判断类型形参T所代表的的实际类型
        int i = d.fun(30); //传递数字，自动装箱
        System.out.println(str); //输出内容
        System.out.println(i); //输出内容 
    } 
};
```

#### 3.泛型构造器

正如泛型方法允许在方法签名中声明类型形参一样，**Java也允许在构造器签名中声明类型形参，这样就产生了所谓的泛型构造器**。和使用普通泛型方法一样，一种是显示指定泛型参数，另一种是隐式推断，如下

```java
public class Person {
    public <T> Person(T t) {
        System.out.println(t); 
    } 
}
public static void main(String[] args){
    //显式，以显示指定的类型参数为准，如果传入的参数类型和指定不符，会编译错误
    new<String> Person("hello"); 
    //隐式
    new Person(22); 
}
```

需要注意类是一个泛型类同时构造器也是泛型构造器时的使用。



### 三. 类型通配符

**类型通配符是一个问号(?)，是匹配任意类型的类型实参**，用法如下

```java
//将一个问号作为类型实参元素传给List集合，List<?>表示元素类型未知的List
public void test(List<?> c){
    for(int i =0;i<c.size();i++){
        System.out.println(c.get(i)); 
    } 
}
```

然后可以传入任何类型的List来调用test()方法，程序依然可以访问集合c中的元素，其类型是Object

但是不能把元素加入到其中，因为程序无法确定c集合中元素的类型，所以不能向其添加对象，如下

```java
List<?> c = new ArrayList<String>();
//编译器报错
c.add(new Object());
```

可以使用**带限通配符**，来确定集合元素中的类型。**使用通配符的目的是来限制泛型的类型参数的类型，使其满足某种条件，固定为某些类**，主要分为两种：上限通配符和下限通配符。

上限通配符使用关键字extends来实现，实例化时，指定类型实参只能是extends后类型的子类或其本身，例如

```java
//它表示集合中的所有元素都是Shape类型或者其子类 
List<? extends Shape>

//Circle是其子类 
List<? extends Shape> list = new ArrayList<Circle>();
```

下限通配符使用关键字super来实现，实例化时，指定类型实参只能是super后类型的父类或其本身，例如

```java
//Shape是其父类 
List<? super Circle> list = new ArrayList<Shape>();
```



### 四. 类型擦除

不管为泛型的类型形参传入哪一种类型实参，对于Java来说，它们依然被当成同一类处理，在内存中也只占用一块内存空间。Java泛型只是作用于代码编译阶段，在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦除，也就是说，成功编译过后的class文件中是不包含任何泛型信息的。泛型信息不会进入到运行时阶段。举例如下

```java
Class c1=new ArrayList<Integer>().getClass();
Class c2=new ArrayList<String>().getClass();
System.out.println(c1==c2); //输出结果为true
```

在静态方法、静态初始化块或者静态变量的声明和初始化中不允许使用类型形参。 

由于系统中并不会真正生成泛型类，所以**instanceof**运算符后不能使用泛型类。 