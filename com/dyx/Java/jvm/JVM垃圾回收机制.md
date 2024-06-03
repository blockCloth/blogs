## 垃圾回收的概念
垃圾回收（Garbage Collection, GC）是Java虚拟机（JVM）内存管理的一项重要机制，其主要功能是自动回收不再使用的对象所占用的内存空间。通过垃圾回收，程序员不再需要手动释放内存，从而减少了内存泄漏和内存管理相关的错误，提高了程序的健壮性和开发效率。
Java 语言出来之前，大家都在拼命的写 C 或者 C++ 的程序，而此时存在一个很大的矛盾，C++ 等语言创建对象要不断的去开辟空间，不用的时候又需要不断的去释放控件，既要写构造函数，又要写析构函数，很多时候都在重复的 allocated，然后不停的析构。于是，有人就提出，能不能写一段程序实现这块功能，每次创建，释放控件的时候复用这段代码，而无需重复的书写呢？
1960年，基于 MIT 的 Lisp 首先提出了垃圾回收的概念，用于处理C语言等不停的析构操作，而这时 Java 还没有出世，所以实际上 GC 并不是Java的专利，GC 的历史远远大于 Java 的历史。
## Stop The World
**"Stop The World"**是 Java 垃圾收集中的一个重要概念。在垃圾收集过程中，JVM 会暂停所有的用户线程，这种暂停被称为"Stop The World"事件。
这么做的主要原因是为了防止在垃圾收集过程中，用户线程修改了堆中的对象，导致垃圾收集器无法准确地收集垃圾。
值得注意的是，"Stop The World"事件会对 Java 应用的性能产生影响。如果停顿时间过长，就会导致应用的响应时间变长，对于对实时性要求较高的应用，如交易系统、游戏服务器等，这种情况是不能接受的。
因此，在选择和调优垃圾收集器时，需要考虑其停顿时间。Java 中的一些垃圾收集器，如 G1 和 ZGC，都会尽可能地减少了"Stop The World"的时间，通过并发的垃圾收集，提高应用的响应性能。
总的来说，"Stop The World"是 Java 垃圾收集中必须面对的一个挑战，其目标是在保证内存的有效利用和应用的响应性能之间找到一个平衡。
## 垃圾判断算法
既然 JVM 要做垃圾回收，就要搞清楚什么是垃圾，什么不是垃圾，通常会有这么几种算法来确定一个对象是否是垃圾。

