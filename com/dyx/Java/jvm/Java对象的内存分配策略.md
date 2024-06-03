Java 的对象是在堆中创建的，但堆又分为新生代和老年代，新生代又细分为 Eden、From Survivor、To Survivor。**那我们创建的对象到底在哪里**？
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381527078-aa8b9b48-e39f-48bb-bdc9-ff3ce5465692.png#averageHue=%238bad6f&clientId=u162be008-018a-4&from=paste&id=u3a3d932f&originHeight=324&originWidth=814&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u8d0436ae-fe27-4bc4-8206-5f2621900f8&title=)
## 对象优先在 Eden 分配
堆分为新生代和老年代，新生代用于存放使用后就要被回收的对象（朝生夕死），老年代用于存放生命周期比较长的对象。
我们创建的大部分对象，都属于生命周期较短的对象，所以会存放在新生代。新生代又细分 Eden、From Survivor、To Survivor，那我们创建的对象会优先在 Eden 区分配。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381590780-f55ae9c1-1a01-417e-a273-5f610d97494e.png#averageHue=%238ce6d9&clientId=u162be008-018a-4&from=paste&id=uef1141b1&originHeight=301&originWidth=732&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u4ef884fb-05e1-4b29-aee3-425055a3108&title=)
随着对象的不断创建，Eden 剩余地内存空间就会越来越少，随后就会触发 Minor GC，于是 JVM 会把 Eden 区存活的对象转入 From Survivor 空间。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381590743-11f51609-91c6-4b58-af47-f6f525961c2e.png#averageHue=%238ee8db&clientId=u162be008-018a-4&from=paste&id=u6fc849ca&originHeight=312&originWidth=732&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u3b9741b9-f908-4267-8a9a-1f4d436ca89&title=)
Minor GC 后，又创建的新对象会继续往 Eden 区分配。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381591367-438d567f-42cb-42db-a84d-aa71bb4417a2.png#averageHue=%238de8db&clientId=u162be008-018a-4&from=paste&id=ud7a17435&originHeight=299&originWidth=732&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u782bd541-d177-4486-8553-b7acba74d29&title=)
于是，随着新对象的创建，Eden 的剩余内存空间就会越来越少，又会触发 Minor GC，此时，JVM 会对 Eden 区和 From Survivor 区中的对象进行存活判断，对于存活的对象，会转移到 To Survivor 区。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381591370-357757d2-06c5-4fb7-9834-6d54000aae8f.png#averageHue=%23b0e3ee&clientId=u162be008-018a-4&from=paste&id=u144bc4d6&originHeight=295&originWidth=732&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u781f98c8-1bba-4620-9592-551d4d10691&title=)
下一次 Minor GC，存活的对象又会从 To 到 From，这样就总有一个 Survivor 区是空的，而另外一个是无碎片的。
## 大对象直接进入老年代
对于上面的流程，也有例外的存在，如果一个对象很大，一直在 Survivor 空间复制来复制去，就会很浪费性能，所以这些大对象会直接进入老年代。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381623213-112838b0-e45a-478a-a4a5-eb56f2b0aaff.png#averageHue=%238de7da&clientId=u162be008-018a-4&from=paste&id=u95de0a54&originHeight=309&originWidth=732&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ua0706388-91ca-4589-aeb8-af15509344f&title=)
这种策略的目的是减少垃圾回收时的复制开销，因为大对象的复制比小对象更耗时。
可以通过 `**-XX:PretenureSizeThreshold**` 参数设置直接分配大对象到老年代的阈值。如果对象的大小超过这个阈值，它将直接在老年代中分配。例如，如果想将阈值设置为 1MB（1024KB），可以这样设置：
```
-XX:PretenureSizeThreshold=1048576
```
## 长期存活的对象将进入老年代
对象在每次从一个 Survivor 区转移到另外一个 Survivor 区时，它的年龄就会增加。当对象的年龄达到一定阈值（默认为 15），则它会被转移到老年代。
可以用 `**-XX:PretenureSizeThreshold=10**` 来设置年龄。
虚拟机为了给对象计算他到底经历了几次 Minor GC，会给每个对象定义了一个对象年龄计数器。如果对象在 Eden 中经过第一次 Minor GC 后仍然存活，移动到 Survivor 空间年龄加 1，在 Survivor 区中每经历过 Minor GC 后仍然存活年龄再加 1。年龄到了 15，就到了老年代。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381662945-2c37cc31-7348-4bac-be40-cf52ff2f14c3.png#averageHue=%23afe3ee&clientId=u162be008-018a-4&from=paste&id=uaf46d169&originHeight=305&originWidth=732&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=uccba8d33-96f1-4be8-a7e8-eba52aa523b&title=)
## 动态年龄判断
除了年龄达到 **MaxTenuringThreshold**，还有另外一个方式进入老年代，那就是动态年龄判断：JVM 会检查每个年龄段的对象大小，并估算它们在 Survivor 空间中所占的总体积。JVM 会选择一个最小的年龄，使得该年龄及以上的对象可以填满 Survivor 空间的一部分（通常小于总空间的一半），然后将这些对象晋升到老年代。
比如 Survivor 是 100M，Hello1 和 Hello2 都是 3 岁，且总和超过了 50M，Hello3 是 4 岁，这个时候，这三个对象都将到老年代。
![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381693464-14ffe5b5-9712-4425-9e23-4220fe7a847d.png#averageHue=%238de6d9&clientId=u162be008-018a-4&from=paste&id=ud0281ba7&originHeight=305&originWidth=732&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=uec662d93-2b34-4090-b611-878a4a0226d&title=)
## 空间分配担保
上面提到过，存活的对象会放入另外一个 Survivor 空间，如果这些存活的对象比 Survivor 空间还大呢？
整个流程如下：

- Minor GC 之前，JVM 会先检查老年代最大可用的连续空间是否大于新生代所有对象的总空间，如果大于，则发起 Minor GC。
- 如果小于，则看 HandlePromotionFailure 有没有设置，如果没有设置，就发起 Full GC。
- 如果设置了 HandlePromotionFailure，则看老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果小于，就发起 Full GC。
- 如果大于，发起 Minor GC。Minor GC 后，看 Survivor 空间是否足够存放存活对象，如果不够，就放入老年代，如果够放，就直接存放 Survivor 空间。如果老年代都不够放存活对象，担保失败（Handle Promotion Failure），发起 Full GC。

![](https://cdn.nlark.com/yuque/0/2024/png/22796888/1717381693488-592eb6f6-f325-46f6-8e06-53a6a6b0a5d9.png#averageHue=%23f8f3f3&clientId=u162be008-018a-4&from=paste&id=u8abed9b3&originHeight=585&originWidth=646&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u704ab2c8-bced-4493-90c0-f2fe765fae3&title=)
HandlePromotionFailure 的作用，当设置为 true 时（默认值），JVM 会尝试继续 Minor GC，即使老年代空间不足以容纳所有需要晋升的对象。JVM 会尝试清理更多的老年代空间或者采用其他措施来应对空间不足的情况。避免因为老年代空间不足而过早触发 Full GC（全堆回收）。Full GC 通常比 Minor GC 更耗时，会导致更长时间的停顿。
## 栈和方法区
Java 创建的对象几乎都在堆中，这包括通过 new 关键字创建的对象和数组。
对象的引用，通常存放在栈中，比如说当你在方法中声明一个变量 `**MyClass obj = new MyClass(); **`时，变量 obj（一个指向堆中对象的引用）存储在栈上。
方法区用于存储已被 JVM 加载的类信息、常量、静态变量以及即时编译器编译后的代码。
Java 8 中，永久代被元空间（Metaspace）所取代。元空间使用本地内存（操作系统的内存），而非 JVM 内存
