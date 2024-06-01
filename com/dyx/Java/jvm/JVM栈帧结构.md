## 栈帧概述
Java的源码文件经过编译器编译后会生成字节码文件，然后由JVM的类加载器进行加载，再交给执行引擎执行。在执行过程中，JVM会划出一块内存空间来存储程序执行期间所需要用到的数据，这块空间被称为运行时数据区。
栈帧（Stack Frame）是运行时数据区中用于支持虚拟机进行方法调用和方法执行的数据结构。每一个方法从调用开始到执行完成，都对应着一个栈帧在虚拟机栈/本地方法栈里从入栈到出栈的过程。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717206729510-827686e1-2a0f-4ade-ba34-2a4614028592.png#averageHue=%23ededf1&clientId=u2f043bff-a976-4&from=paste&id=u74eafa5f&originHeight=1174&originWidth=1478&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u177d6616-b739-490c-9d10-e5664075da6&title=)
### 栈帧组成
每一个栈帧都包括以下几部分：

1. **局部变量表**：用于存储方法参数和局部变量。
2. **操作数栈**：用于操作数临时存储和操作。
3. **动态链接**：用于支持方法调用过程中的符号引用和实际引用之间的转换。
4. **方法返回地址**：用于保存方法返回时的地址，以便恢复到正确的执行位置。
5. **附加信息**：包括一些实现相关的信息，辅助虚拟机进行调试和垃圾回收等操作。

在编译程序代码时，栈帧中需要多大的局部变量表、多深的操作数栈都已经完全确定，并且写入到方法表的Code属性之中。
一个线程中的方法调用链可能会很长，很多方法都处于执行状态。在当前线程中，位于栈顶的栈帧被称为当前栈帧（Current Stack Frame），与这个栈帧相关联的方法称为当前方法。执行引擎运行的所有字节码指令都是对当前栈帧进行操作。在概念模型上，栈帧的结构如下图所示：
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717206762357-f0d2ea6a-0b3e-4a32-82dc-5bcb3800e586.png?x-oss-process=image%2Fformat%2Cwebp#averageHue=%23f8f7f6&from=url&id=GwhHW&originHeight=716&originWidth=1144&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=)
## 局部变量表
### 什么是局部变量表？
局部变量表（Local Variables Table）用来保存方法中的局部变量，以及方法参数。当 Java 源代码文件被编译成 class 文件的时候，局部变量表的最大容量就已经确定了。
```java
public class LocalVariablesTable {
    private void write(int age) {
        String name = "Hello JVM";
    }
}
```
在**write**方法中有一个参数**age**和一个局部变量**name**。使用Intellij IDEA的jclasslib插件查看编译后的字节码文件**LocalVariablesTable.class**，可以看到**write**方法的Code属性中，Maximum local variables（局部变量表的最大容量）的值为3。
#### 为什么是3？
按理说，局部变量表的最大容量应该为2才对，因为只有一个**age**和一个**name**。但实际上是3，这是因为当一个非静态方法被调用时，第0个变量是调用这个方法的对象引用，即**this**。所以调用**write(18)**实际上是调用**write(this, 18)**。
查看Code属性中的LocalVariableTable，可以看到详细的信息：

