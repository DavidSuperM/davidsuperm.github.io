# JVM

源码经过编译后变成class字节码文件。jvm再加载class文件。

### 内存简图（Hotspot）
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_1_jvm.png){:height="100px" width="100px"}
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_2_jvm.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_3_jvm_load.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_4_jvm_link.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_5_jvm_init.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_6_jvm_data.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_7_jvm_detail.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_8_jvm_thread.png)

二进制文件->加载器->运行时数据区->执行引擎

运行时方法区由5部分构成
线程共享的：
堆： 存放创建的对象     存在error，存在gc
方法区 ：存放类型信息，运行时常量池        存在error，存在gc,也存在垃圾回收 

线程不共享：
java栈（虚拟机栈）   存在error，不存在GC，存在OOM
本地方法栈           存在error，不存在GC，存在OOM
程序计数器(PC寄存器) ：用来存储指向下一条指令的地址，也即将要执行的指令代码。由执行引擎读取下一条指令。不存在error，不存在GC，也不存在OOM

加载器内部过程：
加载->验证->准备->解析->初始化

# new一个对象发生了什么
（下面的是简版）
1. 加载。加载类信息到方法区
2. 验证。验证格式，语义，(文件规范，final有没有子类）
3. 准备。为静态变量分配内存空间
4. 解析
5. 初始。为静态变量赋值，执行static代码块
6. 创建对象。堆区为对象分配内存。执行初始化代码

本地方法栈使用的是本地方法接口，本地方法库是为了调用本地的一些c代码写的接口。

虚拟机栈：
由于跨平台的设计，java指令都是根据栈来设计的，不同平台CPU架构不同，所以不能设计为基于寄存器。用虚拟机栈优点是跨平台，指令集小，编译器容易实现。
缺点是性能下降，使用同样的功能需要更多的指令。

栈：解决程序的运行问题，保存方法的局部变量，部分结果，参与方法的调用。访问速度仅词语程序计数器。每个方法执行伴随着进栈，执行结束会出栈。
对于栈来说不存在垃圾回收问题，但是存在OOM问题（可能会报StackOverflowError,或者栈动态扩展申请不到足够的内存时会报OutOfMemoryError）

栈内部结构以栈针为单位，一个栈针对应一个方法，栈针内部又有局部变量等信息，详见图9
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_9_stack.png)


本地方法栈和虚拟机栈其实原理一样，只不过调的方法不一样。本地方法栈调的是本地类库的c的方法等等，虚拟机栈调的是代码里java的方法




### 双亲委派机制
概念：java虚拟机对class文件采用按需加载的方式，即需要使用该类时才会将它的class文件加载到内存生成class对象。而且加载某个类的class文件时，java虚拟机采用的双亲委派模式，即把请求交由父类处理，它是一种任务委派模式。

原理：
1. 如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行。
2. 如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器
3. 如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式。

子类加载器和父类加载器不是继承关系，而是类似上下级关系。
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/20210611_3_shuang_qin.png)

优点：1. 避免类的重复加载
2. 保护程序安全，防止核心API被随意篡改

什么是本地方法 
见图10
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_10_jvm_native.png)

# 堆
-X 是jvm的运行参数，ms是memory  start  mx是memory max
-Xms10m  初始堆空间10兆（新生代+老年代）
-Xmx10m  最大堆空间10兆 

默认堆空间大小是电脑内存 1/64,最大堆空间是1/4

开发中建议初始堆内存和堆最大内存设置成相同的值，因为不同的话堆可能频繁会扩容和缩小。
  
- 堆中有块空间叫缓存区，是线程私有的，因为完全线程共享，存在线程安全问题，所以划分一块线程私有的缓存区。
- 方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除，因为如果马上移除的话频率会很高，影响用户线程执行。
- 堆是GC（Garbage Collection，垃圾收集器）执行垃圾回收的重点区域
- 栈中不存放对象实例，存放的是指向对象实例的引用

堆空间分为年轻代和老年代，年轻代又分为Eden，Survivor0，Survivor1 空间 
调整新生代老年代占比，见图11，默认1：2 
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_11_jvm_new_ratio.png)
默认情况下 eden:s0:s1 = 8:1:1
几乎所有的java对象都是在eden区被new出来的，绝大部分java对象在新生代销毁，
-Xmn 设置新生代空间大小（如果和之前比例设置的矛盾，以这个为准）