- 引用计数算法
- 可达性分析算法
### 引用计数算法
引用计数算法（Reachability Counting）是通过在对象头中分配一个空间来保存该对象被引用的次数（Reference Count）。如果该对象被其它对象引用，则它的引用计数加1，如果删除对该对象的引用，那么它的引用计数就减1，当该对象的引用计数为0时，那么该对象就会被回收。
```java
String s = new String("青");
```
先创建一个字符串，这时候"青"有一个引用，就是 s。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717298038129-13df8e49-4415-4599-b2fc-16ad6b94ca58.png#averageHue=%23f9f4f3&clientId=u7acb33eb-072d-4&from=paste&height=231&id=u394964ab&originHeight=289&originWidth=734&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=4186&status=done&style=none&taskId=u3cdf6681-e239-4ac0-90f7-7c09006d25d&title=&width=587.2)
然后将 s 设置为 null
```java
s = null;
```
这时候 "青" 的引用次数就等于0了，在引用计数算法中，意味着这块内容就需要被回收了。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717298360597-01a2620e-aba7-43a2-9730-8ee578e2f25c.png#averageHue=%23f9f5f4&clientId=u7acb33eb-072d-4&from=paste&height=228&id=ue743efa9&originHeight=285&originWidth=912&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=9567&status=done&style=none&taskId=u58b50203-1c47-4cea-85cd-d6916f769d9&title=&width=729.6)
引用计数算法将垃圾回收分摊到整个应用程序的运行当中，而不是集中在垃圾收集时。因此，采用引用计数的垃圾收集不属于严格意义上的"Stop-The-World"的垃圾收集机制。
引用计数算法看似很美好，但实际上它存在一个很大的问题，那就是无法解决循环依赖的问题。来看下面的代码。
```java
public class ReferenceCountingGC {

    public Object instance;  // 对象属性，用于存储对另一个 ReferenceCountingGC 对象的引用

    public ReferenceCountingGC(String name) {
        // 构造方法
    }

    public static void testGC() {
        // 创建两个 ReferenceCountingGC 对象
        ReferenceCountingGC a = new ReferenceCountingGC("李七夜");
        ReferenceCountingGC b = new ReferenceCountingGC("叶凡");

        // 使 a 和 b 相互引用
        a.instance = b;
        b.instance = a;

        // 将 a 和 b 设置为 null
        a = null;
        b = null;

        // 这个位置是垃圾回收的触发点
    }
}

```
代码中创建了两个 ReferenceCountingGC 对象 a 和 b。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717298627715-36b1eb87-7e33-4cf9-843e-365690da17cc.png#averageHue=%23f8f1f0&clientId=u7acb33eb-072d-4&from=paste&height=390&id=udced5d76&originHeight=488&originWidth=703&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=8098&status=done&style=none&taskId=u578b4d67-9980-4331-aa71-86f77d12c32&title=&width=562.4)
然后使它们相互引用。接着，将这两个对象的引用设置为 null，理论上它们会在接下来被垃圾回收器回收。但由于它们相互引用着对方，导致它们的引用计数永远都不会为 0，通过引用计数算法，也就永远无法通知 GC 收集器回收它们![绘图1.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717336821251-c5428e6a-ffb7-4fd5-af28-cea46121a97c.png#averageHue=%23faf7f6&clientId=u7acb33eb-072d-4&from=paste&height=515&id=uae4c6e29&originHeight=644&originWidth=1454&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=21712&status=done&style=none&taskId=uf10f1e16-e0a3-404d-aaa1-f64de953b69&title=&width=1163.2)
### 可达性分析算法
可达性分析算法（Reachability Analysis）的基本思路是，通过一些被称为引用链（GC Roots）的对象作为起点，从这些节点开始向下搜索，搜索走过的路径被称为（Reference Chain)，当一个对象到 GC Roots 没有任何引用链相连时（即从 GC Roots 节点到该节点不可达），则证明该对象是不可用的。
![GC Roots.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717377529659-8c610081-dfd1-44a2-a330-f7b4cdf7da02.png#averageHue=%23faf7f5&clientId=ua20f41c5-415c-4&from=paste&height=767&id=udb4cd799&originHeight=959&originWidth=1858&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=23533&status=done&style=none&taskId=ua0454744-18c2-41f2-aba6-72668deef0d&title=&width=1486.4)
通过可达性算法，成功解决了引用计数无法解决的问题-“循环依赖”，只要你无法与 GC Root 建立直接或间接的连接，系统就会判定你为可回收对象。
所谓的 GC Roots，就是一组必须活跃的引用，不是对象，它们是程序运行时的起点，是一切引用链的源头。在 Java 中，GC Roots 包括以下几种：

- 虚拟机栈中的引用（方法的参数、局部变量等）
- 本地方法栈中 JNI 的引用
- 类静态变量
- 运行时常量池中的常量（String 或 Class 类型）
#### 虚拟机栈中的引用（方法的参数、局部变量等）
```java
public class StackReference {
    public void greet() {
        Object localVar = new Object(); // 这里的 localVar 是一个局部变量，存在于虚拟机栈中
        System.out.println(localVar.toString());
    }

    public static void main(String[] args) {
        new StackReference().greet();
    }
}
```
在 greet 方法中，localVar 是一个局部变量，存在于虚拟机栈中，可以被认为是 GC Roots。
在 greet 方法执行期间，localVar 引用的对象是活跃的，因为它是从 GC Roots 可达的。
当 greet 方法执行完毕后，localVar 的作用域结束，localVar 引用的 Object 对象不再由任何 GC Roots 引用（假设没有其他引用指向这个对象），因此它将有资格作为垃圾被回收掉
#### 本地方法栈中 JNI 的引用
Java 通过 `**JNI（Java Native Interface）**`提供了一种机制，允许 Java 代码调用本地代码（通常是 C 或 C++ 编写的代码）。
当调用 Java 方法时，虚拟机会创建一个栈帧并压入虚拟机栈，而当它调用本地方法时，虚拟机会通过动态链接直接调用指定的本地方法。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717377794085-a588bd94-a14f-4869-9e1b-cf96668a6d95.png#averageHue=%23fefdfc&clientId=ua20f41c5-415c-4&from=paste&id=u42a5bb21&originHeight=442&originWidth=486&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u2833a626-8578-4bc2-bdf8-c4d634fedff&title=)
pecuyu：动态链接
JNI 引用是在 Java 本地接口（JNI）代码中创建的引用，这些引用可以指向 Java 堆中的对象。
```java
// 假设的JNI方法
public native void nativeMethod();

// 假设在C/C++中实现的本地方法
/*
 * Class:     NativeExample
 * Method:    nativeMethod
 * Signature: ()V
 */
JNIEXPORT void JNICALL Java_NativeExample_nativeMethod(JNIEnv *env, jobject thisObj) {
    jobject localRef = (*env)->NewObject(env, ...); // 在本地方法栈中创建JNI引用
    // localRef 引用的Java对象在本地方法执行期间是活跃的
}
```
在本地（C/C++）代码中，localRef 是对 Java 对象的一个 JNI 引用，它在本地方法执行期间保持 Java 对象活跃，可以被认为是 GC Roots。
一旦 JNI 方法执行完毕，除非这个引用是全局的（Global Reference），否则它指向的对象将会被作为垃圾回收掉（假设没有其他地方再引用这个对象）
#### 类静态变量
```java
public class StaticFieldReference {
    private static Object staticVar = new Object(); // 类静态变量

    public static void main(String[] args) {
        System.out.println(staticVar.toString());
    }
}
```
**StaticFieldReference** 类中的 **staticVar** 引用了一个 Object 对象，这个引用存储在元空间，可以被认为是 GC Roots。
只要 StaticFieldReference 类未被卸载，staticVar 引用的对象都不会被垃圾回收。如果 StaticFieldReference 类被卸载（这通常发生在其类加载器被垃圾回收时），那么 staticVar 引用的对象也将有资格被垃圾回收（如果没有其他引用指向这个对象）
#### 运行时常量池中的常量
```java
public class ConstantPoolReference {
    public static final String CONSTANT_STRING = "Hello, World"; // 常量，存在于运行时常量池中
    public static final Class<?> CONSTANT_CLASS = Object.class; // 类类型常量

    public static void main(String[] args) {
        System.out.println(CONSTANT_STRING);
        System.out.println(CONSTANT_CLASS.getName());
    }
}
```
在 **ConstantPoolReference** 中，**CONSTANT_STRING** 和 **CONSTANT_CLASS** 作为常量存储在运行时常量池。它们可以用来作为 GC Roots。
这些常量引用的对象（字符串"Hello, World"和 Object.class 类对象）在常量池中，只要包含这些常量的 ConstantPoolReference 类未被卸载，这些对象就不会被垃圾回收。
## 垃圾收集方法
在确定了哪些垃圾可以被回收后，垃圾收集器要做的事情就是进行垃圾回收，但是这里面涉及到一个问题是：**如何高效地进行垃圾回收**。由于 JVM 规范并没有对如何实现垃圾收集器做出明确的规定，因此各个厂商的虚拟机可以采用不同的方式来实现垃圾收集器，这里我们讨论几种常见的垃圾收集算法。
### 标记清除算法
标记清除算法（Mark-Sweep）是最基础的一种垃圾回收算法，它分为 2 部分，先把内存区域中的这些对象进行标记，哪些属于可回收的标记出来（用前面提到的可达性分析法），然后把这些垃圾拎出来清理掉。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717378111366-1ec2d37f-5899-4a44-a122-279c622030ca.png#averageHue=%23f8f6f5&clientId=u94a46ed1-3676-4&from=paste&id=u0346edb8&originHeight=704&originWidth=1678&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u92fbb4e8-01e5-4452-b70d-39108382911&title=)
就像上图一样，清理掉的垃圾就变成可使用的空闲空间，等待被再次使用。逻辑清晰，并且也很好操作，但它存在一个很大的问题，那就是内存碎片。碎片太多可能会导致当程序运行过程中需要分配较大对象时，因无法找到足够的连续内存而不得不提前触发新一轮的垃圾收集。
### 复制算法
复制算法（Copying）是在标记清除算法上演化而来的，用于解决标记清除算法的内存碎片问题。它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。
当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。这样就保证了内存的连续性，逻辑清晰，运行高效。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717378208371-7687ce5f-958e-4d71-b247-db9ed2ddf269.png#averageHue=%23faf8f8&clientId=u94a46ed1-3676-4&from=paste&id=uf39f74d1&originHeight=736&originWidth=1694&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ue10786d2-f808-4080-ad9c-70167432b89&title=)
但复制算法也存在一个很明显的问题，内存空间只能拿出一半用来使用，代价实在太高了。
### 标记整理算法
标记整理算法（Mark-Compact），标记过程仍然与标记清除算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，再清理掉端边界以外的内存区域。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717378303785-84a48bdf-920a-4389-a835-030969fd1f8d.png#averageHue=%23f7f4f4&clientId=u94a46ed1-3676-4&from=paste&id=u772e0ca0&originHeight=662&originWidth=1686&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ub51d4919-fc52-4268-9c9d-d0747fbad64&title=)
标记整理算法一方面在标记-清除算法上做了升级，解决了内存碎片的问题，也规避了复制算法只能利用一半内存区域的弊端。看起来很美好，但内存变动更频繁，需要整理所有存活对象的引用地址，在效率上比复制算法差很多。
### 分代收集算法
**分代收集算法（Generational Collection）**严格来说并不是一种思想或理论，而是融合上述 3 种基础的算法思想，而产生的针对不同情况所采用不同算法的一套组合拳。
根据对象存活周期的不同会将内存划分为几块，一般是把 Java 堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717378339011-a2cdd230-85f1-4420-a1f6-db30331fa0b2.png#averageHue=%238bad6f&clientId=u94a46ed1-3676-4&from=paste&id=uedcb0cd5&originHeight=324&originWidth=814&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u7c2ba808-ab00-463b-a041-7bfec35d39a&title=)
在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。
老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用标记清理或者标记整理算法来进行回收。
## 新生代和老年代
堆（Heap）是 JVM 中最大的一块内存区域，也是垃圾收集器管理的主要区域。
堆主要分为 2 个区域，年轻代与老年代，其中年轻代又分 Eden 区和 Survivor 区，其中 Survivor 区又分 From 和 To 两个区。
![heap.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717380614528-fd1488f6-f6ca-42d8-b376-be67b34c7488.png#averageHue=%23fbf8f6&clientId=u94a46ed1-3676-4&from=paste&height=767&id=uae3554aa&originHeight=959&originWidth=1858&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=30588&status=done&style=none&taskId=u150666ff-84e2-425e-bbf2-3219a6cc1be&title=&width=1486.4)
### Eden 区
据 IBM 公司之前的研究表明，有将近 98% 的对象是朝生夕死，所以针对这一现状，大多数情况下，对象会在新生代 Eden 区中进行分配，当 Eden 区没有足够空间进行分配时，JVM 会发起一次 Minor GC，Minor GC 相比 Major GC 更频繁，回收速度也更快。
通过 Minor GC 之后，Eden 区中绝大部分对象会被回收，而那些无需回收的存活对象，将会进到 Survivor 的 From 区，如果 From 区不够，则直接进入 To 区。
### Minor GC 和 Major GC
#### Minor GC
**Minor GC**是指发生在新生代（Young Generation）的垃圾回收操作。由于大多数对象在新生代中被创建并迅速成为垃圾，Minor GC的回收频率较高，但回收速度也相对较快。
**过程**：

1. **对象分配**：新对象首先分配在Eden区。
2. **Eden区满**：当Eden区没有足够的空间分配新对象时，会触发一次Minor GC。
3. **存活对象移动**：在Minor GC过程中，Eden区的所有存活对象会被移动到Survivor区。如果Survivor区空间不足，部分存活对象会直接晋升到老年代（Old Generation）。
4. **清空Eden区**：完成回收后，Eden区会被清空，等待下一次对象分配。
#### Major GC（或Full GC）
**Major GC**（也称为**Full GC**）是指发生在老年代（Old Generation）的垃圾回收操作。由于老年代对象的生命周期较长，Major GC的发生频率较低，但回收速度较慢，并且通常会导致应用程序暂停（Stop-The-World）。
**过程**：

1. **触发条件**：当老年代的空间不足以分配新的对象或存活对象晋升时，会触发一次Major GC。
2. **全堆回收**：Major GC通常会对整个堆（包括新生代和老年代）进行回收。
3. **存活对象移动**：老年代的存活对象在回收过程中可能会被压缩和整理，以减少内存碎片。
### Survivor 区
Survivor 区相当于是 Eden 区和 Old 区的一个缓冲，类似于我们交通灯中的黄灯。
#### 为啥需要 Survivor 区？
如果没有 Survivor 区，Eden 区每进行一次 Minor GC，存活的对象就会被送到老年代，老年代很快就会被填满。而有很多对象虽然一次 Minor GC 没有消灭，但其实也并不会蹦跶多久，或许第二次，第三次就需要被清除。
这时候移入老年区，很明显不是一个明智的决定。
所以，Survivor 的存在意义就是减少被送到老年代的对象，进而减少 Major GC 的发生。Survivor 的预筛选保证，只有经历 16 次 Minor GC 还能在新生代中存活的对象，才会被送到老年代。
#### Survivor 区为啥划分为两块？
设置两个 Survivor 区最大的好处就是解决内存碎片化，我们先假设一下，Survivor 只有一个区域会怎样。
Minor GC 执行后，Eden 区被清空，存活的对象放到了 Survivor 区，而之前 Survivor 区中的对象，可能也有一些是需要被清除的。那么问题来了，这时候我们怎么清除它们？
在这种场景下，我们只能标记清除，而我们知道标记清除最大的问题就是内存碎片，在新生代这种经常会消亡的区域，采用标记清除必然会让内存产生严重的碎片化。
但因为 Survivor 有 2 个区域，所以每次 Minor GC，会将之前 Eden 区和 From 区中的存活对象复制到 To 区域。第二次 Minor GC 时，From 与 To 职责兑换，这时候会将 Eden 区和 To 区中的存活对象再复制到 From 区域，以此反复。
这种机制最大的好处就是，整个过程中，永远有一个 Survivor space 是空的，另一个非空的 Survivor space 是无碎片的。
那么，Survivor 为什么不分更多块呢？比方说分成三个、四个、五个？
显然，如果 Survivor 区再细分下去，每一块的空间就会比较小，容易导致 Survivor 区满，两块 Survivor 区可能是经过权衡之后的最佳方案。
### Old 区
老年代占据着 2/3 的堆内存空间，只有在 Major GC 的时候才会进行清理，每次 GC 都会触发“Stop-The-World”。内存越大，STW 的时间也越长，所以内存也不仅仅是越大就越好。
由于复制算法在对象存活率较高的老年代会进行很多次的复制操作，效率很低，所以老年代这里采用的是标记整理算法。
除了上述所说，在内存担保机制下，无法安置的对象会直接进到老年代，以下几种情况也会进入老年代。
#### 大对象
大对象指需要大量连续内存空间的对象，这部分对象不管是不是“朝生夕死”，都会直接进到老年代。这样做主要是为了避免在 Eden 区及 2 个 Survivor 区之间发生大量的内存复制。当你的系统有非常多“朝生夕死”的大对象时，得注意了。
#### 长期存活对象
虚拟机给每个对象定义了一个对象年龄（Age）计数器。正常情况下对象会不断的在 Survivor 的 From 区与 To 区之间移动，对象在 Survivor 区中每经历一次 Minor GC，年龄就增加 1 岁。当年龄增加到 15 岁时，这时候就会被转移到老年代。当然，这里的 15，JVM 也支持进行特殊设置 `**-XX:MaxTenuringThreshold=10**`
可通过 `**java -XX:+PrintFlagsFinal -version | findstr MaxTenuringThreshold (windows)**`查看默认的阈值。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381094373-bcc21586-ae12-45ee-a3ed-856ce4afdf31.png#averageHue=%231c1c1c&clientId=u94a46ed1-3676-4&from=paste&height=106&id=uc1190500&originHeight=132&originWidth=1403&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=27000&status=done&style=none&taskId=u840e0273-a0ed-4172-8386-3ba3e55c1fa&title=&width=1122.4)
#### 动态对象年龄
JVM 并不强制要求对象年龄必须到 15 岁才会放入老年区，如果 Survivor 空间中某个年龄段的对象总大小超过了 Survivor 空间的一半，那么该年龄段及以上年龄段的所有对象都会在下一次垃圾回收时被晋升到老年代，无需等你“成年”。
有点类似于负载均衡，轮询是负载均衡的一种，保证每台机器都分得同样的请求。看似很均衡，但每台机器的硬件不同，健康状况不同，所以我们可以基于每台机器接收的请求数、响应时间等，来调整负载均衡算法。
这种动态调整机制有助于优化内存使用和减少垃圾收集的频率，特别是在处理大量短生命周期对象的应用程序时
> 参考链接 1：[二哥Java进阶之路 · 深入理解垃圾回收机制](https://www.javabetter.cn/jvm/gc.html)
> 参考链接 2：[从头到尾再讲一次 Java 的垃圾回收](https://zhuanlan.zhihu.com/p/73628158)

