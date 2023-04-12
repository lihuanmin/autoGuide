# :avocado:一、ThreadLocal

## :eggplant:1.1 介绍

`ThreadLocal`线程变量，是用于线程内部存储变量，这些变量是每一个线程独享的，线程互相之间是访问不到变量的。线程变量与普通变量的区别在于，每一个线程都有`ThreadLocal`变量的副本，`ThreadLocal`变量通常是用`private static`修饰，当一个线程结束的时，它所使用的所有`ThreadLocal`变量的实例副本都可被回收。

`ThreadLocal`变量适用于线程需要独立实例，但是该实例需要在方法间共享。

## :potato:1.2 使用场景

从上述的介绍可以看出线程变量适用于线程需要独立变量并且变量需要在方法间共享。在实际的开发中主要有以下几个场景

- 保存用户信息

  把用户信息保存在`ThreadLocal`中，线程在方法中通过`ThreadLocal`获取用户信息

- 在不同的方法中传递信息，在方法1中有一个变量，在方法中需要使用，常规做法是通过方法传参，但是当方法调用层级太多之后，不需要使用变量的方法也要负责传参，如果通过`ThreadLocal来`实现，只需要在方法1中`set`进去，在方法3中`get`就行。

  ```java
  void method1(){
    int a = 1;
    method2();
  }
  void method2(){
    method3();
  }
  void method3(){
    // 需要使用变量a
  }
  ```

## :carrot:1.3 代码示例

下面写一个简单的示例，演示`ThreadLocal`如果在方法中传递值的。用两个线程模拟实际项目中用户的两次访问。用户第一次访问的时候在`ThreadLocal`设置了一个值，然后访问`method1`，然后再访问`method2`，在`method2`通过`ThreadLocal`的`get`方法可以获取设置的值，并且两次访问的设置值和获取值是相互隔离的。

```java
public class Demo {
    static ThreadLocal<String> local = new ThreadLocal<>();
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            local.set("用户第一次访问，这是我请求的参数");
            method1();
        }, "用户第一次访问的线程");
        t1.start();

        Thread t2 = new Thread(() -> {
            local.set("用户第二次访问，这是我请求的参数");
            method1();
        }, "用户第二次访问的线程");
        t2.start();
    }

    private static void method1() {
        method2();
    }

    private static void method2() {
        System.out.println(Thread.currentThread().getName() + ": " + local.get());
    }
}
```

## :corn:1.4 使用原理

要理解`ThreadLocal`使用原理，首先要了解使用`ThreadLocal`涉及到的几个类：`Thread`、`ThreadLocal`、`ThreadLocalMap`、`Entry`。

在`Thread`类里面有一个`ThreadLocalMap`的变量，而`ThreadLocalMap`是`ThreadLocal`的一个内部类，同时`ThreadLocalMap`有一个`Entry`数组类型的变量，这个`Entry`类是继承了弱引用。

![](E:/project/study/doc/Java/Java%E5%9F%BA%E7%A1%80/%E5%A4%9A%E7%BA%BF%E7%A8%8B/pic/threadlocal3.png)

`ThreadLocal`的`set`其实是调用了`ThreadLocalMap`的`set`，而`ThreadLocalMap`的`set`是在`Entry[]`里面存放一个`Entry`对象，而`Entry`的构造方法有两个参数，一个`key`和一个`value`，`key`就是`ThreadLocal`的引用，而`value`是我们要设置的值。

![](E:/project/study/doc/Java/Java%E5%9F%BA%E7%A1%80/%E5%A4%9A%E7%BA%BF%E7%A8%8B/pic/threadlocal2.png)

## :hot_pepper:1.5 源码解析

- ThreadLocal的set方法

  `ThreadLocal`的`set`就干了一件事情，调用当前线程的`ThreadLocalMap`变量的`set`方法，如果当前线程的`ThreadLocalMap`变量为空，就新建一个`ThreadLocalMap`变量。`ThreadLocalMap`的`set`方法有两个参数，第一个是当前`ThreadLocal`对象，另外一个是要设置的`value`。

  ```java
  public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
      map.set(this, value);
    else
      createMap(t, value);
  }
  ```