### 对象分配过程
参考<https://www.jianshu.com/p/53ceabaa2175>
1. new的对象先放eden区，此区由大小限制
2。 当eden填满，程序又要创建对象，gc将对eden进行垃圾回收。将eden的剩余对象移动到s0区（ survivor0，幸存者0区） （这时s1和eden是空的），
然后将新对象放入eden。（如果新对象还是放不下，eden为空都放不下说明超大，s0，s1肯定也放不下，所以直接放到老年代区，如果老年代也放不下，就会对老年代执行full gc，也叫major jc，然后再放，再放不下就会oom）
4. 如果再次触发垃圾回收，s0区的没有被回收的，将放到s1区，eden中的没有被回收的放到s1区（这时s0是空的）（如果s1放不下eden过来的对象，则直接放入老年代）
5. 如果再经历垃圾回收，此时会重新放回s0区，接着再放去s1区 （s0，s1总有一个是空的）
6. 啥时去老年区？可以设置次数（移动一次就算1次），默认是15次 （-XX:MaxTenuringThreshold=15进行设置）
7. 当养老区内存不足时，再次出发GC，对养老区内存清理
8。 若养老区执行了Major gc 后仍无法进行对象保存，就会产生OOM异常

- eden满了会出发ygc（young gc），但是s0，s1满了，不会出发gc。因为eden满了触发gc，会将eden和s0，s1一起进行垃圾回收。

### Minor GC, Major GC, Full GC
对eden进行回收的young gc就是Minor GC
针对老年代的gc叫Major GC
Full GC就是对整个java堆和方法区的垃圾回收
Major GC的速度一般比Minor GC慢10倍以上。
出现Major GC，经常会伴随至少一次的Minor GC（不绝对，有些收集器策略不同）
Full GC触发机制：
- 调用System.gc()，系统建议Full GC，但是不必然执行
- 老年代空间不足
- 方法区空间不足
- 通过MinorGC后进入老年代的平均大小大于老年代的可用内存
- 由eden区，from区向to区复制时，对象大小大于to区的可用内存，则对象转到老年代，且老年代的可用内存小于该对象的大小（s0向s1转移时，则s0被称为from区，s1被称为to区，反之亦然）

### 为什么要把java堆分代
为了优化GC性能

### TLAB（Thread LOcal Allocation Buffer）
为什么要有TLAB
因为堆区是线程共享的，对象实例创建频繁，在并发环境下从堆区划分内存空间是线程不安全的，为避免多个线程操作同一地址，如果使用加锁等机制，这样会影响分配速度。
所以对Eden区域继续进行划分，JVM为每个线程分配一个私有的缓存区域，它包含在Eden空间内。多线程同时分配内存时，使用TLAB可以避免非线程安全问题，提升内存分配的吞吐量。（线程先用TLAB的空间，用完了再用Eden其他地方的空间）
这种内存分配方式被称为 快速分配策略。
因为TLAB空间小，也并不是所有的对象实例都能成功的在TLAB中成功分配内存。默认TLAB仅占用Eden空间的1%。一旦对象在TLAB中分配失败，则会使用加锁的方式在Eden中分配空间，对象实例化后会将引用入栈。

### 堆空间参数设置
|  参数设置   | 说明  |
|  ----  | ----  |
| -XX:PrintFlagsInitial  | 查看所有参数的默认初始值 |
| -XX:+PrintFlagsFinal  | 查看所有参数的最终值 |
| -Xms  | 堆初始大小（默认为物理内存的1/64） |
| -Xmx  | 堆最大空间内存(默认物理内存的1/4) |
| -Xmn  | 新生代大小 |
| -XX:NewRatio  | 配置新生代与老年代在堆结构的占比 |
| -XX：SurvivorRatio  | 设置新生代中Eden和s0/s1空间的比列 |
| -XX：MaxTenuringThreshold  | 设置新生代垃圾的最大年龄 |
| -XX:+PrintGCDetails  | 单元格 |
| 单元格单元格  | 单元格 |
| 单元格单元格  | 单元格 |

建一个空的java程序，main方法里面是空，然后java程序的configurations的 VM options选项设置上面的参数，然后运行，就能看到信息。



