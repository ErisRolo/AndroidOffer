## Java异常

### 一. 简介

Java异常是Java提供的一种识别及响应错误的一致性机制。 

Java异常机制可以使程序中异常处理代码和正常业务代码分离，保证程序代码更加优雅，并提高程序健壮性。在有效使用异常的情况下，异常能清晰的回答what, where, why这3个问题：异常类型回答了“什么”被抛出，异常堆栈跟踪回答了“在哪“抛出，异常信息回答了“为什么“会抛出。 

Java异常机制用到的几个关键字如下

**try**：用于监听。将要被监听的代码（可能抛出异常的代码）放在try语句块之内，当try语句块内发生异常时，异常就被抛出。

**catch**：用于捕获异常。catch用来捕获try语句块中发生的异常。 

**finally**：finally语句块总是会被执行。它主要用于回收在try块里打开的物力资源（如数据库连接、网络连接和磁盘文件）。只有finally块执行完成之后，才会回来执行try或者catch块中的return或者throw语句，如果finally中使用了return或者throw等终止方法的语句，则就不会跳回执行，直接停止。

**throw**：用于抛出异常。

**throws**：用在方法签名中，用于声明该方法可能抛出的异常。 



### 二. 异常框架

#### 1. Throwable

Throwable是Java语言中所有错误或异常的超类。 

Throwable包含两个子类: **Error**和**Exception**。它们通常用于指示发生了异常情况。

Throwable包含了其线程创建时线程执行堆栈的快照，它提供了`printStackTrace()`等接口用于获取堆栈跟踪数据等信息。

#### 2. Exception

Exception及其子类是Throwable的一种形式，它指出了合理的应用程序想要捕获的条件。 

#### 3. RuntimeException

RuntimeException是那些可能在Java虚拟机正常运行期间抛出的异常的超类。

编译器不会检查RuntimeException异常。例如，除数为零时，抛出ArithmeticException异常。RuntimeException是ArithmeticException的超类。当代码发生除数为零的情况时，倘若既没有通过throws声明抛出ArithmeticException异常，也没有通过try...catch...处理该异常，也能通过编译，也就是所说的编译器不会检查RuntimeException异常。

如果代码会产生RuntimeException异常，则需要通过修改代码进行避免。例如，若会发生除数为零的情况，则需要通过代码避免该情况的发生。

#### 4. Error

和Exception一样，Error也是Throwable的子类，它用于指示合理的应用程序不应该试图捕获的严重问题，大多数这样的错误都是异常条件。 

和RuntimeException一样，编译器也不会检查Error。