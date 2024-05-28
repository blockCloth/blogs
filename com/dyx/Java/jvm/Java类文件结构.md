> 类文件结构=.class文件的结构=Class文件结构，这三个说法都是一个意思，.class是从文件后缀名的角度来说的，Class是从Java类的角度来说的，类文件结构就是 Class 的中文译名。

## 什么是class文件？
我们用 IDEA 编写一段简单的 Java 代码，文件名为 HelloJVM.java。
```java
package com.github.dyx.hello;

/**
 * @User Administrator
 * @CreateTime 2024/5/27 9:22
 * @className com.github.dyx.hello.HelloJVM
 */
public class HelloJVM {
    public static void main(String[] args) {
        System.out.println("Hello JVM");
    }
}
```
点击编译按钮后，IDEA 会帮我们生成一个名为 HelloJVM.class 的文件，在 target/classes 的对应包目录下。直接双击打开后长下面这样子：
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package com.github.dyx.hello;

public class HelloJVM {
    public HelloJVM() {
    }

    public static void main(String[] args) {
        System.out.println("Hello JVM");
    }
}

```
看起来和源代码很像，只是多了一个空的构造方法。它是 class 文件被 IDEA 自带的反编译工具 Fernflower 反编译后的样子。这就是 class 文件的十六进制形式。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716859559929-ec85e6bc-be94-44a4-9396-c5e717ae2008.png#averageHue=%23f5f3f1&clientId=u0c27bb62-a166-4&from=paste&height=614&id=uc1b911dd&originHeight=768&originWidth=1106&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=270504&status=done&style=none&taskId=u0330e9d4-3ea7-4b29-808a-11e34fd1a1f&title=&width=884.8)
## Java类文件内容
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716859685258-ad1ef162-5c05-4a22-acd2-d95891457088.png#averageHue=%23998963&clientId=u0c27bb62-a166-4&from=paste&height=706&id=u2c132528&originHeight=928&originWidth=568&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u062e698c-0ed8-406e-91f0-ad957fd60dd&title=&width=432)
### 魔数
回看 class 文件的十六进制形式截图。
第一行中有一串特殊的字符 **cafebabe**，它就是一个魔数，是 JVM 识别 class 文件的标志，JVM 会在**验证阶段**检查 class 文件前四个字节是否以该魔数开头，如果不是则会抛出 ClassFormatError。
### 版本号
紧跟着魔数后面的四个字节 0000 0034 分别表示副版本号和主版本号。也就是说，主版本号为 52（0x32 的十进制），也就是 Java 8 对应的版本号，副版本号为 0。计算主版本号也非常简单，就是JDK 版本加上44，就是当前的主版本号。JVM会通过版本号检查类文件是否兼容当前运行的Java版本。
### 常量池
紧跟在版本号之后的是常量池，它包含了类、接口、字段和方法的符号引用，以及字符串字面量和数值常量。这些信息在编译时被创建，并在运行时被Java虚拟机（JVM）使用。
相当于一个资源仓库，主要存放量大类型常量：

- 字面量（Literals）：字面量是不变的数据，主要包括数值（如整数、浮点数）和字符串字面量。例如，一个整数100或一个字符串"Hello World"，在源代码中直接赋值，编译后存储在常量池中。
- 符号引用（Symbolic References）：符号引用是对类、接口、字段、方法等的引用，它们不是由字面量值给出的，而是通过符号名称（如类名、方法名）和其他额外信息（如类型、签名）来表示。这些引用在类文件中以一种抽象的方式存在，它们在类加载时被虚拟机解析为具体的内存地址。
#### 了解Java常量池
Java常量池是一个关键的组成部分，用于存储编译期间生成的各种常量，包括字符串、整数、浮点数等。为了更好地理解常量池的工作机制，我们可以通过一些实际代码示例来进行探讨。
#### 基本数据类型的常量池处理
Java定义了基本数据类型如**boolean**、**byte**、**short**、**char**和**int**，它们在常量池中都会被当做**int**来处理。以下是一个简单的Java代码示例：
```java
public class ConstantTest {
    public final boolean bool = true;
    public final char aChar = 'a';
    public final byte b = 66;
    public final short s = 67;
    public final int i = 68;
}
```
在这个示例中，我们定义了一些**final**的基本类型常量。编译生成的常量会被存储在常量池中。具体地，它们会被处理为**int**类型，并以十六进制表示：

- 布尔值**true**的十六进制是**0x01**
- 字符**a**的十六进制是**0x61**
- 字节**66**的十六进制是**0x42**
- 短整型**67**的十六进制是**0x43**
- 整型**68**的十六进制是**0x44**

所以编译生成的整型常量在 class 文件中的位置如下图所示。第一个字节 **0x03** 表示常量的类型为 **CONSTANT_Integer_info**，是 JVM 定义的 14 种常量类型之一，具体可查看JVM常量类型；对于 int 和 float 来说，它们占 4 个字节；对于 long 和 double 来说，它们占 8 个字节。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716862416362-0aced308-6035-40be-82e3-dfd052774954.png#averageHue=%23e6e2e2&clientId=u0c27bb62-a166-4&from=paste&height=598&id=u7012f4a5&originHeight=672&originWidth=1266&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=uab18dce7-38ed-4cf8-a3bc-9926dfd8ee4&title=&width=1126)
#### 观察long类型的最大值
为了更直观地了解常量池，我们来看一个**long**类型的示例：
```java
public class ConstantTest {
    public final long ong = Long.MAX_VALUE;
}
```
在这个例子中，**Long.MAX_VALUE**的十六进制表示是**0x7FFFFFFFFFFFFFFF**。在常量池中的表示如下：
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716863070362-e13a4082-0992-406e-9a3d-004244e7f86a.png#averageHue=%23e7e4e3&clientId=u0c27bb62-a166-4&from=paste&id=ud6f34bba&originHeight=330&originWidth=1232&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u7f4a6822-04a8-4ad6-920d-b0e392751bd&title=)
#### 字符串常量在常量池中的存储
```java
class Hello {
    public final String s = "hello";
}
```
`“hello”`是一个字符串，它的十六进制为 `68 65 6c 6c 6f`，我们来看一下它在 class 文件中的位置：
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716863417330-c4d15bc9-510f-4cff-ba23-53b0c0e46e8f.png#averageHue=%23e7e6e6&clientId=u0c27bb62-a166-4&from=paste&id=ufd822f8f&originHeight=1012&originWidth=1286&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u2c3407ef-05a6-44eb-b6d7-c28a14815f6&title=)
前面还有 3 个字节，第一个字节` 0x01` 是标识，标识类型为 `CONSTANT_Uft8_info`，第二个和第三个 `0x00 0x05` 用来表示第三部分字节数组的长度，如下图所示：
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716863471452-ca7f9021-495d-4842-ad99-5eace1dd1f07.png#averageHue=%23faf9f8&clientId=u0c27bb62-a166-4&from=paste&id=u543b6ddd&originHeight=442&originWidth=1462&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ue0c177ee-d357-48bf-be97-1a7d943152f&title=)
与`CONSTANT_Uft8_info` 类型对应的，还有一个`CONSTANT_String_info`，用来表示字符串对象的引用（之前代码中的 s），标识是 0x08。前者存储了字符串真正的值，后者并不包含字符串的内容，仅仅包含了一个指向常量池中 `CONSTANT_Uft8_info` 的索引。

- **CONSTANT_Utf8_info 存储实际的字符串值**：用于存储类文件中出现的所有 UTF-8 编码的字符串。这些字符串包括类名、方法名、字段名、描述符、以及字符串字面量的实际内容。
- **CONSTANT_String_info 存储字符串对象的引用**：用于引用常量池中已经存在的 **CONSTANT_Utf8_info** 项。这种引用表示程序中的字符串对象，但并不存储实际的字符串内

**依然以上方例子为例：**
`**CONSTANT_String_info**` 通过索引 19 来找到`**CONSTANT_Uft8_info**`，见下图：
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716863898497-8f26926e-1a27-4690-9096-a71d58af7a78.png#averageHue=%23e6e6e6&clientId=u0c27bb62-a166-4&from=paste&id=u99ad140b&originHeight=1010&originWidth=1244&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ud0794a47-d20e-4c0a-abed-b870668347c&title=)
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716863908778-cbe4e7e0-072f-445d-ba7b-7b1e90f4a2d9.png#averageHue=%23f9f8f7&clientId=u0c27bb62-a166-4&from=paste&id=u754b12bd&originHeight=496&originWidth=2254&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=uc49523a1-f2de-43a1-b188-f96338d5789&title=)
#### 类的全路径限定名在常量池中的位置
`**CONSTANT_Class_info**`，用来表示类和接口，和 `**CONSTANT_String_info**`类似，第一个字节是标识，值为 **0x07**，后面两个字节是常量池索引，指向 `**CONSTANT_Utf8_info ———— 字符串存储的是类或者接口的全路径限定名**`。
拿`**Hello.java**` 类来说，它的全路径限定名为 `**com/itwanger/jvm/Hello**`，对应的十六进制为“`**636f6d2f697477616e6765722f6a766d2f48656c6c6f**`”，是一串 `**CONSTANT_Uft8_info**`，指向它的 `**CONSTANT_Class_info **`在 class 文件中的什么位置呢？
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716865044562-5b36964d-7df1-4f22-b5c4-a1b193ff270f.png#averageHue=%23eeeeee&clientId=u0c27bb62-a166-4&from=paste&height=681&id=u907f5dbf&originHeight=800&originWidth=1200&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ub287869a-7fb6-44d2-b9c4-fb77bab13e5&title=&width=1022)
从上图中可以看到，常量池的总大小为 23，索引为 04 的 `CONSTANT_Class_info` 指向的是是索引为 21 的 CONSTANT_Uft8_info，值为 `com/itwanger/jvm/Hello`。21 的十六进制为 0x15，有了这个信息，我们就可以找到 CONSTANT_Class_info 在 class 文件中的位置了。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716865083374-32bafa29-a048-4bc8-8509-e0dc4d3dcc17.png#averageHue=%23e6e4e4&clientId=u0c27bb62-a166-4&from=paste&id=u42a47605&originHeight=970&originWidth=1262&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u97e36cca-e353-4bd2-b749-d7ea1689eab&title=)
### 常量池项类型及其标识符
| **常量类型** | **标识符 (tag)** | **描述符** |
| --- | --- | --- |
| CONSTANT_Class_info | 0x07 | 类或接口的全限定名 |
| CONSTANT_Fieldref_info | 0x09 | 字段 |
| CONSTANT_Methodref_info | 0x0A | 普通方法 |
| CONSTANT_InterfaceMethodref_info | 0x0B | 接口方法 |
| CONSTANT_String_info | 0x08 | 字符串字面量 |
| CONSTANT_Integer_info | 0x03 | 整数 |
| CONSTANT_Float_info | 0x04 | 浮点数 |
| CONSTANT_Long_info | 0x05 | 长整数 |
| CONSTANT_Double_info | 0x06 | 双精度浮点数 |
| CONSTANT_NameAndType_info | 0x0C | 字段或方法的名称和类型 |
| CONSTANT_Utf8_info | 0x01 | UTF-8编码的字符串 |
| CONSTANT_MethodHandle_info | 0x0F | 方法句柄 |
| CONSTANT_MethodType_info | 0x10 | 方法类型 |
| CONSTANT_InvokeDynamic_info | 0x12 | 动态调用点 |

### 访问标记
常量池之后的区域就是访问标记（Access flags），这个标记用于识别类或接口的访问信息，比如说：

- 到底是 class 类还是 interface 接口
- 是 public吗？
- 是 abstract 抽象类吗？
- 是 final 类吗？
- 等等。

总共有 16 个标记位可供使用，但常用的只有其中 7 个，见下图。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716860343329-c4c5de30-f3fa-4a08-9975-65c3f19428c3.png#averageHue=%23f8f6f4&clientId=u0c27bb62-a166-4&from=paste&height=403&id=u58545511&originHeight=790&originWidth=2024&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u95808eb6-2f21-4edd-a294-cfa19d7e4d3&title=&width=1032)
这里用一个表格来表示下。

| 标记位 | 标识符 | 描述 |
| --- | --- | --- |
| 0x0001 | ACC_PUBLIC | public 类型 |
| 0x0010 | ACC_FINAL | final 类型 |
| 0x0020 | ACC_SUPER | 调用父类的方法时，使用 invokespecial 指令 |
| 0x0200 | ACC_INTERFACE | 接口类型 |
| 0x0400 | ACC_ABSTRACT | 抽象类类型 |
| 0x1000 | ACC_SYNTHETIC | 标记为编译器自动生成的类 |
| 0x2000 | ACC_ANNOTATION | 标记为注解类 |
| 0x4000 | ACC_ENUM | 标记为枚举类 |
| 0x8000 | ACC_MODULE | 标记为模块类 |

### 类索引、父类索引和接口索引
这三部分用来确定类的继承关系，**this_class** 为当前类的索引，**super_class **为父类的索引，**interfaces** 为接口。
来看下面这段简单的代码，没有接口，默认继承 Object 类。
```java
class Hello {
    public static void main(String[] args) {
        
    }
}
```
通过 jclasslib 可以看到类的继承关系。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1716865280544-1c923a67-d729-47bf-8fe4-16ac30c34948.png#averageHue=%23efeeee&clientId=u0c27bb62-a166-4&from=paste&id=u31274d66&originHeight=610&originWidth=1294&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u07e3d810-714f-496d-a61a-4c2e3daf247&title=)

- **this_class** 指向常量池中索引为 2 的 CONSTANT_Class_info。
- **super_class** 指向常量池中索引为 3 的 CONSTANT_Class_info。
- 由于没有接口，所以 interfaces 的信息为空。
### 字段表
在Java类文件中，字段表用于描述类或接口中声明的字段。字段表包含字段的名称、类型、访问权限以及其他属性信息。每个字段在类文件中都有一个对应的**field_info**结构。
#### 字段表的结构
字段表由一个字段计数器和若干**field_info**结构组成。字段计数器表示字段的数量，而每个**field_info**结构包含字段的详细信息。
```java
field_info {
    u2 access_flags;       // 访问标志
    u2 name_index;         // 字段名称索引
    u2 descriptor_index;   // 字段描述符索引
    u2 attributes_count;   // 属性计数
    attribute_info attributes[attributes_count]; // 属性表
}
```

- **access_flags：**指定字段的访问权限和属性，如**public**、**private**、**protected**、**static**、**final**、**volatile**、**transient**等。
- **name_index：**指向常量池中的**CONSTANT_Utf8_info**项，表示字段的名称。
- **descriptor_index：**指向常量池中的**CONSTANT_Utf8_info**项，表示字段的类型描述符。
- **attributes_count：**表示字段的属性数量。
- **attributes：**包含字段的额外信息，如常量值等。
#### 字段描述符（**descriptor_index**）

- **基本数据类型**：单字符表示
   - **I**：int
   - **B**：byte
   - **C**：char
   - **D**：double
   - **F**：float
   - **J**：long
   - **S**：short
   - **Z**：boolean
   - **V**：void（仅用于方法返回类型）
- **引用数据类型**：**L<class-name>;** 格式
   - **Ljava/lang/String;**：String
- **数组类型**：前缀 **[** 表示
   - **[I**：int[]
   - **[Ljava/lang/String;**：String[]
### 方法表
在Java类文件中，**方法表**跟**字段表**一样用于描述类或接口中声明的方法。方法表包含方法的名称、描述符、访问权限以及其他属性信息。每个方法在类文件中都有一个对应的**method_info**结构。
#### 方法表的结构
方法表由一个方法计数器和若干**method_info**结构组成。方法计数器表示方法的数量，而每个**method_info**结构包含方法的详细信息。
```java
method_info {
    u2 access_flags;       // 访问标志
    u2 name_index;         // 方法名称索引
    u2 descriptor_index;   // 方法描述符索引
    u2 attributes_count;   // 属性计数
    attribute_info attributes[attributes_count]; // 属性表
}