# 方法区
方法区主要存放的是类型信息（类，接口，枚举等信息）。运行时常量池（常量），静态变量，即时编译器编译后的代码缓存
方法区在JVM启动的时候被创建。
方法区的大小可以选择固定和可扩展。
如果系统定义了太多类，方法区放不下，虚拟机会抛出OOM错误。
JVM为每一个已加载的类型（类或接口）都维护一个
### 栈，堆，方法区的交互关系
见图12
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_12_jvm_relationship.png)
### 方法区参数设置
略
### 为什么需要常量池

### 方法区的演进
jdk1.6及以前，有永久代，静态变量存放在永久代上
jdk1.7，有永久代，但是字符串常量池，静态变量保存在堆中
jdk1.8及之后，无永久代，类型信息，常量等保存在本地内存的元空间，字符串常量池、静态变量仍在堆中。
### 为什么取消永久代
1. 因为为永久代设置空间大小很难确定，容易OOM，而元空间不在虚拟机中，使用的是本地内存，元空间大小仅受本地内存限制。
2. 对永久代调优比较困难

### 符号引用和直接引用
符号引用即用用字符串符号的形式来表示引用，其实被引用的类、方法或者变量还没有被加载到内存中。而直接引用则是有具体引用地址的指针，被引用的类、方法或者变量已经被加载到内存中。

那为什么要用符号引用呢？这是因为类加载之前，javac会将源代码编译成.class文件，这个时候javac是不知道被编译的类中所引用的类、方法或者变量他们的引用地址在哪里，所以只能用符号引用来表示。
比如org.simple.People类引用了org.simple.Language类，在编译时People类并不知道Language类的实际内存地址，因此只能使用符号org.simple.Language（假设是这个，当然实际中是由类似于CONSTANT_Class_info的常量来表示的）来表示Language类的地址。

符号引用要转换成直接引用才有效

### 字符串常量池，运行时常量池，class常量池
class常量池：java文件被编译成 class文件，class文件中除了包含类的版本、字段、方法、接口等描述信息外，还有一项就是常量池（Constant Pool），用于存放编译器生成的各种字面量（ Literal ）和 符号引用（Symbolic References）。

字符串常量池：是在类加载完成，经过验证，准备阶段之后 在 堆 中生成字符串对象实例，然后 将该字符串对象实例的 引用值 存到 字符串常量池中（String Pool ）中。
String Pool 中存的是 引用值，而不是具体的实例对象，具体的实例对象是在堆中开辟的一块空间存放的。
在 HotSpot VM 里实现的 String Pool 功能的是一个 StringTable 类，它是一个哈希表，里面存的是 驻留字符（ 也就是用双引号括起来的部分）的 引用（而不是驻留字符串实例本身），也就是说在堆中的某些字符串实例被这个 StringTable 引用之后就等同被赋予了”驻留字符串”的身份。这个StringTable在每个 HotSpot VM 的实例只有一份，被所有的类共享。

运行时常量池：jvm在执行某个类的时候，必须经过加载、连接、初始化，而连接又包括验证、准备、解析三个阶段。而当类加载到内存中后，jvm就会将 class常量池 中的内容存放到 运行时常量池 中，由此可知，运行时常量池 也是每个类都有一个。
在上面我也说了，class常量池 中存的是字面量和符号引用，也就是说他们存的并不是对象的实例，而是对象的符号引用值。而经过解析（resolve）之后，也就是把符号引用替换为直接引用，解析的过程会去查询 字符串常量池 ，也就是我们上面所说的StringTable，以保证 运行时常量池所 引用的字符串与 字符串常量池 中所引用的是一致的。

### 题
```
public class StringExer{
    String str = new String("good");
    char[] ch = {'t', 'e'};
    public void change(Stirng str, char ch[]){
        str = "test"
        ch[0]='b';
    }
    public static void main(String[] args){
        StringExer ex = new StringExer();
        ex.change(ex.str,ex.ch)
        System.out.println(ex.str)  // good 正常的引用传递会改变指向的方向，String比较特殊，不可变性，
        System.out.println(ex.ch)   // be
    }
}

String s1 = "a"+"b"+"c";   // 编译生成字节码文件时，将s1等同于"abc",在常量池中生成"abc",返回引用给s1
String s2 = "abc"         // 将常量池中已有的"abc"的引用给s2
System.out.println(s1==s2); // true

String s1 = "a" ;
String s2 = "b";
String s3 = "ab";
String s4 = "a"+"b";
String s5 = s1 + "b";
String s6 = "a" + s2;
String s7 =  s1 + s2;

System.out.println(s3 == s4); // true
System.out.println(s3 == s5); // false   只要其中有一个变量，那就是在堆中生成"ab"实例，相当于在堆中 new String() 拼接的原理是StringBuilder
System.out.println(s3 == s6); // false
System.out.println(s3 == s7); // false
System.out.println(s5 == s6); // false
System.out.println(s6 == s7); // false      // s5,s6,s6分别在堆中生成实例,引用互不相等

String s8 = s6.intern();
System.out.println(s3 == s8); // true

```

