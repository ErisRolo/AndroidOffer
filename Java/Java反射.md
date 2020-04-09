## Java反射

### 一. 概述

**定义**：Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类中的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取信息以及动态调用对象的方法的功能称为Java的反射机制

**功能**：① 在运行时判断任意一个对象所属的类；

​           ② 在运行时构造任意一个类的对象；

​           ③ 在运行时判断任意一个类所具有的成员变量和方法；

​           ④ 在运行时调用任意一个对象的方法；

​           ⑤ 生成动态代理

**应用场景**：① 逆向代码，例如反编译；

​                   ② 与注解相结合的框架，例如Retrofit；

​                   ③ 单纯的反射机制应用框架，例如EventBus;

​                   ④ 动态生成类框架，例如Gson



### 二. 通过Java反射查看类信息

每个类被加载后，系统就会为该类生成一个对应的Class对象，通过该Class对象就可以访问到JVM中的这个类。

**获得Class对象的三种方式**：① 使用Class类的`forNanme(Strubg className)`静态方法；

​                                               ② 调用某个类的class属性来获取该类对应的Class对象；

​                                               ③ 调用某个对象的`getClass()`方法，代码如下

```java
//第一种方式 通过Class类的静态方法——forName()来实现，参数值必须是某个类的全限定名（必须添加完整包名）
class1 = Class.forName("com.ghx.reflection.Person");
//第二种方式 通过类的class属性
class1 = Person.class;
//第三种方式 通过对象getClass方法
Person person = new Person();
Class<?> class1 = person.getClass();
```

**获取Class对象的成员变量**，代码如下

```java
Field[] allFields = class1.getDeclaredFields(); //获取Class对象的所有属性
Field[] publicFields = class1.getFields(); //获取Class对象的public属性
Field ageField = class1.getDeclaredField("age"); //获取Class指定属性 
Field desField = class1.getField("des"); //获取Class指定的public属性
```

**获取Class对象的方法**，代码如下

```java
Method[] methods = class1.getDeclaredMethods(); //获取Class对象的所有声明方法
Method[] allMethods = class1.getMethods(); //获取Class对象的所有public方法 包括父类的方法 
Method method = class1.getMethod("info", String.class); //返回此Class对象对应类的、带指定形参列表的public方法 
Method declaredMethod = class1.getDeclaredMethod("info", String.class); //返回此Class对象对应类的、带指定形参列表的方法
```

其他方法还有获取Class对象的注解、Type以及各种信息；判断是否为基础类型、集合类、注解类等，略



### 三. 通过Java反射生成并操作对象

**生成类的实例对象的两种方式**：

① 使用Class对象的`newInstance()`方法来创建该Class对象对应类的实例。这种方式要求该Class对象的对应类有默认构造器，而执行`newInstance()`方法时实际上是利用默认构造器来创建该类的实例

② 先使用Class对象获取指定的Constructor对象，再调用Constructor对象的`newInstance()`方法来创建该Class对象对应类的实例，通过这种方式可以选择使用指定的构造器来创建实例，代码如下

```java
//第一种方式 Class对象调用newInstance()方法生成
Object obj = class1.newInstance();
//第二种方式 对象获得对应的Constructor对象，再通过该Constructor对象的newInstance()方法生成
Constructor<?> constructor = class1.getDeclaredConstructor(String.class);//获取指定声明构造函数 
obj = constructor.newInstance("hello");
```

**调用类的方法**

通过Class对象的`getMethods()`方法或`getDeclaredMethod()`方法获得指定方法，返回Method数组或对象；调用Method对象中的`Object invoke(Object obj, Object... args)`方法，第一个参数对应调用该方法的实例对象，第二个参数对应该方法的参数，代码如下

```java
//生成新的对象：用newInstance()方法
Object obj = class1.newInstance();
//首先需要获得与该方法对应的Method对象
Method method = class1.getDeclaredMethod("setAge", int.class);
//调用指定的函数并传递参数
method.invoke(obj, 28);
```