- 第0个是**this**，类型为**LocalVariablesTable**对象；
- 第1个是方法参数**age**，类型为整型**int**；
- 第2个是方法内部的局部变量**name**，类型为字符串**String**。
- ![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717207322391-e580f272-3623-4676-acc9-039971e80012.png#averageHue=%23282d35&clientId=u2f043bff-a976-4&from=paste&height=150&id=u0a5c6721&originHeight=188&originWidth=802&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=15587&status=done&style=none&taskId=u3a9f522d-d891-481d-b9a5-3cc81121cbd&title=&width=641.6)
### 局部变量表的大小
局部变量表的大小并不是方法中所有局部变量的数量之和，它与变量的类型和作用域有关。当一个局部变量的作用域结束后，它占用的局部变量表中的位置就会被接下来的局部变量所取代。
来看下面这段代码：
```java
public static void method() {
    // 1
    if (true) {
        // 2
        String name = "沉默王二";
    }
    // 3
    if (true) {
        // 4
        int age = 18;
    }
    // 5
}
```
**method**方法的局部变量表大小为1，因为是静态方法，所以不需要**this**作为局部变量表的第一个元素：

- 在2时，局部变量有一个**name**，局部变量表的大小为1；
- 在3时，**name**变量的作用域结束；
- 在4时，局部变量有一个**age**，局部变量表的大小为1；
- 在5时，**age**变量的作用域结束。
#### 局部变量表的槽（slot）
局部变量表的容量以槽（slot）为最小单位，一个槽可以容纳一个32位的数据类型（例如**int**）。像**float**和**double**这种明确占用64位的数据类型会占用两个紧挨着的槽。
来看下面的代码：
```java
public void slot() {
    double d = 1.0;
    int i = 1;
}
```
用jclasslib查看**slot**方法的`**Maximum local variables**`的值为4。
#### 为什么是4？
包括**this**在内应该是3个变量，但实际上是4个。通过`**LocalVariableTable**`就知道了，变量 i 的下标为 3，这是因为变量**d**占用了两个槽。

- ![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717207907125-5e6a5b02-7b18-478f-8a09-28ba271ef672.png#averageHue=%23282d35&clientId=u2f043bff-a976-4&from=paste&height=155&id=u4b8b22eb&originHeight=194&originWidth=821&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=14399&status=done&style=none&taskId=uaffec750-8a63-48b7-939c-cf30239886d&title=&width=656.8)
- ![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717207923322-7110b960-0151-4369-b1aa-a3fd3c64e862.png#averageHue=%235d6362&clientId=u2f043bff-a976-4&from=paste&id=u2b6778b6&originHeight=360&originWidth=1164&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u2a8f24e9-6037-42f4-b98a-e6426f06658&title=)
## 操作数栈
### 什么是操作数栈？
同局部变量表一样，操作数栈（Operand Stack）的最大深度也在编译的时候就确定了，被写入到了 Code 属性的 `**maximum stack size**` 中。当一个方法刚开始执行的时候，操作数栈是空的，在方法执行过程中，会有各种字节码指令往操作数栈中写入和取出数据，也就是入栈和出栈操作。
### 代码示例
先来看下这段代码：
```java
public class OperandStack {
    public void test() {
        add(1, 2);
    }

    private int add(int a, int b) {
        return a + b;
    }
}

```
**OperandStack**类共有两个方法：**test**方法调用了**add**方法，并传递了两个参数。用jclasslib查看字节码文件，可以看到**test**方法的**maximum stack size**值为3。
#### 为什么是3？
这是因为调用成员方法时会将**this**和所有参数压入栈中，调用完毕后**this**和参数都会一一出栈。通过jclasslib的**Bytecode**面板可以查看到对应的字节码指令：

- **aload_0：**将局部变量表下标为0的引用类型变量（即**this**）加载到操作数栈中
- **iconst_1：**将常量1 加载到操作数栈中
- **iconst_2：**将常量2 加载到操作数栈中
- **invokespecial：**调用对象的成员方法
- **pop：**将栈顶的值出栈
- **return：**返回结果

再来看下**add**方法的字节码指令：

- **iload_1：**将局部变量表下标为 1 的 **int** 类型变量加载到操作数栈中（下标为 0 的是 **this**）
- **iload_2：**将局部变量表下标为 2 的 **int** 类型变量加载到操作数栈中
- **iadd：**用于执行**int**类型的加法运算
- **ireturn：**返回值为 int类型的字节码指令
- ![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717208827531-95e65ac7-bb4c-479d-91a5-cb836195e4fb.png#averageHue=%23ebebea&clientId=u2f043bff-a976-4&from=paste&height=343&id=uc2665ded&originHeight=522&originWidth=1240&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u871e6a2a-5ab9-4697-ab4c-2eece8fc034&title=&width=815)
#### 操作数栈的匹配
操作数栈中的数据类型必须与字节码指令匹配。例如，**iadd**指令只能用于整型数据的加法运算。执行时，栈顶的两个数据必须是**int**类型，不能出现一个**long**型和一个**double**型的数据进行**iadd**命令相加的情况。
## 动态链接
### 什么是动态链接呢？
每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用。这种引用的存在是为了支持方法调用过程中的动态链接（Dynamic Linking）。

- 方法区就是 JVM 的一个运行时内存区域，属于逻辑定义，不同版本的 JDK 都有不同的实现，但主要的作用就是用于存储已被虚拟机家住在的类信息、常量、静态变量，以及即时编译器编译之后的代码
- 运行时常量池（Runtime Constant Pool）是方法区的一部分，用于存放编译期生成的各种字面量和符号引用（在类加载之后进入运行时常量池）

![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717209260226-21e4f67c-163c-433b-8335-0d2c41887425.png#averageHue=%23f4f1e1&clientId=u2f043bff-a976-4&from=paste&id=u969b7093&originHeight=1526&originWidth=1410&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u69366fe5-38d4-47b1-b874-f8a34b59319&title=)
### 代码示例
```java
public class DynamicLinking {
    static abstract class Human {
       protected abstract void sayHello();
    }
    
    static class Man extends Human {
        @Override
        protected void sayHello() {
            System.out.println("男人哭吧哭吧不是罪");
        }
    }
    
    static class Woman extends Human {
        @Override
        protected void sayHello() {
            System.out.println("山下的女人是老虎");
        }
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        man.sayHello();
        woman.sayHello();
        man = new Woman();
        man.sayHello();
    }
}
```
在这段代码中，Man 类和 Woman 类继承了 Human 类，并重写了`sayHello()`方法：

- 运行结果也很好理解：
- man 的引用类型为 Human，但是指向的是 Man 对象
- woman 的引用类型也是 Human，但是指向的是 Woman对象
- 之后，man 又指向了新的 Woman 对象
```latex
男人哭吧哭吧不是罪
山下的女人是老虎
山下的女人是老虎
```
#### 字节码分析
```shell
 0: new           #2                  // class com/github/dyx/stack/DynamicLinking$Man
 3: dup
 4: invokespecial #3                  // Method com/github/dyx/stack/DynamicLinking$Man."<init>":()V
 7: astore_1
 8: new           #4                  // class com/github/dyx/stack/DynamicLinking$Woman
11: dup
12: invokespecial #5                  // Method com/github/dyx/stack/DynamicLinking$Woman."<init>":()V
15: astore_2
16: aload_1
17: invokevirtual #6                  // Method com/github/dyx/stack/DynamicLinking$Human.sayHello:()V
20: aload_2
21: invokevirtual #6                  // Method com/github/dyx/stack/DynamicLinking$Human.sayHello:()V
24: new           #4                  // class com/github/dyx/stack/DynamicLinking$Woman
27: dup
28: invokespecial #5                  // Method com/github/dyx/stack/DynamicLinking$Woman."<init>":()V
31: astore_1
32: aload_1
33: invokevirtual #6                  // Method com/github/dyx/stack/DynamicLinking$Human.sayHello:()V
36: return
```

- 第 1 行：new 指令创建了一个 Man 对象，并将对象的内存地址压入栈中
- 第 2 行：dup 指令将栈顶的值复制一份并压入栈顶，因为接下来的操作会消耗掉一个当前类的引用
- 第 3 行：incokespecial 指令用于调用构造函数进行初始化
- 第 4 行：astore_1，Java 虚拟机从栈顶弹出 Man 对象的引用，然后将其存入下标为 1 局部变量 man 中。
- 第 5、6、7、8 行的指令和第 1、2、3、4 行类似，不同的是 Woman 对象。
- 第 9 行：aload_1 指令将第局部变量 man 压入操作数栈中。
- 第 10 行：invokevirtual 指令调用对象的成员方法 sayHello()，注意此时的对象类型为 `**com/github/dyx/stack/DynamicLinking$Human**`
- 第 11 行：aload_2 指令将第局部变量 woman 压入操作数栈中。
- 第 12 行同第 10 行。

虽然字节码指令中对**man**和**woman**调用**sayHello()**的方法是相同的，但最终执行的方法不同，这是因为**invokevirtual**指令实现了多态。
### 动态链接的实现
根据《Java虚拟机规范》，**invokevirtual**指令在运行时的解析过程如下：

1. 找到操作数栈顶的元素所指向的对象的实际类型，记作C。
2. 如果在类型C中找到与常量池中的描述符匹配的方法，进行访问权限校验，若通过则返回方法的直接引用，查找结束；否则返回**java.lang.IllegalAccessError**异常。
3. 如果未找到，则按照继承关系从下往上依次对C的各个父类进行搜索和验证。
4. 如果始终未找到合适的方法，则抛出**java.lang.AbstractMethodError**异常。

也就是说，**invokevirtual**指令在第一步就确定了运行时的实际类型，所以会根据方法接受者的实际类型来选择方法版本。这种在运行期根据实际类型确定方法执行版本的过程就是动态链接。
## 方法返回地址
### 方法的退出方式
当一个方法开始执行后，只有两种方式可以退出这个方法：
#### 正常退出：

   - 可能会有返回值传递给上层的方法调用者。
   - 返回值的类型由方法的返回指令决定。
      - **ireturn**：返回**int**类型。
      - **lreturn**：返回**long**类型。
      - **freturn**：返回**float**类型。
      - **dreturn**：返回**double**类型。
      - **areturn**：返回引用类型。
      - **return**：用于**void**方法。
#### 异常退出：

   - 方法在执行过程中遇到异常且未得到妥善处理。
   - 不会返回任何值给上层调用者。
### 方法退出后的操作
无论是正常退出还是异常退出，在方法退出后，都必须返回到方法最初被调用的位置，程序才能继续执行。以下是方法退出后的操作：
#### 正常退出：

   - PC计数器的值会作为返回地址。
   - 栈帧中保存这个计数器的值。
   - 恢复上层方法的局部变量表和操作数栈。
   - 如果有返回值，将其压入调用者栈帧的操作数栈中。
   - 调整PC计数器的值，找到下一条要执行的指令。
#### 异常退出：

   - 不会保存返回地址，因为没有返回值。
   - 异常机制处理过程会影响PC计数器的值和执行流程。
### PC计数器

- **PC计数器**：JVM运行时数据区的一部分，用于跟踪当前线程执行字节码的位置。
### 方法退出的过程
方法退出的过程实际上等同于将当前栈帧出栈，恢复上层调用方法的执行状态：

1. **恢复上层方法的局部变量表**：在方法调用时，当前方法的局部变量表可能会覆盖调用者的方法的部分或全部局部变量表空间。
2. **恢复操作数栈**：恢复调用者方法的操作数栈，以确保上层方法可以继续执行。
3. **处理返回值**：如果当前方法有返回值，将返回值压入调用者栈帧的操作数栈中。
4. **调整PC计数器**：设置PC计数器的值，以便继续执行调用者方法的下一条指令。
> 转载：[二哥的Java进阶之路·深入理解栈帧结构](https://www.javabetter.cn/jvm/stack-frame.html)

