数组的大小是固定的，一旦创建的时候指定了大小，就不能再调整了。也就是说，如果数组满了，就不能再添加任何元素了。ArrayList 在数组的基础上实现了自动扩容，并且提供了比数组更丰富的预定义方法（各种增删改查），非常灵活。
### System.arraycopy
```java
System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)

参数说明：
src：目标源数组，即你需要复制的数组
srcPos：源数组的起始位置，从该位置开始复制数组
dest：目标数组，即复制后的数组
destPos：目标数组的起始位置，从该位置复制到目标数组
length：你需要复制元素的数量


int[] sourceArray = {1, 2, 3, 4, 5};
int[] destinationArray = new int[5];

System.arraycopy(sourceArray, 0, destinationArray, 0, sourceArray.length);

// 输出目标数组的内容
for (int num : destinationArray) {
    System.out.print(num + " ");
}

输出：1 2 3 4 5
```
### 创建ArrayList
```java
ArrayList<String> list = new ArrayList<String>();
//上方语句是创建一个String 类型的 ArrayList，ArrayList可以通过<>来指定数组的类型，
//如果其他类型的数据时，会产生编译错误

List<String> list = new ArrayList<>();
//也可以通过这种方式创建ArrayList

//此时会调用此构造方法，初始化一个空数组
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
//一个为空的常量
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

//如果你非常确定数组的大小时，可以再创建的时候指定容量大小
ArrayList<String> list = new ArrayList<String>(28);
```
### 如何往 ArrayList 中添加元素
我们可以先看一下集合在添加过程中执行了哪些操作；
```
堆栈过程图示：
add(element)
└── if (size == elementData.length) // 判断是否需要扩容，第一次添加会初始化容量 10
    ├── grow(minCapacity) // 扩容
    │   └── newCapacity = oldCapacity + (oldCapacity >> 1) // 计算新的数组容量
    │   └── Arrays.copyOf(elementData, newCapacity) // 创建新的数组
    ├── elementData[size++] = element; // 添加新元素
    └── return true; // 添加成功
```

1. 查看具体 `add（）`源码
```java
/**
 * 添加元素添加到集合末尾
 * @param 需要添加的元素信息
 * @return 添加成功返回 
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1); //判断是否需要扩容以及初始化容量
    elementData[size++] = e; //将数据添加到对应下标
    return true; //返回添加结果
}
```

2. 继续查看`ensureCapacityInternal()`
   1. 此时 minCapacity 为1（ size + 1 传递过来的）、
   2. elementData用于存放元素的底层数组，创建集合时是为空的
   3. DEFAULTCAPACITY_EMPTY_ELEMENTDATA 默认为 `{}`
   4. DEFAULT_CAPACITY 默认为10，`private static final int DEFAULT_CAPACITY = 10;`
```java
/**
*	确保集合可以容纳指定元素
*/
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) { //判断是否第一次添加，如果是则需要初始化容量
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity); //使用DEFAULT_CAPACITY跟最小容量进行比较
    }

    ensureExplicitCapacity(minCapacity); //确保能够容纳指定元素的数据
}
```

3. 接下来执行 `ensureExplicitCapacity() `方法
```java
/*
* 判断集合容量是否足够，如果不够则需要扩容
*/
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 检查是否超出数组范围
    if (minCapacity - elementData.length > 0)
        //执行扩容方法
        grow(minCapacity);
}
```

