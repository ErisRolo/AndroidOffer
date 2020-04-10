## Java注解

### 一. 元数据

**概念**：元数据是关于数据的数据，在编程语言上下文中，元数据是添加到程序元素如方法、字段、类和包上的额外信息，对数据进行说明描述的数据。 

**作用**：① 编写文档：通过代码里标识的元数据生成文档；

​           ② 代码分析：通过代码里标识的元数据对代码进行分析；

​           ③ 编译检查：通过代码里标识的元数据让编译器能实现基本的编译检查；

​           ④ 跟踪代码中的依赖性：可声明方法是重载、依赖父类的方法

**Java平台元数据**：注解就是java平台的元数据，是 J2SE5.0新增加的功能，该机制允许在Java代码中添加自定义注释，并允许通过反射，以编程方式访问元数据注释。通过提供为程序元素（类、方法等）附加额外数据的标准方法，元数据功能具有简化和改进许多应用程序开发领域的潜在能力，其中包括配置管理、框架实现和代码生成。



### 二. 注解

**概念**：注解(Annotation)是在JDK1.5之后增加的一个新特性，注解的引入意义很大，有很多非常有名的框架，比如Hibernate、Spring等框架中都大量使用注解。注解作为元数据嵌入到程序，可以被解析工具或编译工具解析。 

**作用**：同上述元数据的作用

**注意**：注解能被用来为程序元素（类、方法、成员变量等）设置元数据。 注解不影响程序代码的执行，无论增加、删除注解，代码都始终如一地执行。如果希望让程序中的注解起一定的作用，只有通过解析工具或编译工具对注解中的信息进行解析和处理。 

Java提供了多种内建的注解，主要实现了元数据的编译检查作用，常用的**内建注解**如下

**@Override**：用于告知编译器需要覆写超类的当前方法。如果某个方法带有该注解但并没有覆写超类相应的方法，则编译器会生成一条错误信息。如果父类没有这个要覆写的方法，则编译器也会生成一条错误信息。@Override可适用元素为方法，仅仅保留在java源文件中。

**@Deprecated**：用于告知编译器某一程序元素(比如方法，成员变量)不建议使用了（即过时了）。 @Deprecated可适用于除注解类型声明之外的所有元素，保留时长为运行时。

**@SuppressWarnings** ：用于告知编译器忽略特定的警告信息，例在泛型中使用原生数据类型，编译器会发出警告，当使用该注解后，则不会发出警告。@SuppressWarnings 可适用于除注解类型声明和包名之外的所有元素，仅仅保留在java源文件中。该注解有方法`value()`，可支持多个字符串参数，指定忽略哪种警告。

**@FunctionalInterface**：用于告知编译器检查这个接口，保证该接口是函数式接口，即只能包含一 个抽象方法，否则就会编译出错。@FunctionalInterface可适用于注解类型声明，保留时长为运行时。 

JDK除了在java.lang提供了上述内建注解外，还在java.lang.annotation包下提供了6个Meta Annotation(元注解)，其中有5个元注解都用于修饰其他的注解定义，@Repeatable为Java 8新增的可重复注解。4个**常用的修饰其他注解的元注解**如下

**@Documented**：指定被该元注解修饰的注解类将会被javadoc工具提取成文档，如果定义注解类时使用了@Documented修饰，则所有使用该注解修饰的程序元素的API文档中将会包含该注解说明。该注解实现了元数据的编写文档功能。

**@Inherited**：指定被它修饰的注解将具有继承性——如果某个类使用了@XXX注解（定义该注解时使用了@Inherite修饰）修饰，则其子类将自动被@XXX修饰。

**@Retention**：表示该注解类型的注解保留的时长。当注解类型声明中没有@Retention元注解，则默认保留策略为RetentionPolicy.CLASS。关于保留策略(RetentionPolicy)是枚举类型，共定义3种保留策略，分别是SOURCE、CLASS、RUNTIME。

**@Target**：表示该注解类型的所适用的程序元素类型。当注解类型声明中没有@Target元注解，则默认为可适用所有的程序元素。如果存在指定的@Target元注解，则编译器强制实施相应使用限制。关于程序元素(ElementType)是枚举类型，共定义8种程序元素，分别是ANNOTATION_TYPE、CONSTRUCTOR、FIELD、LOCAL_VARIABLE、METHOD、PACKAGE、PARAMETER、TYPE。



### 三. 自定义注解

创建自定义注解，与创建接口有几分相似，但注解需要以@开头，例如

```java
@Documented
@Target(ElementType.METHOD)
@Inherited
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotataion{
    String name();
    String website() default "hello";
    int revision() default 1; 
}
```

**自定义注解中定义成员变量是以无形参的方法形式来声明的**，即

① 注解方法不带参数，如`name()`、`website()`；

② 注解方法返回值类型为基本类型、String、Enums、Annotation以及这些类型的数组类型；

③ 注解方法可有默认值，如`default "hello"`，默认website="hello"

当然注解中也可以不存在成员变量，在使用解析注解进行操作时，仅以是否包含该注解来进行操作。当注解中有成员变量时，若没有默认值，需要在使用注解时指定成员变量的值。 

```java
public class AnnotationDemo {
    @AuthorAnno(name="lvr", website="hello", revision=1)
    public static void main(String[] args) {
        System.out.println("I am main method"); 
    }
    
    @SuppressWarnings({ "unchecked", "deprecation" })
    @AuthorAnno(name="lvr", website="hello", revision=2)
    public void demo(){
        System.out.println("I am demo method");
    } 
}
```

由于该注解的保留策略为RetentionPolicy.RUNTIME，故可在运行期通过反射机制来使用，否则无法通过反射机制来获取。这时候注解实现的就是元数据的代码分析作用。



### 四. 注解解析

通过反射技术解析自定义注解，反射类位于包java.lang.reflect，其中有一个接口AnnotatedElement，该接口主要有如下几个实现类：Class，Constructor，Field，Method，Package。除此之外，该接口定义了注释相关的几 个核心方法，如下：

**getAnnotation(Class annotationClass)**：当存在该元素的指定类型注解，则返回相应注释，否则返回null

**getAnnotations()**：返回此元素上存在的所有注解

**getDeclaredAnnotations()**：返回直接存在于此元素上的所有注解

**isAnnotationPresent(Class<? extends Annotation> annotationClass)**：当存在该元素的指定类型注解，则返回true，否则返回false

因此，当获取了某个类的Class对象，然后获取其Field,Method等对象，通过上述4个方法提取其中的注解，然后获得注解的详细信息，示例如下

```java
public class AnnotationParser {
    public static void main(String[] args) throws 
        SecurityException,ClassNotFoundException {
        String clazz = "com.lvr.annotation.AnnotationDemo";
        Method[] demoMethod = AnnotationParser.class
                                              .getClassLoader()
                                              .loadClass(clazz)
                                              .getMethods();
        for (Method method : demoMethod) {
            if(method.isAnnotationPresent(MyAnnotataion.class)){
                MyAnnotataion annotationInfo = method.getAnnotation(MyAnnotataion.class); 
                System.out.println("method: "+ method);
                System.out.println("name= "+ annotationInfo.nam e() + " ,
                                   website= "+ annotationInfo.website( ) + " ,
                                   revision= "+annotationInfo.revisio n()); 
            }
        }
    }
}
```