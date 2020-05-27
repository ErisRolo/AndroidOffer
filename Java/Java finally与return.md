## Java finally与return

### 一. finally不一定被执行

异常捕获机制try...catch...finally块中的finally语句不是一定被执行的，比如以下两种情况：

**1. ** try语句没有被执行到，如在try语句之前就返回了，这样finally语句就不会执 行，这也说明了finally语句被执行的必要而非充分条件是相应的try语句一定被执行到

**2.** 在try块中有 `System.exit(0)`这样的语句，`System.exit(0)`是终止Java虚拟机JVM的，连JVM都停止了，所有都结束了，当然finally语句也不会被执行到



### 二. finally与return执行顺序

#### 1. finally语句在return语句执行之后return返回之前执行

```java
public class FinallyTest1 {
    
    public static void main(String[] args) {
        System.out.println(test1()); 
    }
    
    public static int test1() {
        int b = 20;
        try {
            System.out.println("try block");
            return b += 80;
        } catch (Exception e) {
            System.out.println("catch block"); 
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b > 25, b = " + b);
            } 
        }
        return b; 
    } 
}
// 结果为：
// try block
// finally block
// b > 25, b = 100
// 100

public class FinallyTest1 {
    
    public static void main(String[] args) {
        System.out.println(test11()); 
    }
    
    public static String test11() {
        try {
            System.out.println("try block");
            return test12(); 
        } finally { 
            System.out.println("finally block"); 
        } 
    }
    
    public static String test12() {
        System.out.println("return statement");
        return "after return";
    } 
}
// 结果为：
// try block
// return statement
// finally block
// after return
```

#### 2. finally块中的return语句会覆盖try块中的return返回

```java
public class FinallyTest2 {
    
    public static void main(String[] args) {
        System.out.println(test2()); 
    }
    
    public static int test2() {
        int b = 20;
        try {
            System.out.println("try block");
            return b += 80;
        } catch (Exception e) {
            System.out.println("catch block"); 
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b > 25, b = " + b);
            }
            return 200; 
        }
        // return b; 
    } 
}
// 结果为：
// try block
// finally block
// b > 25, b = 100
// 200
```

#### 3. 如果finally语句中没有return语句覆盖返回值，那么原来的返回值可能因为finally里的修改而改变也可能不变

```java
public class FinallyTest3 {
    
    public static void main(String[] args) {
        System.out.println(test3()); 
    }
    
    public static int test3() {
        int b = 20;
        try {
            System.out.println("try block");
            return b += 80; 
        } catch (Exception e) {
            System.out.println("catch block"); 
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b > 25, b = " + b);
            }
            b = 150; //Java中只有传值没有传址，所以这句无效
        }
        return 2000; 
    }
}
// 结果为：
// try block
// finally block
// b > 25, b = 100
// 100

public class FinallyTest3 {
    
    public static void main(String[] args) {
        System.out.println(getMap().get("KEY").toString()); 
    }
    
    public static Map<String, String> getMap() {
        Map<String, String> map = new HashMap<String, String>();
        map.put("KEY", "INIT");
        try {
            map.put("KEY", "TRY");
            return map; 
        } catch (Exception e) {
            map.put("KEY", "CATCH"); 
        } finally {
            map.put("KEY", "FINALLY");
            map = null; 
        }
        return map; 
    } 
}
// 结果为：
// FINALLY
```

#### 4. try块里的return语句在异常的情况下不会被执行， 这样具体返回哪个看情况

```java
public class FinallyTest4 {
    
    public static void main(String[] args) {
        System.out.println(test4()); 
    }
    
    public static int test4() {
        int b = 20;
        try {
            System.out.println("try block");
            b = b / 0;
            return b += 80; //除0异常，不会执行
        } catch (Exception e) {
            b += 15; 
            System.out.println("catch block"); 
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b > 25, b = " + b); 
            }
            b += 50; 
        }
        return b; 
    } 
}
// 结果为：
// try block
// catch block
// finally block
// b > 25, b = 35
// 85
```

#### 5. 当发生异常后，catch中的return执行情况与未发生异常时try中return的执行情况完全一样

```java
public class FinallyTest5 {
    public static void main(String[] args) {
        System.out.println(test5()); 
    }
    
    public static int test5() {
        int b = 20;
        try {
            System.out.println("try block");
            b = b /0; 
            return b += 80; 
        } catch (Exception e) {
            System.out.println("catch block");
            return b += 15; 
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b > 25, b = " + b); 
            }
            b += 50; 
        }
        //return b;
    } 
}
// 结果为：
// try block
// catch block
// finally block
// b > 25, b = 35
// 35
```

总结：finally块的语句在try或catch中的return语句执行之后返回之前执行且finally里的修改语句可能影响也可能不影响try或catch中 return已经确定的返回值，若finally里也有return语句则覆盖try或catch中的return语句直接返回。