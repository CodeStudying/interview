## 1.数据类型

### 基本类型&包装类型

### 

| 基本类型 |  大小   |      最小值       |                最大值                 | 包装器类型 |
| :------: | :-----: | :---------------: | :-----------------------------------: | :--------: |
| boolean  |    -    |         -         |                   -                   |  Boolean   |
|   char   | 16 bits |     Unicode 0     |       Unicode 2<sup>16</sup>-1        | Character  |
|   byte   | 8 bits  |       -128        |                 +127                  |    Byte    |
|  short   | 16 bits |  -2<sup>15</sup>  |           +2<sup>15</sup>-1           |   Short    |
|   int    | 32 bits |  -2<sup>31</sup>  |           +2<sup>31</sup>-1           |  Integer   |
|   long   | 64 bits |  -2<sup>63</sup>  |           +2<sup>63</sup>-1           |    Long    |
|  float   | 32 bits | 2<sup>-149</sup>  | (2-2<sup>-23</sup>})*2<sup>127</sup>  |   Float    |
|  double  | 64 bits | 2<sup>-1074</sup> | (2-2<sup>-52</sup>})*2<sup>1023</sup> |   Double   |



* Java每种基本类型所占存储空间的大小是不变的，因此比其他大多数语言具有可移植性。
* 所有数值类型都有正负号。

每个包装类的对象可以封装一个相应的基本类型的数据，并提供了相应的方法。包装类对象一经创建，其内容不可改变。基本类型和对应的包装类可以相互转换：

- 由基本类型向对应的包装类转换称为装箱；
- 包装类向对应的基本类型转换称为拆箱。

### 缓存池
当我们使用Integer的时候会存储数据，为避免重复创建对象默认的缓存数据的范围是-128到127，如果超出这个范围则创建一个新的对象。
```java
Integer n1 = 123;
Integer n2 = 123;
Integer n3 = 128;
Integer n4 = 128;
Integer n5 = Integer.valueOf(123);
Integer n6 = Integer.valueOf(123);

System.out.println(n1 == n2); //true
System.out.println(n3 == n4); //false
System.out.println(n5 == n6); //true
```
