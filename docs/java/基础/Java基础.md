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
 
## 4. Object 通用方法

#### 1. clone()

当需要获取到一个对象的拷贝用于某些处理，可以使用Object 类中的 clone() 方法来进行复制。如果该对象的类没有实现 Cloneable ，会抛出 CloneNotSupportedException 异常。由于 clone() 方法是由 protected修饰需要覆写 clone() 方法才能调用。 

- 浅拷贝

  创建一个新对象，然后将当前对象的非静态字段复制到该对象，如果字段类型是基本类型，那么对该字段进行复制；如果字段是引用类型的，则只复制该字段的引用而不复制引用指向的对象，原始对象与新对象里面的引用字段指向的是同一个对象，当修改新对象引用里的内容时，原对象引用的内容也会改变。

- 深拷贝

  为了要在clone对象时进行深拷贝，除了调用父类中的 clone() 方法(super.clone())得到新的对象，还要将该类中的引用变量也 clone 出来。
  
### 2. equals()&hashCode()

equals() 方法用于判断对象的引用是否相等。

equals() 方法的重载需要满足：

- 自反性

  对任何非空引用x，``` x.equals(x)```结果为ture。

- 对称性

  对任何非空引用x和y，如果```x.equals(y)```为true，那么```y.equals(x)```也必须为true。

- 传递性

  对任何非空引用值x、y和z，如果```x.equals(y)```为true，并且```y.equals(z)```为true，那么```x.equals(z)```也必定为true。

- 一致性

  对任何非空引用值x和y，多次调用``` x.equals(y)``` 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改。

hashCode()方法将对象在内存中的地址作为哈希码返回，返回值类型是整形。该方法通常在向哈希表(如HashSet、HashMap等)中添加对象时，通过调用hashCode()方法计算对象的哈希码。

equals() 方法和hashCode()方法关系：

- 如果两个对象equals，他们的hashcode一定相等。
- 如果两个对象不equals，他们的hashcode有可能相等。
- 如果两个对象hashcode相等，他们不一定equals。
- 如果两个对象hashcode不相等，他们一定不equals。

## 5. 继承

**1. 访问权限**

**private**：被其修饰的属性以及方法只能被该类的对象访问，其子类不能访问，也不能跨包访问。

**default**：不加任何访问修饰符，通常称为“默认访问权限“。被其修饰的类、属性以及方法只允许在同一个包中进行访问。

**protected**：被其修饰的属性以及方法只能被类本身的方法及子类访问，即使子类在不同的包中也可以访问。

**public**：被其修饰的类、属性以及方法不仅可以跨类访问，而且允许跨包访问。

只有默认访问权限和 public 能够用来修饰类，修饰类的变量和方法四种权限都可以。

|   权限    | 类内 | 同包 | 不同包子类 | 不同包非子类 |
| :-------: | :--: | :--: | :--------: | :----------: |
|  private  |  √   |  ×   |     ×      |      ×       |
|  default  |  √   |  √   |     ×      |      ×       |
| protected |  √   |  √   |     √      |      ×       |
|  public   |  √   |  √   |     √      |      √       |

Java中的包主要是为了防止类文件命名冲突以及方便进行代码管理。

对于一个Java源代码文件，如果存在 public 类的话，只能有一个 public 类，且此时源代码文件的名称必须和 public 类的名称完全相同；如果还存在其他类，这些类在包外是不可见的。如果源代码文件没有 public 类，则源代码文件的名称可以随意命名。

**2. 抽象类与接口**

- 抽象类

  抽象类和抽象方法都使用abstract进行声明。抽象类一般会包含抽象方法，但是抽象类里面可以没有抽象方法。抽象类和普通类最大的区别是，抽象类不能被实例化，需要继承抽象类才能实例化其子类。

- 接口

  接口是抽象类的延伸，通常以interface来声明。Java 为了安全性而不支持多重继承，一个类只能有一个父类。但是接口不同，一个类可以同时实现多个接口，所以接口弥补了不支持多重继承的缺陷。

  接口中的变量会被隐式的指定为 **public static final** 变量。在传统版本上，接口中的所有方法必须是非静态的，接口中的方法会被隐式的指定为 **public abstract**。但在java版本1.8中，可以通过默认方法或者静态方法加入方法体。

- 抽象类和接口的对比

  |    参数    |                          抽象类                           |                             接口                             |
  | :--------: | :-------------------------------------------------------: | :----------------------------------------------------------: |
  |  方法实现  |                      可以有方法实现                       | 在传统版本中不存在方法实现，<br/>在1.8版本中可以通过static或者<br/>default修饰来实现方法 |
  |    实现    |             子类使用extends关键字来继承抽象类             |              子类使用implements关键字来实现接口              |
  |  构造方法  |                  抽象类允许定义构造方法                   |                    接口中不能定义构造方法                    |
  | 访问修饰符 | 抽象方法可以由public、protected、<br/>和default修饰符修饰 |                    接口修饰符默认是public                    |
  |    继承    |                     抽象类不能多继承                      |                        接口可以多继承                        |
  |  初始化块  |             抽象类允许定义静态或实例初始化块              |              接口中不允许定义静态或实例初始化块              |
  |  成员变量  |   既可以是变量，也可以是常量，<br/>访问权限和普通类相同   |     默认是public static final，<br/>访问权限默认是public     |

  抽象方法都要被实现，所以抽象方法不能由 static 修饰，也不能由 private 修饰。

  

## 参考

- Java 编程思想[M]. Eckel B. 机械工业出版社, 2002.
