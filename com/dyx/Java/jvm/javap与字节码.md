## javap
`javap` 是 JDK 自带的一个命令行工具，主要用于反编译类文件（.class 文件）。Java 内置了一个反编译命令 `javap`，可以通过`javap -help`来了解`javap`的基本用法：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716970665164-858dae08-7c59-498a-a119-6820772b8db5.png#averageHue=%23191919&clientId=u3cd52e9e-b80c-4&from=paste&height=501&id=u0e4cf4b1&originHeight=626&originWidth=968&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=87755&status=done&style=none&taskId=u7dde243d-a9d5-4d41-99a3-e0906394271&title=&width=774.4)
javap 主要用于反编译 Java 类文件，即将编译后的 .class 文件转换回更易于理解的形式。虽然它不会生成原始的 Java 源代码，但它可以显示类的结构，包括构造方法、方法、字段等，帮助我们更好地理解 Java 字节码以及 Java 程序的运行机制
### 实例类
```java
public class Main {
    private int age = 18;
    public int getAge() {
        return age;
    }
}

```
### 通过 javap 反编译文件
我们在 class 文件的同级目录下输入命令 `javap -v -p Main.class` 来查看一下输出的内容（-v 显示附加信息，如局部变量表、操作码等；-p 显示所有类和成员，包括私有的
```java
Classfile /D:/JavaCount/project/javabatterdemo/out/production/jvm/com/github/dyx/hello/Main.class
  Last modified 2024年5月29日; size 393 bytes
  SHA-256 checksum cded8ed514b6efe7c31f2f4b051c0b0ffb14b43148b21a05df9af87a47777207
  Compiled from "Main.java"
public class com.github.dyx.hello.Main
  minor version: 0
  major version: 52
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #3                          // com/github/dyx/hello/Main
  super_class: #4                         // java/lang/Object
  interfaces: 0, fields: 1, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #4.#18         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#19         // com/github/dyx/hello/Main.age:I
   #3 = Class              #20            // com/github/dyx/hello/Main
   #4 = Class              #21            // java/lang/Object
   #5 = Utf8               age
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lcom/github/dyx/hello/Main;
  #14 = Utf8               getAge
  #15 = Utf8               ()I
  #16 = Utf8               SourceFile
  #17 = Utf8               Main.java
  #18 = NameAndType        #7:#8          // "<init>":()V
  #19 = NameAndType        #5:#6          // age:I
  #20 = Utf8               com/github/dyx/hello/Main
  #21 = Utf8               java/lang/Object
{
  private int age;
    descriptor: I
    flags: (0x0002) ACC_PRIVATE

  public com.github.dyx.hello.Main();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: bipush        18
         7: putfield      #2                  // Field age:I
        10: return
      LineNumberTable:
        line 3: 0
        line 4: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/github/dyx/hello/Main;

  public int getAge();
    descriptor: ()I
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field age:I
         4: ireturn
      LineNumberTable:
        line 6: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/github/dyx/hello/Main;
}
SourceFile: "Main.java"
```
## 字节码
### 字节码的基本信息

1. **第一行：**表示字节码文件的位置
```java
Classfile /D:/JavaCount/project/javabatterdemo/out/production/jvm/com/github/dyx/hello/Main.class
```

2. **第二行：**字节码文件的修改日期
```java
Last modified 2024年5月29日; size 393 bytes
```

3. **第三行：**字节码文件的 SHA-256 值，用于校验文件的完整性
```java
SHA-256 checksum cded8ed514b6efe7c31f2f4b051c0b0ffb14b43148b21a05df9af87a47777207
```

4. **第四行：**表名该字节码文件编译于哪个源文件
```java
Compiled from "Main.java"
```

5. **第五行：**类访问修饰符和类型，表名这是一个公开的类，名为`com.github.dyx.hello.Main`
```java
public class com.github.dyx.hello.Main
```