当通过Method的`invoke()`方法来调用对应的方法时，Java会要求程序必须有调用该方法的权限。如果程序确实需要调用某个对象的private方法，则可以先调用Method对象的`setAccessible(boolean flag)`方法，将Method对象的accessible设置为指定的布尔值，值为true指示该Method在使用时应该取消Java的访问权限检查，值为false则指示该Method在使用时要实施Java的访问权限检查

**访问成员变量值**

通过Class对象的`getFields()`方法或者`getField()`方法获得指定属性对象，返回Field数组或对象；然后通过`getXXX(Object obj)`方法和`setXXX(Object obj, XXX val)`方法来获取或设置obj对象的该成员变量的值，XXX对应8种基本类型，如果该成员变量的类型是引用类型则取消get和set后的XXX，代码如下

```java
//生成新的对象：用newInstance()方法
Object obj = class1.newInstance();
//获取age成员变量
Field field = class1.getField("age");
//将obj对象的age的值设置为10
field.setInt(obj, 10);
//获取obj对象的age的值
field.getInt(obj);
```



### 四. 代理模式

**定义**：给某个对象提供一个代理对象，并由代理对象控制对于原对象的访问，即客户不直接操控原对象，而是通过代理对象间接地操控原对象

**代理模式的角色**分为四种，如下

① 主题接口：Subject是委托对象和代理对象共同实现的接口，即代理类的所实现的行为接口。Request() 是委托对象和代理对象共同拥有的方法

② 目标对象：RealSubject是原对象，也就是被代理的对象

③ 代理对象：Proxy是代理对象，用来封装真实主题类的代理类

④ 客户端：使用代理类和主题接口完成一些工作

**代理模式的实现**有两类，分别是

① 静态代理：代理类是在编译时就实现好的，也就是说Java编译完成后代理类是一 个实际的class文件

② 动态代理：代理类是在运行时生成的，也就是说Java编译完之后并没有实际的class文件，而是在运行时动态生成的类字节码，并加载到JVM中

**代理模式的实现思路**：① 代理对象和目标对象均实现同一个行为接口；

​                                      ② 代理类和目标类分别具体实现接口逻辑；

​                                      ③ 在代理类的构造函数中实例化一个目标对象；

​                                      ④ 在代理类中调用目标对象的行为接口；

​                                      ⑤ 客户端想要调用目标对象的行为接口，只能通过代理类来操作

静态代理模式的简单实现如下

```java
public class ProxyDemo {
    public static void main(String args[]){
        RealSubject subject = new RealSubject();
        Proxy p = new Proxy(subject);
        p.request();
    }
}
interface Subject{
    void request(); 
}
class RealSubject implements Subject{
    public void request(){
        System.out.println("request"); 
    } 
}
class Proxy implements Subject{
    private Subject subject;
    public Proxy(Subject subject){
        this.subject = subject; 
    }
    public void request(){
        System.out.println("PreProcess");
        subject.request();
        System.out.println("PostProcess"); 
    } 
}
```

目标对象(RealSubject)以及代理对象(Proxy)都实现了主题接口(Subject)，在代理对象(Proxy)中，通过构造函数传入目标对象(RealSubject)，然后重写主题接口(Subject)的`request()`方法，在该方法中调用目标对(RealSubject)的`request()`方法，并可以添加一些额外的处理工作在目标对象(RealSubject)的`request()`方法的前后。 

**代理模式的好处**：可以在被调用方法前后加上自己的操作，而不需要更改被调用类的源码，大大地降低了模块之间的耦合性，体现了极大的优势。



### 五. Java反射机制与动态代理

**动态代理**：动态代理是指在运行时动态生成代理类，即代理类的字节码将在运行时生成并载入当前代理的 ClassLoader

**动态代理与静态相比具有优势**，如下

① 不需要为目标对象写一个形式上完全一样的封装类，假如主题接口中的方法很多，为每一个接口写一个代理方法也很麻烦。如果接口有变动，则目标对象和代理类都要修改，不利于系统维护； 

