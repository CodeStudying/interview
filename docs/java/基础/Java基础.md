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


- Java每种基本类型所占存储空间的大小是不变的，因此比其他大多数语言具有可移植性。
- 所有数值类型都有正负号。

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

## 2. final

**1. 修饰数据**

向编译器声明一块数据是恒定不变的，可以是永不改变的编译时常量，也可以是在运行时被初始化而不希望被改变的值。

对于基本类型，final使数值不变；对于引用对象，final使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

编译器要确保final修饰的数据在使用前必须进行初始化，对于final修饰的成员变量赋值，可以在声明该成员时赋值，也可以在在构造方法中赋值(先声明，后赋值，可保证一个类中的final域根据对象而有所不同)。

**2. 修饰方法**

确保在继承中使方法行为保持不变且不会被子类覆盖。

private方法隐式地指定为final的。如果在子类中定义的方法和基类中的一个private方法签名相同，此时子类的方法不是覆盖基类方法，而是生成了一个新的方法。

**3. 修饰类**

声明类不允许被继承，final类中的所有方法都不能覆盖。

## 3. static

**1. 静态变量**

静态变量在内存中只存在一份，在类加载时被创建，同时类所有的实例都共享静态变量，可以直接通过类名来访问它(类名.变量名)。

实例变量每创建一个实例就会产生一个实例变量，它与该实例生命周期相同。

**2. 静态方法**

静态方法在类加载的时候就存在了，它不依赖于任何实例，所以 static 方法必须实现，即不能由abstract修饰。

- 仅能调用其他的static方法。
- 只能访问static数据，但在静态成员函数中可以声明它自身的变量。
- 不能以任何方式引用this 或super。

**3. 静态代码块**

静态代码块和静态变量一样在类加载时化时初始化一次。

**4. 初始化顺序**

- 静态数据优先于其它数据的初始化，静态变量和静态代码块哪个先运行取决于它们在代码中的顺序。
- 实例变量和普通代码块的初始化在静态变量和静态代码块初始化结束之后。
- 最后是构造函数中的数据进行初始化。

存在继承的情况下，初始化顺序为:

  1. 父类(静态变量、静态初始化块)
  2. 子类(静态变量、静态初始化块)
  3. 父类(变量、初始化块)
  4. 父类(构造器)
  5. 子类(变量、初始化块)
  6. 子类(构造器)

## 参考

- Java 编程思想[M]. Eckel B. 机械工业出版社, 2002.