- ThreadLocalMap的set方法

  `ThreadLocalMap`的`set`主要是将传入进来的`key`和`value `封装成一个`Entry`对象，然后把`Entry`对象放到`ThreadLocalMap`对象的一个`Entry`数组属性上，本质上就是往数组上添加元素。在数组中存放元素，首先要考虑的就是两个问题：如何保证散列和如何解决冲突，常见的散列算法有：除余散列、随机散列、斐波那契散列等。

  **在`ThreadLocalMap`中采用斐波那契保证了散列，具体过程如下**:

  在添加新元素之前，首先的确定新元素在数组中的索引，在确定新元素的索引是下面这行代码，调用`ThreadLocal`的`threadLocalHashCode`变量，然后和数组长度减1做一个与运算，因为`tab`的长度`len`是2的`n`次方，`len-1`转换成二进制相当于高位是0，后面全是1，`key.threadLocalHashCode `和`len-1`做与运算相当于`key.threadLocalHashCode `只保留低`n`位。

  ```java
  // 确定索引
  int i = key.threadLocalHashCode & (len-1);
  
  private final int threadLocalHashCode = nextHashCode();
  
  private static AtomicInteger nextHashCode = new AtomicInteger();
  
  private static final int HASH_INCREMENT = 0x61c88647;
  
  private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
  }
  ```

  然后看下`key.threadLocalHashCode `的原理并且是如何和斐波那契产生关联的，看下面的源码。

  调用`threadLocalHashCode`会调用一个`nextHashCode`方法，这个方法是一个`AtomicInteger`类型变量，初始值是0，调用一次就会加上这个常量`HASH_INCREMENT`，而这个常量是`0x61c88647`换算成十进制是1640531527。

  而黄金分割数(Math.sqrt(5) - 1)/2乘(1L << 32)也是1640531527，这样`0x61c88647`就和黄金分割数产生了关联，而黄金分割数能保证均匀。

  > 斐波那契数列的一些性质比如在 n 很大时， Fn+1/Fn≈ϕ ，其中 ϕ 是黄金分割数，等于 1.618

  

  **在`ThreadLocalMap`中如何解决冲突，具体过程如下：**

  在确定了新元素在`tab`上的索引的时候，这个位置有可能有元素，这样就产生了冲突，当冲突产生了之后`ThreadLocalMap`采用线性探测开放地址来解决冲突，冲突了之后调用`nextIndex`方法确定下一个索引位置。每次冲突了索引就增加1，当索引大于数组长度又从0开始

  ```java
  private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
  }
  ```

  `k == key `当前`key`和当前索引的`k`相等的时候，相当于在线程内调用同一个`ThreadLocal`的`set`方法两次，就用新的`value`替换以前的值即可。

  `k = null`，也就是数组的当前位置`Entry`对象不为空但是`key`是空的也就是`key`被回收了，说明当前索引位置可以用，用新的`key`和`value`调用`replaceStaleEntry`替换掉即可。

  > 在replaceStaleEntry有个比较有意思的设计，就是该方法判断当前节点已经出现了key=null的情况，那说明该数组其它位置也有这样的情况，会做一次删除脏Entry的操作。

  ```java
  private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
      ThreadLocal<?> k = e.get();
      if (k == key) {
        e.value = value;
        return;
      }
      if (k == null) {
        replaceStaleEntry(key, value, i);
        return;
      }
    }
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
      rehash();
  }
  ```

# :strawberry:二、原理解析

## :cherries:2.1 内存划分

![](E:/project/study/doc/Java/Java%E5%9F%BA%E7%A1%80/%E5%A4%9A%E7%BA%BF%E7%A8%8B/pic/threadlocal1.png)

`JVM`会在堆空间创建一个`Thread`对象，同时也会在堆空间创建一个`ThreadLocal`对象，当进行`set`操作的时候，在堆空间创建一个`Entry`对象，`Entry`对象的`key`弱引用`ThreadLocal`对象

## :peach:2.2 内存泄漏

### 2.2.1 为什么会内存泄漏

从上面内存划分的图中可以得到三条引用路线：

①强引用：ThreadLocalRef -> ThreadLocal对象

②强引用：ThreadRef -> Thread对象 -> ThreadLocalMap对象 -> Entry对象 -> value对象

③弱引用：ThreadRef -> Thread对象 -> ThreadLocalMap对象 -> Entry对象 --> ThreadLocal对象

当引用路线①断开的时候，`ThreadLocal`对象只有一条弱引用路径了，一旦`JVM`执行了`GC`操作，那么`ThreadLocal`对象就被回收了，那么线路③最后`key`指向的`ThreadLocal`为`null`了。

但是线路①的引用一直存在，就会导致`value`没有被回收，导致这样的`value`越来越多就容易内存溢出了。

这里其实有个疑问，为什么线路①一直存在，按理说线程的生命周期结束这条线路也就不会存在了，但是现在一般都是用线程池，线程都是复用的，就会导致这样的情况越来越多。

### 2.2.2 如何解决内存泄漏

:1st_place_medal:`ThreadLocal`在设计的时候就考虑到这个问题，一般在`set`值的时候会主动去清理一次脏`Entry`，就是`key`为`null`，但是值还存在的数据。

:2nd_place_medal:我们平时在使用的时候应该主动的去调用一次`remove`方法。

:3rd_place_medal:一般我们在使用的时候将`ThreadLocal`变量定义成`private static`的，这样`ThreadLocal`不会轻易被回收，也就是`Entry`的`key`不会为空，这样我们就可以获取到`value`值，然后`remove`掉，能防止内存溢出。

## :pear:2.3 key为什么是弱引用

因为当`ThreadLocal`被回收了以后`Entry`的`key`就会为`null`，而`ThreadLocalMap`的`set`方法会主动的清理一次这样的脏`Entry`，就算是用户忘了手动`remove`，这里也多了一次程序自动操作，这样能避免内存溢出。

如果`ThreadLocal`设计成强引用，就算是`TreadLocalRef`的引用断开了，也会因为`Entry`的`key`引用导致堆空间的`ThreadLocal`对象不能被回收，这样加大的内存溢出的概率。

## 2.4 ThreadLocal线程安全的吗

不一定，如果多个线程`set`的是同一个对象，其它线程去`get`的时候也是同一个对象引用，这样并不能保证线程安全。