② 使用一些动态代理的生成方法甚至可以在运行时制定代理类的执行逻辑，从而大大提升系统的灵活性

**动态代理主要涉及两个类**，都是java.lang.reflect包下的类，**内部主要通过反射来实现**，如下

**java.lang.reflect.Proxy**：生成代理类的主类，提供了用户创建动态代理类和代理对象的静态方法，是所有动态代理类的父类，通过Proxy类生成的代理类都继承了Proxy类

**java.lang.reflect.InvocationHandler**：“调用处理器”，是一个接口，当调用动态代理类中的方法时，将会直接转接到执行自定义的InvocationHandler中的`invoke()`方法，即我们动态生成的代理类需要完成的具体内容需要自己定义一个类，而这个类必须实现InvocationHandler接口，通过重写`invoke()`方法来执行具体内容

**Proxy提供了两个方法来创建动态代理类和动态代理实例**，分别是`static Class<?> getProxyClass()`返回代理类的java.lang.Class对象，`static Object newProxyInstance()`返回代理类实例，参数都是类加载器对象、接口和调用处理器类实例,**对应两种创建动态代理对象的方式**如下

```java
//用getProxyClass()创建动态实例的步骤
//创建一个InvocationHandler对象
InvocationHandler handler = new MyInvocationHandler(.arg s..);
//使用Proxy生成一个动态代理类
Class proxyClass = Proxy.getProxyClass(RealSubject.class.getClassLoader(),
                                       RealSubject.class.getInterfaces(), handler);
//获取proxyClass类中一个带InvocationHandler参数的构造器
Constructor constructor = proxyClass.getConstructor(InvocationHandler.class);
//调用constructor的newInstance方法来创建动态实例
RealSubject real = (RealSubject)constructor.newInstance(handler);

//用newProxyInstance()创建动态实例的步骤
//创建一个InvocationHandler对象
InvocationHandler handler = new MyInvocationHandler(.arg s..);
//使用Proxy直接生成一个动态代理对象
RealSubject real =Proxy.newProxyInstance(RealSubject.class.getClassLoader(),
                                         RealSubject.class.getInterfaces(), handler);
```

**newProxyInstance**这个方法实际上做了两件事：第一，创建了一个新的类【代理类】，这个类实现了**Class[]interfaces**中的所有接口，并通过指定的**ClassLoader**将生成的类的字节码加载到**JVM**中，创建**Class**对象；第二，以传入的**InvocationHandler**作为参数创建一个代理类的实例并返回

Proxy类还有一些静态方法，比如`InvocationHandler getInvocationHandler(Object proxy)`获得代理对象对应的调用处理器对象，`Class getProxyClass(ClassLoader loader, Class[] interfaces)`根据类加载器和实现的接口获得代理类；InvocationHandler接口中有方法`invoke(Object proxy, Method method, Object[] args)`，这个函数在代理对象调用任何一个方法时都会调用，方法不同会导致第二个参数method不同，第一个参数是代理对象，第二个参数是Method对象，第三个参数是指定调用方法的参数

动态代理模式的简单实现如下

```java
public class DynamicProxyDemo {
    public static void main(String[] args) {
        //1.创建目标对象
        RealSubject realSubject = new RealSubject();
        //2.创建调用处理器对象
        ProxyHandler handler = new ProxyHandler(realSubject);
        //3.动态生成代理对象
        Subject proxySubject =
       	(Subject)Proxy.newProxyInstance(RealSubject.class.getClassLoader(),
                                		RealSubject.class.getInterfaces(), handler); 
        //4.通过代理对象调用方法
        proxySubject.request(); 
    } 
}

/**
*主题接口
*/ 
interface Subject{
    void request(); 
}

/**
* 目标对象类
*/ 
class RealSubject implements Subject{
    public void request(){
        System.out.println("====RealSubject Request===="); 
    } 
}

/**
* 代理类的调用处理器 
*/ 
class ProxyHandler implements InvocationHandler{
    private Subject subject;
    public ProxyHandler(Subject subject){
        this.subject = subject; 
    }
    @Override 
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //定义预处理的工作，当然你也可以根据 method 的不同进行不同的预处 理工作 
        System.out.println("====before====");
        //调用RealSubject中的方法
        Object result = method.invoke(subject, args);
        System.out.println("====after====");
        return result; 
    } 
}
```