```
### 属性表
在Java类文件中，属性表用于描述类、字段、方法和Code段的附加信息。属性表在类文件的多个部分中使用，包括类定义、字段定义、方法定义和方法的Code段。
#### 属性表的结构
属性表由若干**attribute_info**结构组成。每个**attribute_info**结构包含以下信息：
```java
attribute_info {
    u2 attribute_name_index; // 属性名称索引
    u4 attribute_length;     // 属性长度
    u1 info[attribute_length]; // 属性内容
}
```
#### 常见的属性类型
##### 1. Code 属性
**Code**属性用于存储方法的字节码及相关信息，是方法表中的主要属性之一。
**结构**：
```java
Code_attribute {
    u2 attribute_name_index;   // 指向常量池中名称为 "Code" 的 CONSTANT_Utf8_info 项
    u4 attribute_length;       // 属性长度
    u2 max_stack;              // 操作数栈的最大深度
    u2 max_locals;             // 局部变量表的大小
    u4 code_length;            // 字节码的长度
    u1 code[code_length];      // 方法的字节码
    u2 exception_table_length; // 异常表长度
    exception_info exception_table[exception_table_length]; // 异常表
    u2 attributes_count;       // 属性计数
    attribute_info attributes[attributes_count]; // 属性表
}
```
##### 2. ConstantValue 属性
**ConstantValue**属性用于表示字段的常量值，仅适用于**static final**字段。
**结构**：
```java
ConstantValue_attribute {
    u2 attribute_name_index; // 指向常量池中名称为 "ConstantValue" 的 CONSTANT_Utf8_info 项
    u4 attribute_length;     // 属性长度，固定为 2
    u2 constantvalue_index;  // 指向常量池中的常量项
}
```
##### 3. Exceptions 属性
**Exceptions**属性用于列出方法可能抛出的异常。
**结构**：
```java
Exceptions_attribute {
    u2 attribute_name_index;    // 指向常量池中名称为 "Exceptions" 的 CONSTANT_Utf8_info 项
    u4 attribute_length;        // 属性长度
    u2 number_of_exceptions;    // 异常数量
    u2 exception_index_table[number_of_exceptions]; // 指向常量池中 CONSTANT_Class_info 项的索引表
}
```
##### 4. LineNumberTable 属性
**LineNumberTable**属性用于描述字节码指令与源代码行号的对应关系，便于调试。
**结构**：
```java
LineNumberTable_attribute {
    u2 attribute_name_index;     // 指向常量池中名称为 "LineNumberTable" 的 CONSTANT_Utf8_info 项
    u4 attribute_length;         // 属性长度
    u2 line_number_table_length; // 行号表的长度
    line_number_info line_number_table[line_number_table_length]; // 行号表
}
```
**行号表项结构**：
```java
line_number_info {
    u2 start_pc;      // 字节码的偏移量
    u2 line_number;   // 源代码的行号
}
```
##### 5. LocalVariableTable 属性
**LocalVariableTable**属性用于描述局部变量的范围和描述符。
**结构**：
```java
LocalVariableTable_attribute {
    u2 attribute_name_index;      // 指向常量池中名称为 "LocalVariableTable" 的 CONSTANT_Utf8_info 项
    u4 attribute_length;          // 属性长度
    u2 local_variable_table_length; // 局部变量表的长度
    local_variable_info local_variable_table[local_variable_table_length]; // 局部变量表
}
```
**局部变量表项结构**：
```java
local_variable_info {
    u2 start_pc;       // 局部变量的作用范围的起始字节码偏移量
    u2 length;         // 局部变量的作用范围的长度
    u2 name_index;     // 局部变量的名称索引，指向常量池中的 CONSTANT_Utf8_info 项
    u2 descriptor_index; // 局部变量的描述符索引，指向常量池中的 CONSTANT_Utf8_info 项
    u2 index;          // 局部变量表中的索引
}
```
##### 6. SourceFile 属性
**SourceFile**属性用于指出编译时使用的源文件名称。
**结构**：
```java
SourceFile_attribute {
    u2 attribute_name_index; // 指向常量池中名称为 "SourceFile" 的 CONSTANT_Utf8_info 项
    u4 attribute_length;     // 属性长度，固定为 2
    u2 sourcefile_index;     // 指向常量池中的 CONSTANT_Utf8_info 项，表示源文件名称
}
```
### QA
评论区有读者问到：“怎么通过索引值，定位到在class 文件中的位置，这个是咋算的？”
在Java类文件中，常量池是一个索引表，它从索引值1开始计数，每个条目都有一个唯一的索引。

- 常量池计数器：在常量池之前，类文件有一个16位的常量池计数器，表示常量池中有多少项。它的值比实际常量数大1（因为索引从1开始）。
- 常量池条目：每个常量池条目的开始是一个标签（1个字节），表明了常量的类型（如Class、Fieldref、Methodref等）。根据这个类型，后面跟着的数据结构也不同。

定位过程大致如下：

- 读取常量池计数器：首先，从类文件的开头读取常量池计数器的值，确定常量池中有多少条目。
- 遍历常量池：从常量池的第一项开始遍历。由于不同类型的常量长度不同，需要根据每个常量的类型来确定它的长度。
- 根据索引定位：继续遍历，直到到达所需的索引值。每次遍历时，根据条目类型读取相应长度的数据，直到达到目标索引。

可以抽象成一个数组和一个 for 循环，就能明白了。
```java
int[] constantPool = new int[constantPoolCount];
for (int i = 1; i < constantPoolCount; i++) {
    int tag = constantPool[i];
    switch (tag) {
        case CONSTANT_Integer_info:
            i += 4;
            break;
        case CONSTANT_Float_info:
            i += 4;
            break;
        case CONSTANT_Long_info:
            i += 8;
            break;
        case CONSTANT_Double_info:
            i += 8;
            break;
        case CONSTANT_Utf8_info:
            int length = constantPool[i + 1];
            i += length + 1;
            break;
        case CONSTANT_String_info:
            i += 2;
            break;
        case CONSTANT_Class_info:
            i += 2;
            break;
        case CONSTANT_Fieldref_info:
            i += 4;
            break;
        case CONSTANT_Methodref_info:
            i += 4;
            break;
        case CONSTANT_InterfaceMethodref_info:
            i += 4;
            break;
        case CONSTANT_NameAndType_info:
            i += 4;
            break;
        case CONSTANT_MethodHandle_info:
            i += 3;
            break;
        case CONSTANT_MethodType_info:
            i += 2;
            break;
        case CONSTANT_InvokeDynamic_info:
            i += 4;
            break;
        default:
            throw new RuntimeException("Unknown tag: " + tag);
    }
}
```
> 转载：[二哥的Java进阶之路 · Java类文件结构](https://javabetter.cn/jvm/class-file-jiegou.html)

