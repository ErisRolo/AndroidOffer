## Java transient关键字

### 一. 作用

Java的序列化模式为开发者提供了很多便利，一个类只要实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。在实际开发过程中，常常会遇到这样的问题，一个类的有些属性需要序列化，而其他属性不需要被序列化，此时不需要被序列化的变量就可以加上transient关键字，使这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。也就是说，只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。 示例如下

```java
public class TransientTest {
    public static void main(String[] args) {
        User user = new User();
        user.setUsername("Alexia");
        user.setPasswd("123456");
        System.out.println("read before Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.err.println("password: " + user.getPasswd());
        try {ObjectOutputStream os = new ObjectOutputStream(
            						 new FileOutputStream("C:/user.txt"));
             os.writeObject(user); //将User对象写进文件
             os.flush();
             os.close(); 
            } catch (FileNotFoundException e) {
            e.printStackTrace(); 
        } catch (IOException e) {
            e.printStackTrace(); 
        }try {
            ObjectInputStream is = new ObjectInputStream(
               					   new FileInputStream("C:/user.txt"));
            user = (User)is.readObject(); // 从流中读取User的数据
            is.close();
            System.out.println("\nread after Serializable: ");
            System.out.println("username: " + user.getUsername() );
            System.err.println("password: " + user.getPasswd()); 
        } catch (FileNotFoundException e) { 
            e.printStackTrace(); 
        } catch (IOException e) {
            e.printStackTrace(); 
        } catch (ClassNotFoundException e) { 
            e.printStackTrace(); 
        }
    }
}

class User implements Serializable {
    private static final long serialVersionUID = 829418001491210 3005L;
    private String username;
    private transient String passwd;
    public String getUsername() {
        return username; 
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getPasswd() {
        return passwd; 
    }
    public void setPasswd(String passwd) {
        this.passwd = passwd; 
    } 
}

// 输出为：
// read before Serializable:
// username: Alexia
// password: 123456
// read after Serializable:
// username: Alexia
// password: null
```



### 二. 注意

**1.** 一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问

**2.** transient关键字只能修饰变量，而不能修饰方法和类。本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口

**3.** 被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化（反序列化后类中static型变量的值为当前JVM中对应static变量的值，这个值是JVM中的不是反序列化得出的）



### 三. 细节

讨论被transient关键字修饰的变量是否真的不能被序列化，示例如下

```java
public class ExternalizableTest implements Externalizable {
    
    private transient String content = "是的，我将会被序列化，不管我是否被transient关键字修饰";
    
    @Override
    public void writeExternal(ObjectOutput out) throws IOExcepti on {
        out.writeObject(content); 
    }
    
    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        content = (String) in.readObject(); 
    }
    
    public static void main(String[] args) throws Exception {
        ExternalizableTest et = new ExternalizableTest();
        ObjectOutput out = new ObjectOutputStream(
                           new FileOutputStream(new File("test"))); 
        out.writeObject(et);
        ObjectInput in = new ObjectInputStream(new FileInputStre am(new File( "test"))); 
        et = (ExternalizableTest) in.readObject();
        System.out.println(et.content);
        out.close();
        in.close(); 
    } 
}
```

如上，content变量会被序列化。

在Java中，对象的序列化可以通过实现两种接口来实现，若实现的是Serializable接口，则所有的序列化将会自动进行，若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关。因此第二个例子输出的是变量content初始化的内容，而不是null。 