### String的intern方法
String的intern()方法就是扩充常量池的一个方法；当一个String实例str调用intern()方法时，Java查找常量池中是否有相同Unicode的字符串常量，如果有，则返回其的引用，如果没有，则在常量池中增加一个Unicode等于str的字符串并返回它的引用；
举例
```
String s0= “kvill”; 
String s1=new String(”kvill”); 
String s2=new String(“kvill”); 
System.out.println( s0==s1 );     // false
System.out.println( “**********” ); 
s1.intern(); 
s2=s2.intern(); //把常量池中“kvill”的引用赋给s2 
System.out.println( s0==s1);         // false
System.out.println( s0==s1.intern() );     // true
System.out.println( s0==s2 );       // true
```

### new String()到底创建了几个对象

# 垃圾回收
### 为什么需要GC
1. 不垃圾回收的，内存会被消耗完。
2. 垃圾回收还会进行碎片整理，腾出空的内存给新的大对象
3. 没有GC就不能保证应用程序的正常进行。

### 垃圾回收算法
1. 标记阶段：引用计数算法
2. 标记阶段：可达性分析算法
3. 对象的finalization机制
4. MAT与JProfiler的GC Roots溯源
5. 清除阶段：标记-清除算法
6. 清除阶段：复制算法
7. 清除阶段：标记-压缩算法
8. 小结
9. 分代收集算法
10 增量收集算法、分区算法

### 引用计数算法
每个对象保存一个整型的引用计数器属性，记录对象呗引用的情况。对象被引用一次，则计数器加1，引用失效则计数器减1，当计数器值为0，则表示对象不可能再被使用，可进行回收。
优点：实现简单，垃圾对象便于辨识；判定效率高，回收没有延迟性。
缺点：它需要单独的字段存储计数器，增加了存储空间的开销。
每次赋值都需要更新计数器，伴随加法和减法操作，增加时间开销。
无法处理循环引用的情况，这是致命缺陷，导致在Java的垃圾回收器中没有使用这类算法。

### 可达性分析
与引用计数相比，可达性分析算法同样具备实现简单，执行高效等特点，还能有效解决引用计数算法中的循环引用问题，防止内存泄漏

可达性分析算法是以根对象集合（GC Roots）为起始点，按照从上至下的方式搜索被根对象集合所连接的目标对象是否可达。如果对象没有引用链相连则是不可达，可以被标记为垃圾对象。

##### GC Roots包括以下几类元素
虚拟机栈中引用的对象
本地方法栈中 JNI（即一般说的 Native 方法）引用的对象
方法区中类静态属性引用的对象
方法区中常量引用的对象
所有被同步锁synchronized持有的对象
Java虚拟机内部的引用

### 对象的finalization机制
放垃圾回收器发现没有引用指向一个对象，即垃圾回收此对象之前，总会先调用这个对象的finalize()方法，该方法允许在子类重写，用于在对象回收时进行资源释放。比如关闭文件、数据库连接。
永远不要主动调用finalize()方法，应该交给垃圾回收机制调用。

### 判断对象是否真正的能被回收，需要经历两次标记过程
1. 对象先进行可达性分析，如果没有发现与GC Roots相连接的引用链，此时该对象将会被第一次标记。
2. 接着还要做一次筛选，筛选的条件是此对象是否有必要执行finalize()方法，因为finalize()方法是拯救对象的最后一次机会，所以只有对象没有覆盖finalize()方法或者即使覆盖了也没有拯救自己的意思，或者finalize()方法已经被虚拟机调用过，才能真正宣告一个对象死亡，能够被垃圾回收器回收。
参考<http://huhansi.com/2020/03/28/Java/JVM/2020-03-28-Java-%E5%A6%82%E4%BD%95%E5%88%A4%E6%96%AD%E5%AF%B9%E8%B1%A1%E6%98%AF%E5%90%A6%E5%8F%AF%E4%BB%A5%E5%9B%9E%E6%94%B6/>

