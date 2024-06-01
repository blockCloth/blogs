Java 源代码文件经过编译器编译后会生成字节码文件，经过加载器加载完毕后会交给执行引擎执行。在执行的过程中，JVM 会划出来一块空间来存储程序执行期间需要用到的数据，这块空间一般被称为运行时数据区，见下图：
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717224099239-0ba81d36-ae1a-487b-970a-482b2a4ea9e7.png#averageHue=%23e5ebe4&clientId=ua3dde723-1517-4&from=paste&id=ue7043689&originHeight=1056&originWidth=944&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u23b7ce64-4cc6-42d1-af95-3a90b42c3af&title=)
### 运行时数据区的组成
根据Java虚拟机规范，运行时数据区可以分为以下几个部分：

1. **程序计数器（Program Counter Register）**
2. **Java虚拟机栈（Java Virtual Machine Stacks）**
3. **本地方法栈（Native Method Stack）**
4. **堆（Heap）**
5. **方法区（Method Area）**
6. ![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717224117521-667ee42d-3f07-4725-9833-3f9f7e0ac5d0.png#averageHue=%23f7e6c8&clientId=ua3dde723-1517-4&from=paste&id=uef7f7523&originHeight=622&originWidth=1126&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=udd0609b8-ef5d-438a-9a0d-fd6e46904e0&title=)
> JDK 8 开始，永久代被彻底移除，取而代之的是元空间。元空间不再是 JVM 内存的一部分，而是通过本地内存（Native Memory）来实现的。也就是说，JDK 8 开始，方法区的实现就是元空间

## 程序计数器
### 程序计数器概述
程序计数器（Program Counter Register）所占的内存空间非常小，可以看作是当前线程所执行的字节码指令的行号指示器。字节码解释器会在工作时改变这个计数器的值，以选择下一条需要执行的字节码指令。程序计数器对于实现分支、循环、跳转、异常处理、线程恢复等功能至关重要。
在JVM中，多线程通过线程轮流切换来获得CPU执行时间。在任意时刻，一个CPU的内核只会执行一条线程中的指令。为了保证线程切换后能恢复到正确的执行位置，每个线程都需要一个独立的程序计数器，互不干扰，从而维护程序的正常执行顺序。因此，程序计数器是线程私有的。
**根据《Java虚拟机规范》：**

- 如果线程执行的是非本地方法，程序计数器中保存的是当前需要执行的字节码指令地址。
- 如果线程执行的是本地方法，程序计数器的值为undefined。这是因为本地方法大多通过C/C++实现，并未编译成需要执行的字节码指令。
### 示例代码与字节码指令
通过代码和字节码指令来理解程序计数器的作用：
```java
public static int add(int a, int b) {
    return a + b;
}
```
对应的字节码指令大致如下：
```shell
0: iload_0      // 从局部变量表中加载变量 a 到操作数栈
1: iload_1      // 从局部变量表中加载变量 b 到操作数栈
2: iadd         // 两数相加
3: ireturn      // 返回结果
```
#### 程序计数器的更新过程
逐步分析程序计数器在执行这些指令时的更新过程：

1. **初始状态**：
   - 方法开始执行时，PC计数器设置为0，指向第一条指令**0: iload_0**。
2. **执行第一条指令**：
   - 执行**iload_0**指令，将局部变量表中索引为0的整数（方法的第一个参数a）加载到操作数栈顶。
   - 执行完成后，PC计数器更新为1，指向下一条指令**1: iload_1**。
3. **执行第二条指令**：
   - 执行**iload_1**指令，将局部变量表中索引为1的整数（方法的第二个参数b）加载到操作数栈顶。
   - 执行完成后，PC计数器更新为2，指向下一条指令**2: iadd**。
4. **执行第三条指令**：
   - 执行**iadd**指令，弹出操作数栈顶的两个整数（a和b），将它们相加，然后将结果压入操作数栈顶。
   - 执行完成后，PC计数器更新为3，指向下一条指令**3: ireturn**。
5. **执行最后一条指令**：
   - 执行**ireturn**指令，弹出操作数栈顶的整数（a + b的结果），并将这个值作为方法的返回值。
   - 方法执行完成，控制权返回到方法调用者。
## Java 虚拟机栈
Java虚拟机栈（JVM Stack）是JVM中的一个重要内存区域，每个线程在运行时都会创建一个独立的JVM栈。JVM栈由多个栈帧（Stack Frame）组成，每个栈帧对应一个被调用的方法。当线程执行一个方法时，会创建一个对应的栈帧，并将栈帧压入栈中；当方法执行完毕后，栈帧从栈中弹出。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717225432642-b182c59e-063f-4bd2-82b6-adbdb95c6571.png#averageHue=%23f7f7f7&clientId=ua3dde723-1517-4&from=paste&id=ued8f717a&originHeight=664&originWidth=1456&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ufb28adea-aa10-42ee-826e-a5838f0fec7&title=)
假设我们有一个简单的**add**方法：
```java
public int add(int a, int b) {
    int result = a + b;
    return result;
}
```
当**add**方法被调用时，JVM会为此次方法调用创建一个新的栈帧。栈帧在方法执行过程中存储局部变量和操作数，并处理动态链接和方法返回。当**add**方法执行完毕后，对应的栈帧会从JVM栈中弹出。
### Java虚拟机栈的特点

1. **线程私有**：每个线程都有自己的JVM栈，线程之间的栈是不共享的。
2. **栈溢出**：如果栈的深度超过了JVM栈所允许的深度，将会抛出**StackOverflowError**异常。
## 本地方法栈
本地方法栈（Native Method Stack）与 Java 虚拟机栈类似，只不过 Java 虚拟机栈为虚拟机执行 Java 方法服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。
## 堆
### 堆的概念
堆是所有线程共享的一块内存区域，在JVM启动时创建，用来存储对象和数组。虽然传统上认为Java中的所有对象都在堆上分配，但随着JIT编译器的发展和逃逸分析技术的成熟，这种情况有所改变。从 JDK 7 开始，Java 虚拟机已经默认开启逃逸分析了，意味着如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存。
栈就是前面提到的 JVM 栈（主要存储局部变量、方法参数、对象引用等），属于线程私有，通常随着方法调用的结束而消失，也就无需进行垃圾收集；堆前面也讲了，属于线程共享的内存区域，几乎所有的对象都在对上分配，生命周期不由单个方法调用所决定，可以在方法调用结束后继续存在，直到不在被任何变量引用，然后被垃圾收集器回收。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717225810901-69ceb07b-7471-4186-b2ef-9253f29253ce.png#averageHue=%23f1d8bf&clientId=ua3dde723-1517-4&from=paste&id=u8d7ac6e7&originHeight=556&originWidth=792&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=udd82124a-dabe-4184-acaa-d0538864294&title=)
#### JIT编译器与逃逸分析

- **JIT编译器**：即即时编译器（Just-In-Time Compiler），它在程序运行时将字节码编译为机器码，以提高执行效率。
- **逃逸分析**（Escape Analysis）：一种编译器优化技术，用于判断对象的作用域和生命周期。如果对象不会逃逸出方法或线程的范围，它可以在栈上分配内存，而不是在堆上。
- ![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717225865415-be8d1703-3fb8-4402-9f5b-18b7839c79e8.png#averageHue=%23dae4d4&clientId=ua3dde723-1517-4&from=paste&id=u34f480c1&originHeight=590&originWidth=1092&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u2ddc5154-6685-4eb9-bb0c-4af7ec1bcc6&title=)

示例代码演示了可能触发栈分配的情况：
```java
public class EscapeAnalysisExample {
    private static class Point {
        private int x;
        private int y;

        Point(int x, int y) {
            this.x = x;
            this.y = y;
        }

        int calculate() {
            return x + y;
        }
    }

    public static void main(String[] args) {
        int total = 0;
        for (int i = 0; i < 1000000; i++) {
            total += createAndCalculate();
        }
        System.out.println(total);
    }

    private static int createAndCalculate() {
        Point p = new Point(1, 2);
        return p.calculate();
    }
}
```
在这个示例中，**Point**对象在**createAndCalculate**方法中创建，并且不会逃逸到该方法之外。JVM的逃逸分析可能会在栈上分配**Point**对象，而不是在堆上。
### 堆的分代
堆是Java垃圾收集器管理的主要区域，被称为GC堆（Garbage Collected Heap）。为了更好地管理内存和提高垃圾回收效率，堆通常分为不同的区域：

- **新生代**：包含Eden空间、From Survivor空间和To Survivor空间。
- **老年代**：存储长时间存活的对象。

这种划分有助于优化垃圾回收的性能。
### 常见的堆内存溢出错误

- **OutOfMemoryError: GC Overhead Limit Exceeded**：当JVM花费太多时间执行垃圾回收且回收效果不佳时，会抛出此错误。
- **java.lang.OutOfMemoryError: Java heap space**：当堆内存不足以存放新创建的对象时，会抛出此错误。

通过以下代码可以模拟堆内存溢出：
```java
public class HeapSpaceErrorGenerator {
    public static void main(String[] args) {
        List<byte[]> bigObjects = new ArrayList<>();
        try {
            while (true) {
                // 创建一个大约10MB的数组
                byte[] bigObject = new byte[10 * 1024 * 1024];
                bigObjects.add(bigObject);
            }
        } catch (OutOfMemoryError e) {
            System.out.println("OutOfMemoryError发生在" + bigObjects.size() + "对象后");
            throw e;
        }
    }
}
```
### 查看堆的默认大小
可以通过以下命令查看JVM堆的默认大小：
```shell
java -XX:+PrintFlagsFinal -version | grep HeapSize	//linux
java -XX:+PrintFlagsFinal -version | findstr HeapSize  //widnows
```
也可以通过以下代码在程序中获取堆的最大内存大小：
```java
System.out.println(Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "MB");
```
## 元空间和方法区
### 基本概念

- **方法区**是Java虚拟机规范中的一个逻辑概念，用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。在不同的JDK版本中，方法区有不同的实现。
- **永久代**（PermGen）是JDK 7及之前版本中，HotSpot JVM对方法区的一种实现。
- **元空间**（Metaspace）是在JDK 8及之后版本中引入的，取代了永久代。
#### 方法区与永久代的关系

- **方法区**：Java虚拟机规范中的概念，规定了功能但没有规定实现。
- **永久代**：HotSpot JVM中对方法区的一种具体实现。
- **元空间**：取代永久代的实现，使用操作系统的本地内存，而不受JVM堆大小的限制。
#### JDK 7与JDK 8的变化

- 在JDK 7中，字符串常量池被移到堆中，运行时常量池仍在方法区（即永久代）中。
- 在JDK 8中，永久代被移除，元空间取而代之。字符串常量池仍在堆中，而运行时常量池移到元空间中。

![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717226503204-59b6b1e3-9196-497c-8c83-8a919d24eac4.png#averageHue=%23f4e1c2&clientId=ua3dde723-1517-4&from=paste&id=u6cc16822&originHeight=1310&originWidth=1082&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u51858545-1626-410b-a50d-5503f2ca9ce&title=)
#### 为什么废弃永久代？
旧版的 Hotspot 虚拟机是没有 JIT 的，而 Oracle 旗下的另外一款虚拟机 JRocket 是有的，那为了将 Java 帝国更好的传下去，Oracle 就想把 JRocket 的 JIT 技术融合到 Hotspot 中。
但 JRockit 虚拟机中并没有永久代的概念，因此新的 HotSpot 索性就不要永久代了，直接占用操作系统的一部分内存好了，并且把这块内存取名叫做元空间。
**元空间的优势：**

- 使用操作系统的本地内存，不受JVM堆大小的限制。
- 减少了**OutOfMemoryError**的发生，提升了内存管理的灵活性。

元空间虽然使用操作系统的内存，但仍然可能出现溢出。当元空间数据增长时，JVM会请求更多内存。如果操作系统无法满足请求，则会导致元空间溢出。
### 运行时常量池

- **常量池**：它是字节码文件的资源仓库，先是一个常量池大小，从 1 到 n-1，0 为保留索引，然后是常量池项的集合，包括类信息、字段信息、方法信息、接口信息、字符串常量等。
- ![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717226743362-b3de54ad-7dfe-44a8-a418-957164957d3e.png#averageHue=%23f4f4f4&clientId=ua3dde723-1517-4&from=paste&id=u40675b94&originHeight=456&originWidth=1424&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u927ea81c-9d51-4d8f-b223-6b271bf2dad&title=)
- **运行时常量池**：在运行时，JVM将字节码文件中的常量池加载到内存中，存放在运行时常量池中（JDK 8及以后位于元空间中）。
- ![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717226799748-df1c532d-987a-4076-8862-cbf142fbb28e.png#averageHue=%23f3f5f2&clientId=ua3dde723-1517-4&from=paste&id=u59d9b7ca&originHeight=489&originWidth=925&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u94ac902b-2bc0-4b4b-a9d5-3fd9dc55759&title=)
### 字符串常量池

- **字符串常量：**它的作用是存放字符串常量，也就是我们在代码中写的字符串。依然在堆中。
- ![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717226867681-c8460069-3166-4510-9642-acc414dadeb3.png#averageHue=%23f8f0e5&clientId=ua3dde723-1517-4&from=paste&id=u9c3832c1&originHeight=890&originWidth=1350&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u60e8bc2d-28a9-4815-8d29-d5556861a85&title=)
### 方法区与堆的共同点

- 都是线程共享的内存区域。
- 