可以看到，通过newProxyInstance就产生了一个Subject的实例，即代理类的实例，然后就可以通过`Subject.request()`，调用InvocationHandler中的`invoke()`方法，传入方法Method对象，以及调用方法的参数，通过`Method.invoke()`调用RealSubject中的方法的`request()`方法，同时可以在InvocationHandler中的`invoke()`方法加入其他执行逻辑。



### 六. 泛型和Class类

从JDK 1.5 后，Java中引入泛型机制，Class类也增加了泛型功能，从而允许使用 泛型来限制Class类，例如：String.class的类型实际上是Class<String>。如果 Class对应的类暂时未知，则使用Class<?>(?是通配符)。**通过反射中使用泛型，可以避免使用反射生成的对象需要强制类型转换，防止出现类型转换异常。**



### 七. 使用反射来获取泛型信息

通过指定类对应的Class对象，可以获得该类里包含的所有Field，不管该Field是使用private修饰，还是使用public修饰。获得了Field对象后就可以很容易地获得该 Field 的数据类型，即使用如下代码即可获得指定Field的类型

```java
// 获取 Field 对象 f 的类型
Class<?> a = f.getType();
```

但这种方式只对普通类型的Fiel 有效，如果该Fiel 的类型是有泛型限制的类型，如Map<String, Integer>类型，则不能准确地得到该Field的泛型参数。为了获得指定Field的泛型类型，应先使用如下方法来获取指定Field的类型

```java
// 获得 Field 实例的泛型类型 
Type type = f.getGenericType();
```

然后将Type对象强制类型转换为ParameterizedType对象，ParameterizedType代表被参数化的类型，也就是增加了泛型限制的类型。ParameterizedType类提供了如下两个方法

**getRawType()**：返回没有泛型信息的原始类型

**getActualTypeArguments()**：返回泛型参数的类型

通过反射获取泛型类型的完整程序如下

```java
public class GenericTest {
    private Map<String , Integer> score;
    public static void main(String[] args) throws Exception {
        Class<GenericTest> clazz = GenericTest.class;
        Field f = clazz.getDeclaredField("score"); 
        // 直接使用getType()取出Field类型只对普通类型的Field有效
        Class<?> a = f.getType();
        // 下面将看到仅输出java.util.Map 
        System.out.println("score的类型是：" + a);
        // 获得Field实例f的泛型类型
        Type gType = f.getGenericType();
        // 如果gType类型是ParameterizedType对象
        if(gType instanceof ParameterizedType) {
            // 强制类型转换
            ParameterizedType pType = (ParameterizedType)gType;
            // 获取原始类型
            Type rType = pType.getRawType();
            System.out.println("原始类型是：" + rType);
            // 取得泛型类型的泛型参数 
            Type[] tArgs = pType.getActualTypeArguments();
            System.out.println("泛型类型是：");
            for (int i = 0; i < tArgs.length; i++) {
                System.out.println("第" + i + "个泛型类型是：" + tArgs[i]); 
            } 
        } else {
            System.out.println("获取泛型类型出错！"); 
        } 
    } 
}
//输出结果如下
//score 的类型是：interface java.util.Map
//原始类型是：interface java.util.Map
//泛型类型是：
//第 0 个泛型类型是：class java.lang.String
//第 1 个泛型类型是：class java.lang.Integer
```

从上面的运行结果可以看出，直接使用Field的`getType()`方法只能获取普通类型的Field的数据类型，对于增加了泛型参数的类型的Field应该使用`getGenericType()`方法来取得其类型。

Type也是java.lang.reflect包下的一个接口，该接口代表所有类型的公共高级接口，Class是Type接口的实现类。Type包括原始类型、参数化类型、数组类型、 类型变量和基本类型等。