### 标记-清除算法
标记：垃圾回收器从根节点开始遍历，标记所有被引用的对象。一般是在对象的header中记录为可达对象（其实就是可达性分析算法来标记）
清除：垃圾回收器对堆内存从头到尾进行线性遍历，如果发现某个对象在其header中没有标记为可达对象，则将其回收。
（清楚并不是真的置空，而是把需要清楚的对象地址保存在空闲列表，下次有对象要放的时候就放进来。）

优点：容易理解
缺点：效率不算高。GC时需要停止整个应用程勋。清理出来的空闲内存时不连续的，内存碎片需要维护一个空闲列表。

### 复制算法
概念：将活着的内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后清楚正在使用的内存块的所有对象，交换两个内存的角色，最后完成垃圾回收。
优点：没有标记清除过程，实现简单，运行高效。
复制过去后保证了空间的连续性，不会出现碎片问题
缺点：需要两倍的内存空间。
对于G1这种分拆成大量的region的GC，复制而不是移动，需要维护region之间对象的引用关系，内存占用和时间开销也不小。
适用于存活对象少，垃圾多的情况

### 标记-压缩算法（标记-整理）
概念：第一阶段和标记清除算法一样，从根节点开始标记所有被引用的对象。第二阶段将所有的存活对象压缩到内存的一端，按顺序排放，之后清理边界外所有的空间。
优点: 消除了标记清除算法中的内存区域分散的缺点。消除了复制算法中内存减半的高额代价
缺点：标记-压缩算法效率低于复制算法
移动对象的同时，如果对象呗其他对象引用，还需要调整引用的地址。
移动过程中需要暂停用户应用程序，即STW

### 分代收集算法
目前几乎所有的GC都是采用分代收集算法执行垃圾回收的
年轻代的特点：区域相对老年代较小，生命周期短，存活率低，回收频繁。适合复制算法。而复制算法的内存利用率不高的问题，通过hotspot的两个survivor的设计得到缓解。
老年代：区域较大，对象生命周期长，存活率高，回收不频繁。一般是由标记-清除或者标记-整理的混合实现。 

### 增量收集算法
概念:如果一次性对所有的垃圾进行处理，会造成系统长时间的停顿，我们可以让垃圾收集线程和应用程序线程交替执行。每次，垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次往复，知道垃圾收集完成。
缺点：因为线程切换和上下文转换的消耗，使得垃圾回收的总成本上升，造成系统的吞吐量下降。

### 分区算法
概念：将一块大的内存区域分割成多个小块，根据目标的停顿时间，每次合理的回收若干个小区间，而不是整个堆空间，从而减少一次GC所产生的停顿。

### System.gc()
System.gc() （内部实际调的就是Runtime.getRuntime().gc() ）或者Runtime.getRuntime().gc()的调用，会显式触发Full GC，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。
然而System.gc()调用附带一个免责声明，无法保证对垃圾收集器的调用（就是System.gc()会提醒垃圾回收器进行垃圾收集，但是到底收不收集不一定，由GC决定。）

### 内存溢出
没有空闲内存，并且垃圾收集器也无法提供更多的内存。
### 内存泄漏
对象不被程序用到，但是GC又不能回收他们。

#### 内存泄漏的例子
1. 单例模式
单例的生命周期和应用程序一样长，所以单例模式中，如果持有对外部对象的引用的话，那么这个外部对象是不能被回收的，导致内存泄漏
2，一些体用close的资源未关闭
数据库连接(dataSource.getConnection())，网络连接(socket)和io连接必须手动close，否则是不能被回收的。


## 强引用、软引用、弱引用、虚引用 有什么区别
目标：我们希望能描述这样一类对象，当内存空间还足够时，则能保留在内存中，当内存空间进行垃圾回收后还是很紧张，则可以抛弃这些对象。
这4种引用强度一次减弱

### 强引用
（不回收）
最传统的"引用"定义，指在程序代码中普遍存在的引用赋值，类似 Object obj = new Object() 这种引用关系，无论任何情况，只要强引用关系存在，垃圾收集器就永远不会回收掉被引用的对象。

