## 1.数据类型

### 基本类型&包装类型

### 

| 基本类型 |  大小   |      最小值       |                   最大值                    | 包装器类型 |
| :------: | :-----: | :---------------: | :-----------------------------------------: | :--------: |
| boolean  |    -    |         -         |                      -                      |  Boolean   |
|   char   | 16 bits |     Unicode 0     |          Unicode 2<sup>16</sup>-1           | Character  |
|   byte   | 8 bits  |       -128        |                    +127                     |    Byte    |
|  short   | 16 bits |  -2<sup>15</sup>  |              +2<sup>15</sup>-1              |   Short    |
|   int    | 32 bits |  -2<sup>31</sup>  |              +2<sup>31</sup>-1              |  Integer   |
|   long   | 64 bits |  -2<sup>63</sup>  |              +2<sup>63</sup>-1              |    Long    |
|  float   | 32 bits | 2<sup>-149</sup>  | (2-2<sup>-23</sup>)&middot;2<sup>127</sup>  |   Float    |
|  double  | 64 bits | 2<sup>-1074</sup> | (2-2<sup>-52</sup>)&middot;2<sup>1023</sup> |   Double   |


* Java每种基本类型所占存储空间的大小是不变的，因此比其他大多数语言具有可移植性。
* 所有数值类型都有正负号。

每个包装类的对象可以封装一个相应的基本类型的数据，并提供了相应的方法。包装类对象一经创建，其内容不可改变。基本类型和对应的包装类可以相互转换：

- 由基本类型向对应的包装类转换称为装箱；
- 包装类向对应的基本类型转换称为拆箱。

### 缓存池
当我们使用 Integer 的时候会存储数据，为避免重复创建对象包装类提供了一定范围的缓存池，如果超出这个范围则创建一个新的对象。
```java
Integer n1 = 123; // 装箱
Integer n2 = 123;
Integer n3 = 128;
Integer n4 = 128;
Integer n5 = Integer.valueOf(123);
Integer n6 = Integer.valueOf(123);

System.out.println(n1 == n2); // true
System.out.println(n3 == n4); // false
System.out.println(n5 == n6); // true
```
Java 自动装箱相当于调用 valueOf 方法，其中 valueOf 源码如下：
```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
可以看到如果这个数值在cache数组的范围内（low和high之间），就返回cache数组的中的数据，否则创建一个Integer。其源码如下：
```java
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```
默认的范围是-128～127，可以通过-XX:AutoBoxCacheMax=\<size\>参数去修改最大值，在JVM初始化的时候，这个值被写入sun.misc.VM class系统私有配置文件中并加载。

## 参考

- Java 编程思想[M]. Eckel B. 机械工业出版社, 2002.