6. **第六行：**minor version: 0，次版本号。
7. **第七行：**major version: 52，主版本号
8. **第八行：**类访问标记，表示当前类是`ACC_PUBLIC | ACC_SUPER`（表示这个类是 `public` 的，并且使用了 `super` 关键字）
```java
flags: (0x0021) ACC_PUBLIC, ACC_SUPER
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716971894884-e2391a4b-6aad-4577-8001-31e2784494f3.png#averageHue=%23f8f8f8&clientId=u965de59c-cfd6-4&from=paste&height=394&id=ud5ebc497&originHeight=493&originWidth=1373&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=49725&status=done&style=none&taskId=u14626366-574f-4770-bd4d-08dd29229f6&title=&width=1098.4)

9. **第九行：**当前类的索引，指向常量池中下标为  的常量
```java
this_class: #3                          // com/github/dyx/hello/Main
```

10. **第十行：**父类的索引，指向常量池中下标为  的常量，可以看出当前类的父类是 Object 类
```java
super_class: #4                         // java/lang/Object
```

11. **第十一行：**当前类有 0 个接口，1 个字段（age），2 个方法（`getAge()`方法和缺省的默认构造方法），1 个属性（该类仅有的一个属性是 SourceFIle，包含了源码文件的信息，第一行讲过了）
```java
interfaces: 0, fields: 1, methods: 2, attributes: 1
```
### 常量池
在 Java 中，**.class** 文件包含了类的字节码信息，而这些字节码信息中最重要的部分之一就是常量池（Constant Pool）。常量池可以被看作是字节码文件的资源仓库，存放了许多重要的信息，包括字面量和符号引用。

- **字面量（Literal）**：
   - 类似于 Java 中的常量概念，如文本字符串，**final** 常量等。
- **符号引用（Symbolic References）**，属于编译原理的概念，包含以下三种：
   - 类和接口的全限定名
   - 字段的名称和描述符
   - 方法的名称和描述符

在 Java 虚拟机（JVM）加载字节码文件时，会进行动态链接，这意味着字段和方法的符号引用在运行期会被转换成实际的内存地址。当 JVM 运行时，需要从常量池中获取这些符号引用，并在类创建或运行时解析并翻译到具体的内存地址上。
#### 常量池的详细解释
_注_：

- # 号后面跟的是索引，索引没有从 0 开始而是从 1 开始，是因为设计者考虑到，“如果要表达不引用任何一个常量的含义时，可以将索引值设为 0 来表示”（周志明老师《深入理解 Java 虚拟机》一书描述的）。
- = 号后面跟的是常量的类型，没有包含前缀 CONSTANT_ 和后缀 _info。
- **全文中提到的索引等同于下标**，为了灵活描述，没有做统一。

**第 1 个常量**
```
 #1 = Methodref          #4.#18         // java/lang/Object."<init>":()V
```

- **类型**：**Methodref** 表示方法引用，是用来定义方法的
- **解释**：指向常量池中下标为 4 和 18 的常量。

**第 4 个常量**
```
#4 = Class              #21            // java/lang/Object
```

- **类型**：**Class** 表示类（接口）引用。
- **解释**：指向常量池中下标为 21 的常量。

**第 21 个常量**
```
#21 = Utf8               java/lang/Object
```

- **类型**：**Utf8** 表示 UTF-8 编码的字符串。
- **值**：**java/lang/Object**

**第 18 个常量**
```
#18 = NameAndType        #7:#8          // "<init>":()V
```

- **类型**：**NameAndType** 表示字段或方法的部分符号引用。
- **解释**：指向常量池中下标为 7 和 8 的常量。

**第 7 个常量**
```
#7 = Utf8               <init>
```

- **类型**：**Utf8** 表示 UTF-8 编码的字符串。
- **值**：**<init>**，表示构造方法。

**第 8 个常量**
```
#8 = Utf8               ()V
```

- **类型**：**Utf8** 表示 UTF-8 编码的字符串。
- **值**：**()V**，表示方法的返回类型为 **void**。

组合起来，第 1 个常量表示 **Main** 类使用的是默认的构造方法，来源于 **Object** 类。`#4`执行`Class #21`（即`java/lang/Object`），`#18`指向`NameAndType #7:#8`（即`<init>:()V`）
**第 2 个常量**
```
#2 = Fieldref           #3.#19         // com/github/dyx/hello/Main.age:I
```

- **类型**：**Fieldref** 表示字段引用。
- **解释**：指向常量池中下标为 3 和 19 的常量。

**第 3 个常量**
```
#3 = Class              #20            // com/github/dyx/hello/Main
```

- **类型**：**Class** 表示类（接口）引用。
- **解释**：指向常量池中下标为 20 的常量。

**第 19 个常量**
```
#19 = NameAndType        #5:#6          // age:I
```

- **类型**：**NameAndType** 表示字段或方法的部分符号引用。
- **解释**：指向常量池中下标为 5 和 6 的常量。