4. 最后就是执行扩容方法`grow()`，对`>>`不了解的可以[点击此处](https://www.javabetter.cn/basic-grammar/operator.html#_03%E3%80%81%E4%BD%8D%E8%BF%90%E7%AE%97%E7%AC%A6)学习
```java
/**
 * 扩容 ArrayList 的方法，确保能够容纳指定容量的元素
 *
 * @param 指定容量最小的值
 */
private void grow(int minCapacity) {
    // 获取旧容量大小
    int oldCapacity = elementData.length;
    //在旧容量的基础上扩大1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 判断新容量大小是否足够
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //如果超出了数组的最大长度就使用int最大值
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 扩容数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

5. 最后回到`add()`完成数据的添加
```java
 public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```
### 如何往 ArrayList 指定位置添加元素
在ArrayList 中，还可以通过 `add(int index, E element) `方法把元素添加到 ArrayList 的指定位置：
```java
/**
 *	在指定位置插入指定元素。 移动当前位于该位置的元素该索引右侧有后续元素（为其索引加一）。
 * @param index 要插入元素的索引下标
 * @param element 要插入的值
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    //检查下标是否越界
    rangeCheckForAdd(index);
    //确保容量足够，不够则需要扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //拷贝数组
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    //设置元素
    elementData[index] = element;
    //元素数量 + 1
    size++;
}
```
### 更新ArrayList 中的元素
可以通过 `set()` 进行修改，但是需要提供下标和新的元素
```java
/**
 * 使用指定元素替换指定索引元素
 *
 * @param index 需要替换元素的下标
 * @param element 新元素
 * @return 旧元素信息
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E set(int index, E element) {
    //检查下标是否越界
    rangeCheck(index);
    //通过数组下标获取以前的旧元素
    E oldValue = elementData(index);
    //将新元素替换上去
    elementData[index] = element;
    return oldValue;
}
```
### 删除ArrayList 中的元素
`remove(int index) `方法用于删除指定下标位置上的元素，`remove(Object o)` 方法用于删除指定值的元素。

1. `remove(int index)`
```java
/**
 * 删除指定位置的元素
 *
 * @param index 需要删除元素的下标
 * @return 被删除的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    //检查下标是否越界
    rangeCheck(index);

    modCount++;
    //获取旧元素
    E oldValue = elementData(index);

    //计算从要移除元素的后一个元素到列表末尾的元素数量
    int numMoved = size - index - 1;
    //如果需要移动元素 > 0，就需要进行数组拷贝
    if (numMoved > 0)
        //如果被移除元素后面还有元素，这行代码会将这些元素向前移动一位，覆盖掉被移除的元素
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //将数组末尾的元素置为 null，以便让垃圾回收机制回收该元素占用的空间
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

2. `remove(Object o)`
```java
/**
 * 该方法通过遍历的方式找到要删除的元素，null 的时候使用 == 操作符判断，
 * 非 null 的时候使用 equals() 方法，然后调用 fastRemove() 方法。
 *
 * @param o 需要删除的元素
 * @return <tt>true</tt> if this list contained the specified element
 */
public boolean remove(Object o) {
    //判断需要删除的元素是否为null
    if (o == null) {
        for (int index = 0; index < size; index++)
            //遍历数组找到该元素进行删除
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            // 不为null时遍历数组，通过 equals 方法进行对比是否相同
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

3. `fastRemove()`
```java
/*
 * 快速删除指定位置的元素
 * @param index 要删除的元素的索引
 */
private void fastRemove(int index) {
    modCount++;
    //计算从要移除元素的后一个元素到列表末尾的元素数量
    int numMoved = size - index - 1;

    if (numMoved > 0)
        //如果被移除元素后面还有元素，这行代码会将这些元素向前移动一位，覆盖掉被移除的元素
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //列表最后一个元素设为null，为了让垃圾收集器可以回收这个元素，防止内存泄漏。
    elementData[--size] = null; // clear to let GC do its work
}
```
### 查找ArrayList 中的元素

1. `indexOf()`
```java
/**
 * 返回指定元素在列表中第一次出现的位置。
 * 如果列表不包含该元素，则返回 -1。
 *
 * @param o 要查找的元素
 * @return 指定元素在列表中第一次出现的位置；如果列表不包含该元素，则返回 -1
 */
public int indexOf(Object o) {
    if (o == null) { //如果要找的元素是null
        for (int i = 0; i < size; i++)
            if (elementData[i]==null) //遍历数组并且对比是否为null
                return i;  //返回索引下标
    } else { //如果查找的元素不是null
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i])) //遍历数组通过 equals 进行比较
                return i; //返回索引下标
    }
    return -1; //查找不到返回 -1
}
```

2. `lastIndexOf()`和`indexOf()`类似，只不过是从最后开始遍历的
```java
/**
 * 返回指定元素在列表中最后一次出现的位置。
 * 如果列表不包含该元素，则返回 -1。
 *
 * @param o 要查找的元素
 * @return 指定元素在列表中最后一次出现的位置；如果列表不包含该元素，则返回 -1
 */
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```