### 软引用
（内存不足即回收）
在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收（即软引用的对象会被回收掉）。如果这次回收还没有足够的内存，才会抛出内存溢出异常。
使用场景：高速缓存
通过SoftReference类来实现

### 弱引用
（发现即回收）
被弱引用关联的对象只能生存到下一次垃圾收集之前，当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。
使用场景：可有可无的缓存数据
通过WeakReference类来实现

### 虚引用
（对象回收跟踪）
一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得一个对象的实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。
通过PhantomReference类来实现

# 垃圾回收器
### Serial回收器：串行回收（串行指的是垃圾回收的线程）
第一款GC。jdk1.3前的唯一选择
采用复制算法、串行回收和 STW 机制执行内存回收

优点：简单而高效
适合单CPU，内存不大的情况

见图14
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_14_jvm_serial.png)

### Serial Old
针对老年代执行收集。串行回收，stw机制，内存回收算法使用标记-压缩算法 

### ParNew回收器：并行回收
是Serial回收器的多线程版本。除了并行回收外其他和Serial几乎没区别。
采用复制算法，STW
见图15
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_15_jvm_parnew.png)

### Parallel Scavenge 回收器：吞吐量优先
复制算法、并行回收，STW，
吞吐量优先，自适应调节策略
这种高吞吐量的适合在后台运算不需要太多交互的任务。例如批量处理、订单处理、工资支付、科学计算等
JDK8默认GC
见图16
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_16_jvm_Parallel.png)

### Parallel Old
标记-压缩，并行回收，STW

### CMS回收器：低延迟
关注点是尽可能缩短垃圾收集时用户线程的停顿时间
并发（回收时不用暂停用户线程）
标记-清除算法，STW （为什么不使用标记-压缩算法，因为有用户线程同步在执行 ）
见图17
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_17_jvm_cms.png)
CMS垃圾回收这个过程分4个主要阶段，即初始标记、并发标记、彩虹精心标记、并发清除。
初始标记：公祖哦线程会因为STW出现短暂暂停，这个阶段主要任务仅仅是标记出GC Roots能直接关联到的对象，因为直接关联对象比较小，所以这个阶段速度非常快。
并发标记：从GC Roots直接关联对象开会遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行。
重新标记：由于在并发标记阶段，程序的工作线程会和垃圾收集线程同时运行或交叉运行，因此为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间会比初始标记阶段稍长，但是远比并发标记阶段时间短。也会STW
并发清除：此阶段清除删除掉标记阶段判断的已经死亡的对象，释放内存空间。由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发。

因为垃圾收集阶段用户线程没有中断，所以CMS回收过程中要保证用户线程有足够的内存可用，因此CMS不能像其他收集器那样等到老年代几乎被完全被填满了再进行收集，而是当堆内存使用率达到某一阈值时，便开始进行回收。
如果CMS运行期间预留内存无法满足程序需要，就会临时启用Serial Old收集器来重新进行老年代垃圾收集。

优点：并发收集，低延迟
缺点：会产生内存碎片。（无法分配大对象，提前触发Full GC）
对CPU资源敏感，并发阶段因为占用一部分线程而导致应用程序变慢，吞吐量会降低
无法处理浮动垃圾（并发阶段产生新的垃圾，无法对这些垃圾对象标记，只能下次回收）

### G1回收器：区域化分代式
目标：延迟可控的情况下获得尽可能高的吞吐量。
并行回收器，把堆内存分成很多不想关的区域（region）（物理上不连续）。使用不同的region来表示eden，s0,s1,老年代等。
G1 GC有计划的避免在整个Java堆中进行全区域垃圾收集。G1跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需要时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region。
适用于多核CPU及大容量内存的机器。 
见图18
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_18_jvm_G1.png)
H是hunongous区域，用于存储大对象，如果超过1.5个region，就放入H。如果1个H装不下大对象，G1会寻找连续的H来存储

##### 特点
并行性：G1回收期间，可以有多个GC线程同时工作，此时用户线程STW
并发性：部分工作可以和应用程序同时进行
分代收集：G1属于分代性垃圾回收器，区分年轻代，老年代。年轻代依然有Eden区和Survivro区，不要求整个Eden区，年轻代或者老年代是连续的，也不再坚持固定大小和数量。
将对空间分为若干个区域，包含了逻辑上的年轻代，老年代。

