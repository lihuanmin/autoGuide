# 基本数据类型

## 数据类型分类

在java中有八种基本数据类型

![](../../pic/java_data_type.png)

## 数据类型长度

| 类型    | 长度  | 范围                  | 默认值   |
| ------- | ----- | --------------------- | -------- |
| byte    | 1byte | -128~127              | 0        |
| short   | 2byte | -2^15~2^15-1          | 0        |
| int     | 4byte | -2^31~2^31-1          | 0        |
| long    | 8byte | -2^63~2^63-1          | 0        |
| float   | 4byte | 1.4E-45--3.4028235E38 | 0.0F     |
| double  | 8byte | 4.9E-324 -- 1.79E308  | 0.0D     |
| boolean | 1byte | true,false            | false    |
| char    | 2byte | 0~2^16-1              | '\u0000' |

## 数据类型转换

一般我们在运算时都要求数据类型一致

1.boolean类型不参与转换

2.byte,short,char在运算时都先转换为int。

```java 
byte a = 3;
int b = 4;
int c = a + b;
```

```java 
byte b1 = 3, b2 = 4, b3;
// 会报错，变量相加会看类型问题，最后结果赋值也会考虑类型问题
b3 = b1 + b2;
// 常量相加，会先做加法，然后查看结果是否在赋值的数据类型范围内，如果不是才报错
b3 = 3 + 4;
```

3.可以从大类型强制转换到小类型，目标类型 变量名 = (目标类型)(被转换的数据)

强制转换导致溢出的问题

```java
byte b = (byte)130;
Sysem.out.println(b);//-126
/**
* 计算机中数据运算都是通过补码进行的
* 130的二进制：10000010
* 由于130是正数所以原码，反码，补码一致的
* 做byte截取操作后130为：10000010
* 由10000010求原码
*          符号位         数值位
* 补码：      1           0000010
* 反码        1           0000001
* 原码        1           1111110
*/
```

4.+=和-=包含了强制转换

# 浮点数

## 什么是浮点型

在计算机科学中，浮点是一种对于实数的近似值数值表现法，由一个有效数字（即尾数）加上幂数来表示，通常是乘以某个基数的整数次指数得到。以这种表示法表示的数值，称为浮点数。

一个浮点数a由两个数m和e来表示：a = m × be。在任意一个这样的系统中，我们选择一个基数b（记数系统的基）和精度p（即使用多少位来存储）。m（即尾数）是形如±d.ddd...ddd的p位数（每一位是一个介于0到b-1之间的整数，包括0和b-1）。如果m的第一位是非0整数，m称作正规化的。有一些描述使用一个单独的符号位（s 代表+或者-）来表示正负，这样m必须是正的。e是指数。

由于计算机中保存的小数其实是十进制的小数的近似值，并不是准确值，所以，千万不要在代码中使用浮点数来表示金额等重要的指标。

建议使用BigDecimal或者Long（单位为分）来表示金额。

## 单精度和双精度

位（bit）是衡量浮点数所需存储空间的单位，通常为32位或64位，分别被叫作单精度和双精度。

单精度浮点数在计算机存储器中占用4个字节（32 bits），比起单精度浮点数，双精度浮点数(double)使用 64 位（8字节） 来存储一个浮点数。

# 包装类型

## 什么是包装类型

Java是面向对象的语言，所以八种基本类型都有对应的包装类型。包装类都是final类不能被继承。

| 基本数据类型 | 包装类    |
| ------------ | --------- |
| byte         | Byte      |
| short        | Short     |
| int          | Integer   |
| long         | Long      |
| float        | Float     |
| double       | Double    |
| char         | Character |
| boolean      | Boolean   |

## 自动拆箱装箱

为了使基本类型和包装类型关联起来，Java提供了自动装箱和拆箱操作。自动装箱是将基本类型转换成包装类型，自动拆箱是将包装类转换成基本类型。如果没有自动装箱操作，基本类型转包装类型就需要使用构造方法。

```java
Integer a = 1;// 自动装箱
int b = a;// 自动拆箱
```

## 自动拆箱装箱原理

Java编译时会使用自动装箱拆箱的方法，进行类型转换。使用Javap反编译工具可以看出底层也是这样的。

```java
Integer a = Integer.valueOf(1);// 自动装箱
int b = a.intValue(); // 自动拆箱
```

## 自动拆箱装箱问题

### 空指针

在使用包装类时，如果Integer是null，在进行拆箱的时候会导致空指针。

```java
public void method(Integer value) {
    if (value == 0) {
        // 如果这里value是null，那么这里用了value.intValue()，就会报错
	}
}
```



## 包装类型的缓存机制

### 什么是缓存机制

在Java5中，Integer，Byte，Short，Long，Character包装类引入了缓存机制，使用于[-128, 127]之间的数字，Character是[0， 127]。在Java6中Integer可以设置缓存机制的上限，通过虚拟机参数`-XX:AutoBoxCacheMax=`

来设置缓存上线。缓存机制只适用于自动装箱，并且在使用构造函数创建对象时没有缓存机制。

```java 
Integer a = 100;
Integer b = 100;
System.out.println(a == b);// true

Integer c = 300;
Integer d = 300;
System.out.println(c == d);// false
```

### 缓存机制原理

以Integer为例，在Integer的自动装箱中，使用`valueOf`方法将基本类型包装成对象类型，在Integer源码中可以看到如果基本类型[-128, 127]会使用缓存机制。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
      return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

其中low是-128，high默认是127，可以通过`-XX:AutoBoxCacheMax=`设置，其中IntegerCache是一个静态内部类，cache是一个`final static`的变量。在Integer第一次使用的时候这个类便初始化出来，作为缓存。

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
      int h = 127;
      // 获取设定的上限high
      String integerCacheHighPropValue =
        sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
      if (integerCacheHighPropValue != null) {
        try {
          int i = parseInt(integerCacheHighPropValue);
          // 设定的上限和127选择较大的作为上限
          i = Math.max(i, 127);
          // 上限不能超过Integer最大值，这里减去low的原因是cache数组会整体向右移动low
          h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
        }
      }
      high = h;

      cache = new Integer[(high - low) + 1];
      int j = low;
      for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);
      assert IntegerCache.high >= 127;
    }
}
```











