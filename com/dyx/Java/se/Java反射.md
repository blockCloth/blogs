## 反射简介
![img](https://raw.githubusercontent.com/dunwu/images/master/cs/java/javacore/xmind/Java%E5%8F%8D%E5%B0%84.svg)
### 什么是反射
反射(Reflection)是 Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。
**通过反射机制，可以在运行时访问 Java 对象的属性，方法，构造方法等。**
### 反射的应用场景
反射的主要应用场景有：

- **开发通用框架** - 反射最重要的用途就是开发各种通用框架。很多框架（比如 Spring）都是配置化的（比如通过 XML 文件配置 JavaBean、Filter 等），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象。
- **动态代理** - 在切面编程（AOP）中，需要拦截特定的方法，通常，会选择动态代理方式。这时，就需要反射技术来实现了。
- **注解** - 注解本身仅仅是起到标记作用，它需要利用反射机制，根据注解标记去调用注解解释器，执行行为。如果没有反射机制，注解并不比注释更有用。
- **可扩展性功能** - 应用程序可以通过使用完全限定名称创建可扩展性对象实例来使用外部的用户定义类。
### 反射的缺点

- **性能开销** - 由于反射涉及动态解析的类型，因此无法执行某些 Java 虚拟机优化。因此，反射操作的性能要比非反射操作的性能要差，应该在性能敏感的应用程序中频繁调用的代码段中避免。
- **破坏封装性** -反射允许访问私有字段和私有方法，所以可能会破坏封装从而导致安全问题
- **内部曝光** - 由于反射允许代码执行在非反射代码中非法的操作，例如访问私有字段和方法，所以反射的使用可能会导致意想不到的副作用，这可能会导致代码功能失常并可能破坏可移植性。反射代码打破了抽象，因此可能会随着平台的升级而改变行为。
## 使用反射
### java.lang.reflection 包
Java中的`java.lang.reflecction`包提供了反射功能，`java.lang.reflecction`中的类都没有`public`修饰的构造方法
`java.lang.reflect` 包的核心接口和类如下：

- `Member `接口：反映关于单个成员(字段或方法)或构造函数的标识信息。
- `Field `类：提供一个类的域的信息以及访问类的域的接口。
- `Method `类：提供一个类的方法的信息以及访问类的方法的接口。
- `Constructor `类：提供一个类的构造函数的信息以及访问类的构造函数的接口。
- `Array `类：该类提供动态地生成和访问 JAVA 数组的方法。
- `Modifier `类：提供了 static 方法和常量，对类和成员访问修饰符进行解码。
- `Proxy` 类：提供动态地生成代理类和类实例的静态方法
### 获取Class对象的方法

1. `class.forName()`静态方法
```java
public class ReflectionDemo01 {
    public static void main(String[] args) {
        try {
            User user = new User();
            user.setName("zhangsan");
            Class<?> aClass = Class.forName("com.github.dyx.reflection.User");
            Constructor<?> constructor = aClass.getConstructor();
            //实例化对象
            Object object = constructor.newInstance();

            //获取方法
            Method setNameMethod = aClass.getMethod("setName", String.class);
            //执行方法
            setNameMethod.invoke(object,"张三");
            //获取方法
            Method getNameMethod = aClass.getMethod("getName");
            System.out.println(getNameMethod.invoke(object));
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        } catch (NoSuchMethodException e) {
            throw new RuntimeException(e);
        } catch (InvocationTargetException e) {
            throw new RuntimeException(e);
        } catch (InstantiationException e) {
            throw new RuntimeException(e);
        } catch (IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}
```

2. `类名.class`
```java
public class ReflectionDemo02 {
    public static void main(String[] args) throws Exception {
        Class<Map> c1 = Map.class;
        System.out.println(c4.getCanonicalName());

        Class<String[][]> c2 = String[][].class;
        System.out.println(c5.getCanonicalName());
    }
}
```

3. `Object.getClass`
```java
public class ReflectClassDemo03 {
    enum E {A, B}

    public static void main(String[] args) {
        Class c = "foo".getClass();
        System.out.println(c.getCanonicalName());

        Class c2 = ReflectClassDemo03.E.A.getClass();
        System.out.println(c2.getCanonicalName());

        byte[] bytes = new byte[1024];
        Class c3 = bytes.getClass();
        System.out.println(c3.getCanonicalName());

        Set<String> set = new HashSet<>();
        Class c4 = set.getClass();
        System.out.println(c4.getCanonicalName());
    }
}
```
### 如何创建实例
通过反射来创建实例对象主要有两种方式：

1. 用`class`对象的`newInstance`方法
2. 用`Constructor`对象的`newInstance`方法
```java
public class NewInstanceDemo {
    public static void main(String[] args) throws Exception {
        Class<?> c1 = StringBuilder.class;
        StringBuilder builder = (StringBuilder) c1.newInstance();
        builder.append("hello");
        System.out.println(builder.toString());

        Class<?> c2 = String.class;
        Constructor<?> constructor = c2.getConstructor(String.class);
        String str = (String) constructor.newInstance("new instance");
        System.out.println(str);
    }
}

```
### 创建数组实例
数组在Java中是比较特殊的一种类型，它可以赋值给一个对象引用。在Java中，可以通过`Array.newInstance`创建数组的实例。注：此`Array`是`java.lang.reflection`包中的类
```java
package java.lang.reflect;
public final class Array {}

public class ReflectionArrayDemo {
    public static void main(String[] args) throws Exception {
        Class<?> aClass = Class.forName("java.lang.String");
        Object o = Array.newInstance(aClass, 5);
        Array.set(o,0,"one");
        Array.set(o,1,"two");
        Array.set(o,2,"three");
        Array.set(o,3,"four");
        Array.set(o,4,"five");

        for (int i = 0; i < 5; i++) {
            System.out.println(Array.get(o,i));
        }
    }
}
```
### Field

- `getFiled `- 根据名称获取公有的（public）类成员。
- `getDeclaredField` - 根据名称获取已声明的类成员。但不能得到其父类的类成员。
- `getFields `- 获取所有公有的（public）类成员。
- `getDeclaredFields` - 获取所有已声明的类成员。
```java
public class ReflectionFiledDemo {

    public static void main(String[] args) throws Exception{
        Field[] fields = String.class.getFields();
        //获取所有public修饰的字段
        for (Field field : fields) {
            System.out.println(field);
        }
        System.out.println("---------------------------------------------------");

        Field[] declaredFields = String.class.getDeclaredFields();
        //获取所有字段包括（public、private、default、protected）
        for (Field declaredField : declaredFields) {
            System.out.println(declaredField);
        }
    }
}


public static final java.util.Comparator java.lang.String.CASE_INSENSITIVE_ORDER
---------------------------------------------------
private final char[] java.lang.String.value
private int java.lang.String.hash
private static final long java.lang.String.serialVersionUID
private static final java.io.ObjectStreamField[] java.lang.String.serialPersistentFields
public static final java.util.Comparator java.lang.String.CASE_INSENSITIVE_ORDER
```
### Method

- `getMethod `- 返回类或接口的特定方法。其中第一个参数为方法名称，后面的参数为方法参数对应 Class 的对象。
- `getDeclaredMethod `- 返回类或接口的特定声明方法。其中第一个参数为方法名称，后面的参数为方法参数对应 Class 的对象。
- `getMethods `- 返回类或接口的所有 public 方法，包括其父类的 public 方法。
- `getDeclaredMethods `- 返回类或接口声明的所有方法，包括 public、protected、默认（包）访问和 private 方法，但不包括继承的方法。

**获取一个 Method 对象后，可以用 invoke 方法来调用这个方法。**
```java
public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException


public class ReflectionMethodDemo {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        //获取System的所有方法
        Method[] methods = System.class.getMethods();
        // 只获取public修饰的方法
        for (Method method : methods) {
            System.out.println(method);
        }
        System.out.println("----------------------------------------------------");
        //获取所有方法
        Method[] declaredMethods = System.class.getDeclaredMethods();
        for (Method declaredMethod : declaredMethods) {
            System.out.println(declaredMethod);
        }

        // 利用 Method 的 invoke 方法调用 System.currentTimeMillis()
        Method method = System.class.getMethod("currentTimeMillis");
        System.out.println(method);
        System.out.println(method.invoke(null));
    }
}
```
### Constructor

- `getConstructor `- 返回类的特定 public 构造方法。参数为方法参数对应 Class 的对象。
- `getDeclaredConstructor `- 返回类的特定构造方法。参数为方法参数对应 Class 的对象。
- `getConstructors `- 返回类的所有 public 构造方法。
- `getDeclaredConstructors `- 返回类的所有构造方法。

**获取一个 Constructor 对象后，可以用 newInstance 方法来创建类实例。**
```java
public class ReflectionConstructorDemo {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {

        Constructor<?>[] constructors = String.class.getConstructors();
        //返回所有public修饰的构造函数
        for (Constructor<?> constructor : constructors) {
            System.out.println(constructor);
        }
        System.out.println("----------------------------------------------------");

        //返回String所有构造函数
        Method[] declaredMethods = String.class.getDeclaredMethods();
        for (Method declaredMethod : declaredMethods) {
            System.out.println(declaredMethod);
        }

        //通过构造函数创建对象
        Constructor<String> constructor = String.class.getConstructor(String.class);
        String str = constructor.newInstance("通过反射newInstance创建对象");
        System.out.println(str);

    }
}

```
### 反射输出String
可以通过以上方法对`String`进行反射，获取字段、构造方法、方法；以下是反射之后的内容，代码放在后面，有兴趣的小伙伴可以自行编写一下；
```
public final class java.lang.String{
	private final char[] java.lang.String.value
	private int java.lang.String.hash
	private static final long java.lang.String.serialVersionUID
	private static final java.io.ObjectStreamField[] java.lang.String.serialPersistentFields
	public static final java.util.Comparator java.lang.String.CASE_INSENSITIVE_ORDER

	public java.lang.String(byte[],int,int){

	}
	public java.lang.String(byte[],java.nio.charset.Charset){

	}
	public java.lang.String(byte[],java.lang.String) throws java.io.UnsupportedEncodingException{

	}
	public java.lang.String(byte[],int,int,java.nio.charset.Charset){

	}
	public java.lang.String(byte[],int,int,java.lang.String) throws java.io.UnsupportedEncodingException{

	}
	java.lang.String(char[],boolean){

	}
	public java.lang.String(java.lang.StringBuilder){

	}
	public java.lang.String(java.lang.StringBuffer){

	}
	public java.lang.String(byte[]){

	}
	public java.lang.String(int[],int,int){

	}
	public java.lang.String(){

	}
	public java.lang.String(char[]){

	}
	public java.lang.String(java.lang.String){

	}
	public java.lang.String(char[],int,int){

	}
	public java.lang.String(byte[],int){

	}
	public java.lang.String(byte[],int,int,int){

	}


	public boolean java.lang.String.equals(java.lang.Object){

	}
	public java.lang.String java.lang.String.toString(){

	}
	public int java.lang.String.hashCode(){

	}
	public int java.lang.String.compareTo(java.lang.String){

	}
	public int java.lang.String.compareTo(java.lang.Object){

	}
	public int java.lang.String.indexOf(java.lang.String,int){

	}
	public int java.lang.String.indexOf(java.lang.String){

	}
	public int java.lang.String.indexOf(int,int){

	}
	public int java.lang.String.indexOf(int){

	}
	static int java.lang.String.indexOf(char[],int,int,char[],int,int,int){

	}
	static int java.lang.String.indexOf(char[],int,int,java.lang.String,int){

	}
	public static java.lang.String java.lang.String.valueOf(int){

	}
	public static java.lang.String java.lang.String.valueOf(long){

	}
	public static java.lang.String java.lang.String.valueOf(float){

	}
	public static java.lang.String java.lang.String.valueOf(boolean){

	}
	public static java.lang.String java.lang.String.valueOf(char[]){

	}
	public static java.lang.String java.lang.String.valueOf(char[],int,int){

	}
	public static java.lang.String java.lang.String.valueOf(java.lang.Object){

	}
	public static java.lang.String java.lang.String.valueOf(char){

	}
	public static java.lang.String java.lang.String.valueOf(double){

	}
	public char java.lang.String.charAt(int){

	}
	private static void java.lang.String.checkBounds(byte[],int,int){

	}
	public int java.lang.String.codePointAt(int){

	}
	public int java.lang.String.codePointBefore(int){

	}
	public int java.lang.String.codePointCount(int,int){

	}
	public int java.lang.String.compareToIgnoreCase(java.lang.String){

	}
	public java.lang.String java.lang.String.concat(java.lang.String){

	}
	public boolean java.lang.String.contains(java.lang.CharSequence){

	}
	public boolean java.lang.String.contentEquals(java.lang.CharSequence){

	}
	public boolean java.lang.String.contentEquals(java.lang.StringBuffer){

	}
	public static java.lang.String java.lang.String.copyValueOf(char[]){

	}
	public static java.lang.String java.lang.String.copyValueOf(char[],int,int){

	}
	public boolean java.lang.String.endsWith(java.lang.String){

	}
	public boolean java.lang.String.equalsIgnoreCase(java.lang.String){

	}
	public static java.lang.String java.lang.String.format(java.util.Locale,java.lang.String,java.lang.Object[]){

	}
	public static java.lang.String java.lang.String.format(java.lang.String,java.lang.Object[]){

	}
	public void java.lang.String.getBytes(int,int,byte[],int){

	}
	public byte[] java.lang.String.getBytes(java.nio.charset.Charset){

	}
	public byte[] java.lang.String.getBytes(java.lang.String) throws java.io.UnsupportedEncodingException{

	}
	public byte[] java.lang.String.getBytes(){

	}
	public void java.lang.String.getChars(int,int,char[],int){

	}
	void java.lang.String.getChars(char[],int){

	}
	private int java.lang.String.indexOfSupplementary(int,int){

	}
	public native java.lang.String java.lang.String.intern(){

	}
	public boolean java.lang.String.isEmpty(){

	}
	public static java.lang.String java.lang.String.join(java.lang.CharSequence,java.lang.CharSequence[]){

	}
	public static java.lang.String java.lang.String.join(java.lang.CharSequence,java.lang.Iterable){

	}
	public int java.lang.String.lastIndexOf(int){

	}
	public int java.lang.String.lastIndexOf(java.lang.String){

	}
	static int java.lang.String.lastIndexOf(char[],int,int,java.lang.String,int){

	}
	public int java.lang.String.lastIndexOf(java.lang.String,int){

	}
	public int java.lang.String.lastIndexOf(int,int){

	}
	static int java.lang.String.lastIndexOf(char[],int,int,char[],int,int,int){

	}
	private int java.lang.String.lastIndexOfSupplementary(int,int){

	}
	public int java.lang.String.length(){

	}
	public boolean java.lang.String.matches(java.lang.String){

	}
	private boolean java.lang.String.nonSyncContentEquals(java.lang.AbstractStringBuilder){

	}
	public int java.lang.String.offsetByCodePoints(int,int){

	}
	public boolean java.lang.String.regionMatches(int,java.lang.String,int,int){

	}
	public boolean java.lang.String.regionMatches(boolean,int,java.lang.String,int,int){

	}
	public java.lang.String java.lang.String.replace(char,char){

	}
	public java.lang.String java.lang.String.replace(java.lang.CharSequence,java.lang.CharSequence){

	}
	public java.lang.String java.lang.String.replaceAll(java.lang.String,java.lang.String){

	}
	public java.lang.String java.lang.String.replaceFirst(java.lang.String,java.lang.String){

	}
	public java.lang.String[] java.lang.String.split(java.lang.String){

	}
	public java.lang.String[] java.lang.String.split(java.lang.String,int){

	}
	public boolean java.lang.String.startsWith(java.lang.String,int){

	}
	public boolean java.lang.String.startsWith(java.lang.String){

	}
	public java.lang.CharSequence java.lang.String.subSequence(int,int){

	}
	public java.lang.String java.lang.String.substring(int){

	}
	public java.lang.String java.lang.String.substring(int,int){

	}
	public char[] java.lang.String.toCharArray(){

	}
	public java.lang.String java.lang.String.toLowerCase(java.util.Locale){

	}
	public java.lang.String java.lang.String.toLowerCase(){

	}
	public java.lang.String java.lang.String.toUpperCase(){

	}
	public java.lang.String java.lang.String.toUpperCase(java.util.Locale){

	}
	public java.lang.String java.lang.String.trim(){

	}
}
```
```java
public class ReflectionObjectDemo {
    public static void main(String[] args) {
        try {
            StringBuilder sb = new StringBuilder();
            Class<?> aClass = Class.forName("java.lang.String");
            //获取类修饰符
            String modifier = Modifier.toString(aClass.getModifiers());
            String className = aClass.getName();
            sb.append(modifier + " class " + className + "{\n");

            //获取字段
            Field[] fields = aClass.getDeclaredFields();
            for (Field field : fields) {
                sb.append("\t" + field + "\n");
            }

            sb.append("\n");
            //获取构造函数
            Constructor<?>[] constructors = aClass.getDeclaredConstructors();
            for (Constructor<?> constructor : constructors) {
                sb.append("\t" + constructor + "{\n\n\t}\n");

            }

            sb.append("\n\n");
            //获取方法
            Method[] methods = aClass.getDeclaredMethods();
            for (Method method : methods) {
                sb.append("\t" + method + "{\n\n\t}\n");

            }

            sb.append("}");
            System.out.println(sb.toString());
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}

```
## 参考资料

- [Java进阶之路](https://www.javabetter.cn/basic-extra-meal/fanshe.html)
- [钝悟的博客](https://dunwu.github.io/01.Java/01.JavaSE/01.%E5%9F%BA%E7%A1%80%E7%89%B9%E6%80%A7/10.Java%E5%8F%8D%E5%B0%84.html)