可预测的停顿时间模型：即能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。

#### 空间整合
内存回收以region作为基本单位，Region之间是复制算法，整体上实际可看做是标记-压缩算法。可以避免内存碎片。

#### 缺点
内存占用和程序运行时的额外执行负载比CMS高
经验上来说，小内存应用上CMS表现大概率优于G1，大内存上G1更好，平衡点在6-8GB之间

#### 使用场景
- 面对服务端应用，针对大内存、多处理器的机器
- 最主要的应用是需要低GC延迟，并具有大堆的应用程序提供解决方案（在堆大小约6GB或更大时，可预测的暂停时间可用低于0.5秒） 

#### G1垃圾回收过程
见图19  
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_19_jvm_G1_GC.png)
图20,21，22，23，24，25        
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_20_jvm_g1_gc2.png)      
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_21_jvm_g1_gc3.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_22_jvm_g1_gc.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_23_jvm_g1_gc.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_24_jvm_g1_gc.png)
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_25_jvm_g1_gc.png)

#### RS(Remembered Set )
有可能eden的对象在old的region中引用了，所以每个region有一个rs，记录这个region中的对象被哪个region引用了，gc时只要根据rs去清理对应的region即可。避免GC时全局扫描。

# 7款GC总结
见图26
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_26_jvm_summary.png)



# 
串行回收器：Serial，Serial Old
并行回收器：ParNew，Parallel Scavenge，Parallel Old
并发回收器：CMS，G1  （并发指的是GC线程可以和用户线程同时执行，不阻塞用户线程。）

## 评估GC的性能指标
- 吞吐量（重要）：运行用户代码的时间占总运行时间的比例（总运行时间=程序运行时间+内存回收时间）
- 垃圾收集开销：吞吐量的补数，垃圾收集所用时间与总运行时间的比例
- 暂停时间（重要）：执行垃圾收集时，程序的工作线程被暂停的时间
- 收集频率：相对于应用程序的执行，收集操作发生的频率
- 内存占用（重要）：Java堆区所占内存大小
- 快速：一个对象从诞生到被回收所经历的时间

高吞吐量和低暂停时间是矛盾的，因为上下文线程切换需要时间，如果想要低暂停时间，则必然多次短暂暂停，多次线程切换，浪费了时间，就不能高吞吐。
所以注重速度就要高吞吐，注重用户感受就低暂停时间
现在标准：在最大吞吐量优先的情况下，降低停顿时间

## 垃圾回收器的组合关系
见图13
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/jvm/20210607_13_jvm_gc_relation.png)
红色的线在jdk8以前是实线，jdk8中取消了红色的线。不建议这么搭配，但是也能用，9中是完全移除红色线关系，不能用这个搭配。
绿色的是jdk14弃用了这个关系
CMS GC 在jdk14中删除了

## 默认GC
jdk8中默认GC是Parallel。新生代使用 Parallel Scavenge 自动触发老年代使用 Parallel Old
jdk8中默认GC是G1。


# 程序计数器（PC寄存器）面试题
### 使用PC寄存器存储字节码指令地址有什么用
因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行

# 栈面试题
### 举例栈溢出的情况（StackOverflowError）
假如方法嵌套调用很多，每次嵌套都会将方法压栈，栈空间不足就会报溢出的错。通过-Xss设置栈的大小。
### 调整栈大小，能保证不出现溢出吗
不能。调大了栈大小，但是代码递归的次数不确定。 
### 分配的栈内存越大越好吗
不是，影响其他地方的内存分配

# 堆面试题
### 堆是分配对象存储的唯一选择吗
不是。有一种特殊情况。如果经过逃逸分析后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配。

逃逸分析的基本行为就是分析对象动态作用域：
- 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
- 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中，或者是将方法内的对象return出去。

# 对象头包含什么信息

# JVM内存模型
1. 线程共享的：方法区，堆
2. 线程私有的，java栈，本地方法栈，程序计数器

堆：几乎所有的对象实例都在这里分配内存。
方法区：它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据
java栈：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息
本地方法栈：本地方法栈则是为虚拟机使用到的Native方法服务
程序计数器：当前线程所执行的字节码的行号指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。
