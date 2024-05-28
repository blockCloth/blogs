Java基本数据类型缓存池解析（IntegerCache）

​		基本数据类型中，除了Float 和 Double之外，其它留个包装类中（Integer，Long，Character，Boolean，Short，Byte）都有自己的常量缓存池

- Byte：-127~128 包含整个char
- Integer：-127~128
- Long：-127~128
- Boolean：true / false
- Short：-127~128
- Character：\u0000 - \u007F

​		在Integer 中，它的内部中内置了256个Integer类型的缓存数据，当它的使用数据在 -127~128 之中时，会直接返回常量池中的数据~引用。而不是去重新创建一个引用对象，但是当超出这个范围是会创建一个新的对象引用。当使用 `Integer.valueOf()` 方法时，如果对应的整数值没有在内部的整数缓存池中找到，则会创建一个新的 `Integer` 对象。

```java
public static Integer valueOf(int i) {
    //先判断value范围是否在cache中
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        //返回常量值
        return IntegerCache.cache[i + (-IntegerCache.low)];
    //当缓存中不存在时则创建一个新值
    return new Integer(i);
}
```

```java
private static class IntegerCache {
    static final int low = -128; //初始化最低值
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127; //设置默认最高值
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high"); //从配置中查看是否手动设置值
        if (integerCacheHighPropValue != null) {
            try {
                //将这个值转成int类型，最大值为Integer.MAX_VALUE - 129
                int i = parseInt(integerCacheHighPropValue); 
                i = Math.max(i, 127); //比较最大值
                // Maximum array size is Integer.MAX_VALUE  避免h超出最大值Integer.MAX_VALUE - 129
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1]; //构建cache容量
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

