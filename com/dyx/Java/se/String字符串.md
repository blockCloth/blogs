## String字符串为什么是不可变的

### String字符串的不可变性

- 因为String字符串是被 final 修饰的，它不可以被子类继承，所以子类也无法修改它的方法
- String的数据存储在`char[]`中，它也是被 final 修饰的，数据只会初始一次并且不能被修改

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

### String不可变性的优点

- 可以保证String 对象的安全性，避免被篡改，类似密码这些信息都是保存到String 中的
- 保证 HASH 值不会频繁刷新，经常刷新的话会导致性能下降，毕竟String 的使用量太多了

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

- 可以实现**字符常量池**，Java会将相同内容的字符串存入到常量池中。当具有相同内容的字符串就可以指向一个字符串对象，节省了内存空间

> 注：由于字符串的不可变性，很多方法都是返回一个新的String 对象
>
> ```java
> public String substring(int beginIndex) {
>     if (beginIndex < 0) {
>         throw new StringIndexOutOfBoundsException(beginIndex);
>     }
>     int subLen = value.length - beginIndex;
>     if (subLen < 0) {
>         throw new StringIndexOutOfBoundsException(subLen);
>     }
>     return (beginIndex == 0) ? this : new String(value, beginIndex, subLen); //创建一个新对象
> }
> 
> public String replace(char oldChar, char newChar) {
>     if (oldChar != newChar) {
>         int len = value.length;
>         int i = -1;
>         char[] val = value; /* avoid getfield opcode */
> 
>         while (++i < len) {
>             if (val[i] == oldChar) {
>                 break;
>             }
>         }
>         if (i < len) {
>             char buf[] = new char[len];
>             for (int j = 0; j < i; j++) {
>                 buf[j] = val[j];
>             }
>             while (i < len) {
>                 char c = val[i];
>                 buf[i] = (c == oldChar) ? newChar : c;
>                 i++;
>             }
>             return new String(buf, true); //构建一个新的String对象
>         }
>     }
>     return this;
> }
> ```



## 字符串常量池

### new String()

`String s = new String("test")`，当使用 `new` 关键字创建String对象时，总共会创建两个对象；

- 当创建一个新对象时，Java虚拟机会现在字符串常量池中查找有没有 `"test"`这个字符串对象，如果没有就会先在字符串常量池中创建一个`"test"`对象，然后再在堆中创建一个`"test"`字符串对象，最后堆中的字符串对象的引用地址就会赋值给变量 s
- 若字符串常量池中存在`"test"`这个字符串对象，就会直接在堆中创建一个`"test"`字符串对象，然后把这个对象地址赋值给变量s

### 字符串常量池的作用

​		通常情况下，创建字符串的一般使用双引号方式创建字符串`String s = "test" `  ，当执行`String s = "test" ` 时，JVM会先在字符串常量池中查找有没有 "test" 这个字符串。如果有的话，不会创建任何对象，直接将字符串中的对象引用地址赋值给变量；没有的话，则先在字符串常量池中创建一个对象，在将对象引用地址赋值给变量

- 使用`new `关键字创建字符串时，不会管字符串是否存在相同内容，都会重新创建一个String 对象
- 使用`"双引号"`方式创建字符串时，会先从字符串常量池中查找是否用相同内容的字符串，没有才会创建一个新的字符串

