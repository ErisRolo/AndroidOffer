## Java深拷贝和浅拷贝

### 一. 简介

对象拷贝(Object Copy)就是将一个对象的属性拷贝到另一个有着相同类类型的对象中去。在程序中拷贝对象是很常见的，主要是为了在新的上下文环境中复用对象的部分或全部数据。**Java中有三种类型的对象拷贝：浅拷贝(Shallow Copy)、深拷贝(Deep Copy)、延迟拷贝(Lazy Copy)。** 



### 二. 浅拷贝

浅拷贝是**按位拷贝**对象，它会创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。**如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。**示例如下

```java
public class Subject {
    private String name;
    
    public Subject(String s) {
        name = s; 
    }
    
    public String getName() {
        return name; 
    }
    
    public void setName(String s) {
        name = s; 
    }   
}

public class Student implements Cloneable {
    
    // 对象引用
    private Subject subj;
    private String name;
    
    public Student(String s, String sub) {
        name = s;
        subj = new Subject(sub);
    }
    
    public Subject getSubj() {
        return subj; 
    }
    
    public String getName() {
        return name; 
    }
    
    public void setName(String s) {
        name = s; 
    }
    
    /**
    * 重写clone()方法
    * @return 
    */
    public Object clone() {
        //浅拷贝
        try {
            // 直接调用父类的clone()方法
            return super.clone(); 
        } catch (CloneNotSupportedException e) {
            return null; 
        } 
    } 
}

public class CopyTest {
    public static void main(String[] args) {
        // 原始对象
        Student stud = new Student("John", "Algebra");
        System.out.println("Original Object: " + stud.getName() + " - " 
                           + stud.getSubj().getName());
        // 拷贝对象
        Student clonedStud = (Student) stud.clone();
        System.out.println("Cloned Object: " + clonedStud.getNam e() + " - " 
                           + clonedStud.getSubj().getName());
        
        // 原始对象和拷贝对象是否一样：
        System.out.println("Is Original Object the same with Clo ned Object: " 
                           + (stud == clonedStud));
        
        // 原始对象和拷贝对象的name属性是否一样
        System.out.println("Is Original Object's field name the same with Cloned Object: " + (stud.getName() == clonedStud.getName()));
        
        // 原始对象和拷贝对象的subj属性是否一样
        System.out.println("Is Original Object's field subj the same with Cloned Object: " + (stud.getSubj() == clonedStud.getSubj()));
        
        stud.setName("Dan");
        stud.getSubj().setName("Physics");
        System.out.println("Original Object after it is updated: " 
                           + stud.getName() + " - " + stud.getSubj().getName());
        System.out.println("Cloned Object after updating original object: " 
                           + clonedStud.getName() + " - " 
                           + clonedStud.getSubj().getName()); 
    } 
}
```



### 三. 深拷贝

深拷贝会拷贝所有的属性,**并拷贝属性指向的动态分配的内存**。当对象和它所引用的对象一起拷贝时即发生深拷贝。深拷贝相比于浅拷贝速度较慢并且花销较大。 示例如下

```java
public class Student implements Cloneable {
    
    // 对象引用
    private Subject subj;
    private String name;
    
    public Student(String s, String sub) {
        name = s;
        subj = new Subject(sub); 
    }
    
    public Subject getSubj() {
        return subj; 
    }
    
    public String getName() {
        return name; 
    }
    
    public void setName(String s) {
        name = s; 
    }
    
    /**
    * 重写clone()方法 
    *
    * @return 
    */
    public Object clone() {
        // 深拷贝，创建拷贝类的一个新对象，这样就和原始对象相互独立
        Student s = new Student(name, subj.getName());
        return s; 
    } 
}
```

很容易发现clone()方法中的一点变化，因为它是深拷贝，所以需要创建拷贝类的一个对象。因为在Student类中有对象引用，所以需要在Student类中实现Cloneable接口并且重写clone方法。 

也可以**通过序列化来实现深拷贝**。序列化是干什么的？它将整个对象图写入到一个持久化存储文件中并且当需要的时候把它读取回来，这意味着当需要把它读取回来时需要整个对象的一个拷贝，这就是当深拷贝一个对象时真正需要的东西。请注意，当通过序列化进行深拷贝时，必须确保对象图中所有类都是可序列化的。示例如下

```java
public class ColoredCircle implements Serializable {
    
    private int x;
    private int y;
    
    public ColoredCircle(int x, int y) {
        this.x = x;
        this.y = y; 
    }
    
    public int getX() {
        return x; 
    }
    
    public void setX(int x) {
        this.x = x; 
    }
    
    public int getY() {
        return y; 
    }
    
    public void setY(int y) {
        this.y = y; 
    }
    
    @Override
    public String toString() {
        return "x=" + x + ", y=" + y; 
    } 
}

public class DeepCopy {
    public static void main(String[] args) throws IOException {
        ObjectOutputStream oos = null;
        ObjectInputStream ois = null;
        try {
            // 创建原始的可序列化对象
            ColoredCircle c1 = new ColoredCircle(100, 100);
            System.out.println("Original = " + c1);
            ColoredCircle c2 = null;
            // 通过序列化实现深拷贝
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            oos = new ObjectOutputStream(bos);
            // 序列化以及传递这个对象
            oos.writeObject(c1);
            oos.flush();
            ByteArrayInputStream bin = new ByteArrayInputStream(bos.toByteArray());
            ois = new ObjectInputStream(bin);
            // 返回新的对象
            c2 = (ColoredCircle) ois.readObject();
            // 校验内容是否相同
            System.out.println("Copied = " + c2);
            // 改变原始对象的内容
            c1.setX(200);
            c1.setY(200);
            // 查看每一个现在的内容
            System.out.println("Original = " + c1);
            System.out.println("Copied = " + c2); 
        } catch (Exception e) {
            System.out.println("Exception in main = " + e);
        } finally {
            oos.close();
            ois.close(); 
        } 
    }
}
```

上述代码可总结为如下几个步骤：

① 确保对象图中的所有类都是可序列化的

② 创建输入输出流

③ 使用这个输入输出流来创建对象输入和对象输出流

④ 将想要拷贝的对象传递给对象输出流

⑤ 从对象输入流中读取新的对象并且转换回所发送的对象的类

除此之外，序列化这种方式存在一定问题：

① 无法序列化transient变量，使用这种方法将无法拷贝transient变量

② 性能问题，比通过实现Clonable接口这种方式实现深拷贝几乎多花100倍的时间



### 四. 延迟拷贝

延迟拷贝是浅拷贝和深拷贝的一个组合，实际上很少会使用。 当最开始拷贝一个对象时，会使用速度较快的浅拷贝，还会使用一个计数器来记录有多少对象共享这个数据。当程序想要修改原始的对象时，它会决定数据是否被共享（通过检查计数器）并根据需要进行深拷贝。 

延迟拷贝从外面看起来就是深拷贝，但是只要有可能它就会利用浅拷贝的速度。当原始对象中的引用不经常改变的时候可以使用延迟拷贝。由于存在计数器，效率下降很高，但只是常量级的开销。而且, 在某些情况下, 循环引用会导致一些问题。



### 五. 选择

如果对象的属性全是基本类型的，那么可以使用浅拷贝，但是如果对象有引用属性，那就要基于具体的需求来选择浅拷贝还是深拷贝。如果对象引用任何时候都不会被改变，那么没必要使用深拷贝，只需要使用浅拷贝就行了。如果对象引用经常改变，那么就要使用深拷贝。没有一成不变的规则，一切都取决于具体需求。