**第 5 个常量**
```
#5 = Utf8               age
```

- **类型**：**Utf8** 表示 UTF-8 编码的字符串。
- **值**：**age**，表示字段名。

**第 6 个常量**
```
#6 = Utf8               I
```

- **类型**：**Utf8** 表示 UTF-8 编码的字符串。
- **值**：**I**，表示字段的类型为 **int**。
#### 字段类型描述符映射表
在解释字段类型时，我们需要使用描述符。以下是字段类型的描述符映射表：

| **Java 类型** | **描述符** |
| --- | --- |
| 基本数据类型 int | I |
| 基本数据类型 long | J |
| 基本数据类型 float | F |
| 基本数据类型 double | D |
| 基本数据类型 boolean | Z |
| 基本数据类型 char | C |
| 基本数据类型 short | S |
| 基本数据类型 byte | B |
| 引用数据类型，以分号“；”结尾 | L...; |
| 一维数组 | [ |

### 字段表集合
字段表用来描述接口或者类中声明的变量，包括类变量和成员变量，但不包含声明在方法中局部变量。
**字段的修饰符一般有：**

- 访问权限修饰符，比如 `**public private protected**`
- 静态变量修饰符，比如 `**static**`
- `**final**` 修饰符
- 并发可见性修饰符，比如 `**volatile**`
- 序列化修饰符，比如 `**transient**`

然后是字段的类型（可以是**基本数据类型、数组和对象**）和名称。
在 `Main.class` 字节码文件中，字段表的信息如下所示。
```java
private int age;
    descriptor: I
    flags: (0x0002) ACC_PRIVATE
```
表明字段的访问权限修饰符为 `**private**`，类型为 `**int**`，名称为 `**age**`。字段的访问标志和类的访问标志非常类似。
#### 访问修饰符列表
| **标记名** | **标记值** | **释义** |
| --- | --- | --- |
| ACC_PUBLIC | 0x0001 | 是否为 **public** |
| ACC_PRIVATE | 0x0002 | 是否为 **private** |
| ACC_PROTECTED | 0x0004 | 是否为 **protected** |
| ACC_STATIC | 0x0008 | 是否为 **static** |
| ACC_FINAL | 0x0010 | 是否为 **final** |
| ACC_SYNCHRONIZED | 0x0020 | 是否为同步的 |
| ACC_VOLATILE | 0x0040 | 是否为 **volatile** |
| ACC_TRANSIENT | 0x0080 | 是否为 **transient** |
| ACC_NATIVE | 0x0100 | 是否为本地方法 |
| ACC_INTERFACE | 0x0200 | 是否为接口 |
| ACC_ABSTRACT | 0x0400 | 是否为抽象类/方法 |
| ACC_STRICT | 0x0800 | 限制浮点计算以确保移植性 |
| ACC_SYNTHETIC | 0x1000 | 是否由编译器自动生成 |
| ACC_ANNOTATION | 0x2000 | 是否为注解 |
| ACC_ENUM | 0x4000 | 是否为枚举 |
| ACC_VARARGS | 0x0080 | 是否接受可变参数 |

### 方法表集合
方法表用来描述接口或者类中声明的方法，包括类方法和成员方法，以及构造方法。方法的修饰符和字段略有不同，比如说 **volatile** 和 **transient** 不能用来修饰方法，再比如说方法的修饰符多了** synchronized、native、strictfp 和 abstract**。
#### 构造方法
以下是构造方法的字节码解析：
```java
 public com.github.dyx.hello.Main();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: bipush        18
         7: putfield      #2                  // Field age:I
        10: return
      LineNumberTable:
        line 3: 0
        line 4: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/github/dyx/hello/Main;
```

- **声明：**`public com.github.dyx.hello.Main();` 这是 Main 类的构造方法，用于创建 Main 类的实例。它是公开的（public）。
- **描述符：**`descriptor: ()V`这表示构造方法没有参数 (`()`) 并且没有返回值 （`V`，代表 `void`）。
- **访问标志：**`flags: (0x0001) ACC_PUBLIC`，表示这个构造方法是公开的，可以从其他类中访问。

详细看下`code`属性：
```java
Code:
  stack=2, locals=1, args_size=1
     0: aload_0
     1: invokespecial #1                  // Method java/lang/Object."<init>":()V
     4: aload_0
     5: bipush        18
     7: putfield      #2                  // Field age:I
    10: return
  LineNumberTable:
    line 3: 0
    line 4: 4
  LocalVariableTable:
    Start  Length  Slot  Name   Signature
        0      11     0  this   Lcom/github/dyx/hello/Main;
```

- **stack=2**：stack 为最大操作数栈，Java 虚拟机在运行的时候会根据这个值来分配栈帧的操作数栈深度，这里的值为 2，意味着操作数栈的深度为 2。操作栈是一个 LIFO（后进先出）栈，用于存放临时变量和中间结果。在构造方法中，bipush 和 aload_0 指令可能会同时需要栈空间，所以需要 2 个操作数栈深度。
- **locals=1**：locals 为局部变量所需要的存储空间，单位为槽**（slot）**，方法的参数变量和方法内的局部变量都会存储在**局部变量表**中。
   - 局部变量表的容量以变量槽为最小单位，一个变量槽可以存放一个 **32** 位以内的数据类型，比如 **boolean、byte、char、short、int、float、reference** 和 **returnAddress** 类型。
   - 局部变量表所需的容量大小是在编译期间完成计算的，大小由编译器决定，因此不同的编译器编译出来的字节码可能会不一样。locals=1，这表示局部变量表中有 1 个变量的空间。
   - 对于实例方法（如构造方法），局部变量表的第一个位置（索引 0）总是用于存储 this 引用。
- **args_size=1**：方法的参数个数为 1（包括隐含的 **this** 引用）只要不是静态方法，都会有一个当前类的对象 this 悄悄的存在着
- **LineNumberTable**，该属性的作用是描述源码行号与字节码行号(字节码偏移量)之间的对应关系。这对于调试非常重要，因为它允许调试器将正在执行的字节码指令精确地关联到源代码的特定行。
```java
LineNumberTable:
    line 6: 0
    line 7: 4
```

   - 这里的意思是，第 6 行对应的字节码行号为 0，第 7 行对应的字节码行号为 4。
   - 在调试过程中，当一个断点被触发或出现异常时，通过 LineNumberTable，我们可以知道这是源代码中的哪一行导致的。
- **LocalVariableTable**，该属性的作用是描述帧栈中的局部变量与源码中定义的变量之间的关系。大家仔细看一下，就能看到 this 的影子了。
   - Start 和 Length：定义变量在方法中的作用域。Start 是变量生效的字节码偏移量，Length 是它保持活动的长度。
   - Slot：变量在局部变量数组中的索引。
   - Name：变量的名称，如在源代码中定义的。
   - Signature：变量的类型描述符。

这里，只有一个局部变量 this，它指代构造方法正在初始化的对象。它的作用域是从指令偏移量 0 开始，持续整个方法的长度（长度为 11），并且被分配到局部变量表的第一个槽位（索引 0）。`Lcom/github/dyx/hello/Main`; 表明这个变量的类型是 `com/github/dyx/hello/Main`
#### 成员方法
下面这部分为成员方法 `getAge()`，返回类型为 `int`，访问标志为 `public`。
```java
public int getAge();
  descriptor: ()I
  flags: (0x0001) ACC_PUBLIC
```
理解了构造方法的 Code 属性后，再看 `getAge()`方法的 Code 属性时，就很容易理解了。
```java
Code:
  stack=1, locals=1, args_size=1
      0: aload_0
      1: getfield      #2                  // Field age:I
      4: ireturn
  LineNumberTable:
    line 9: 0
  LocalVariableTable:
    Start  Length  Slot  Name   Signature
        0       5     0  this   Lcom/itwanger/jvm/Main;
```
最大操作数栈为 1，局部变量所需要的存储空间为 1，方法的参数个数为 1，是因为局部变量只有一个隐藏的 this，并且字节码指令中只执行了一次 `**aload_0**`。
①、字节码指令

- **aload_0:  **加载 this 引用到栈顶，以便接下来访问实例字段 age。
- **getfield #2:  **获取字段值。这条指令读取 this 对象的 age 字段的值，并将其推送到栈顶。#2 是对常量池中的字段引用。
- **ireturn: ** 返回栈顶整型值。这里返回的是 age 字段的值。

②、附加信息
`**LineNumberTable**`** **和 `**LocalVariableTable**`** **同样提供了源代码的行号对应和局部变量信息，有助于调试和理解代码的执行流程。
