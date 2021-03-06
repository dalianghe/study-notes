# <center>JVM</center>

### - JVM相关概念
![](http://i.imgur.com/BjS0UG6.png)
### - 内存结构

	程序计数器
		当前线程所执行的字节码的行号指示器（线程私有）。
		如果线程正在执行的是一个java方法，计数器记录的是正在执行的虚拟机字节码的地址；如果线程正在执行的是Native方法，计数器的值为空（Undefind）。
		此区域是java虚拟机规范中唯一没有规定任何OutOfMemoryError情况的区域。
		
	JVM虚拟机栈	
		虚拟机栈描述的java方法执行的内存模型（线程私有），每个方法在执行的同时都会创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。
		局部变量表存放编译期可知的各种基本数据类型、引用类型和returnAddress类型。
		局部变量表所需的内存空间在编译期完成分配。
		java虚拟机规范，对该区域规定了两种异常状况：
		1、线程请求的深度大于虚拟机所允许的深度，抛出StackOverflowError异常；
		2、如果扩展时无法申请到足够的内存，抛出OutOfMemoryError异常。

	本地方法栈
		与虚拟机栈作用相似，区别是虚拟机栈为虚拟机执行java方法服务，本地方法栈为虚拟机使用到的Native方法服务。
		虚拟机规范没有强制规定本地方法栈中方法使用的语音、使用方式与数据结构，具体虚拟机可自由实现。
		异常与虚拟机栈相同有两种：
		1、StackOverflowError
		2、OutOfMemoryError

	方法区
		线程共享、用于存储被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。
		该区域内存回收目标主要是针对常量池的回收和类型的卸载。
		java虚拟机规定，方法区无法满足内存分配需求时，抛出OutOfMemoryError异常。
	
	运行时常量池
		方法区的一部分，存放Class文件常量池中编译期生成的各种字面量和符合引用，在类加载后进入方法区的运行时常量池中。
		虚拟机规范未规定任何细节要求，提供商可自行实现。
		运行时常量池相对于Class文件常量池的一个重要特征是具备动态性，即java语言未要求常量必须在编译期产生，运行期间也可能将新的常量放入池中。
		无法再申请内存时抛出OutOfMemoryError异常。

	java堆
		java虚拟机管理内存的最大一块，所有线程共享，虚拟机启动创建，存放对象实例的一块内存空间。
		从内存回收的角度，堆可分为：新生代和老年代，再细化一点有Eden区、From Survivor和To Survivor等。
		从内存分配的角度，线程共享的java堆，可划分出多个线程私有的本地内存分配缓存（Thread Local Allocation Buffer）TLAB。
		虚拟机规范规定，java堆可以处在物理上不连续的内存空间，只要逻辑上连续即可。
		通过参数 -Xmx和-Xms控制。
		内存空间无法完成分配，并且无法扩展时，抛出OutOfMemoryError异常。

### - 对象的创建、布局和访问

对象创建
	
	虚拟机遇到new指令，首先检查指令的参数是否能在常量池中定位一个类的符号引用，并检查这个符号引用代表的类是否已加载、解析和初始化，如果没有，必须先进行类的加载。
	类加载完成后，接着就是为新生对象分配内存空间。两种方法：
	1、指针碰撞
	2、空闲列表
	采用哪种方式取决于内存是否规整，内存是否规整取决于采取的内存回收器是否具有压缩整理机制。

对象布局

	在HotSpot虚拟机中，对象在内存中布局分为：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）。
	一、对象头
	对象头存储两部分数据：
	1、存储对象自身的运行时数据，如哈希码、GC分代年龄、锁状态标志等，此部分数据在32位和64位虚拟机中分别为32bit和64bit，官方称它为“Mark Word”；
	2、存储对象指针，即对象指向它的类元数据的指针，虚拟机通过此指针确定此对象属于哪个类的实例。
	注：如果对象是数组，对象头中还需记录数组长度数据，虚拟机可通过普通的java对象元数据确定java对象大小，但从数组的元数据无法确定数组大小。
	二、实例数据
	对象真正存储的有效信息，即程序代码中所定义的各种类型的字段内容。
	存储顺序受虚拟机分配策略参数（FieldsAllocationStyle）和字段在java源码中定义顺序影响。
	三、对齐填充
	仅起占位符作用，不必然且无特殊含义。
	HotSpot VM内存管理系统要求对象大小必须是8字节的整数倍，对象头部分正好是8字节的倍数，因此，当对象实例数据没有对齐是，需要通过对齐补充来补全。

对象访问

	java程序通过虚拟机栈（局部变量表）上的reference来操作堆上的对象。
	虚拟机规范只规定了reference类型指向对象的引用，未定义引用通过何种方式定位、访问堆中对象的具体位置，取决于虚拟机实现，目前主流的访问方式有两种：
	1、句柄方式
	在java堆中划分出一块内存作为句柄池，reference中存储的是对象句柄地址，句柄中包括对象实例数据和类型数据各自的具体地址信息。
	2、直接指针
	java堆对象布局中须考虑如何放置访问类型数据的相关数据，reference中直接存储对象地址。
	优缺点：
	句柄访问最大好处是reference中存储稳定的句柄地址，对象移动只会改变句柄中实例数据指针，reference本身无需修改。
	直接指针最大好处就是速度快，因为节省了一次指针定位的时间开销，HotSpot使用的此方式进行对象访问。
句柄访问示例图：
![](http://i.imgur.com/xJN8ASM.png)
	
直接指针示例图：
![](http://i.imgur.com/DwSep0K.png)
	
### - 垃圾收集

收集算法

	引用计数器
	定义：在对象中添加一个引用计数器，每当一个地方引用它时，计数器的值加1；每当引用失效时，计数器的值减1；当计数器的值为0的对象就是不可能被使用的。
	注：主流的java虚拟机不是使用此算法管理内存的，主要原因是它很难解决对象之间相互引用的问题。
	
	可达性分析
	基本思路：通过一系列称为“GC Roots”的对象作为起点开始向下搜索，搜索走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的，此对象将被判定为是可回收的对象。
	java语言中，可作为GC Roots的对象包括以下几种：
	1、虚拟机栈（栈帧中的局部变量表）中引用的对象；
	2、方法区中类静态属性引用的对象；
	3、方法区中常量引用的对象；
	4、本地方法栈中JNI（Native方法）引用的对象。

	再谈引用
	JDK1.2之前定义：如果reference类型数据中存储的数值代表的另一块内存的起始地址，就称这块内存代表着一个引用。
	JDK1.2之后对java引用的概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）和虚引用（Phantom Reference），四种引用强度依次减弱。
	1、强引用
	强引用就是指在程序代码中普遍存在的，类似Object obj = newObject（）这类的引用，只要强引用存在，垃圾收集器永远不会回收掉被引用的对象。
	2、软引用
	软引用描述一些有用但并不必需的对象。对于软引用关联着的对象，系统将在发生内存溢出异常之前，将此类对象列入回收范围之中进行第二次回收。JDK1.2后提供SoftReference类实现软引用。
	3、弱引用
	弱引用描述非必需的对象。此类对象只能生存到下一次垃圾收集之前，无论当前内存是否足够，垃圾回收时都会回收掉该类对象。JDK1.2后提供WeakReference类实现弱引用。
	4、虚引用
	虚引用也称为幽灵引用或幻影引用。一个对象是否有虚引用存在，完全不会对其生存时间构成影响，也无法通过需引用获取对象实例。设置虚引用关联的唯一目的是能在此对象被收集器回收时收到一个系统通知。JDK1.2后提供PhantomReference类实现虚引用。

	对象死亡判定
	真正判定一个对象死亡，需经历两次标记过程：
	1、如果对象在进行可达性分析后没有与任何GC Roots连接的引用链，将第一次标记并且进行一次筛选，筛选条件是对象是否有必要执行finalize()方法，对象未覆盖Object finalize()方法，或finalize()方法已经被虚拟机调用过，虚拟机将此两种情况视为“没有必要执行”。；
	2、在F-Queue队列中进行第二次小规模标记。如果对象判定为有必要执行finalize()方法，该对象将被放置F-Queue队列，并由虚拟机自动建立、低优先级的Finalizer线程触发执行。

	回收方法区
	方法区垃圾回收主要回收两部分内容：废弃常量和无用的类。
	判定无用的类需同时满足3个条件：
	1、该类所有的实例都已经被回收，也就是java堆中不存在该类的任何实例；
	2、加载该类的ClassLoader已经被回收；
	3、该类对应的java.lang.Class对象没有在任何地方被引用，无法通过反射访问该类的方法。
	满足以上3个条件的无用类可以进行回收，仅仅是“可以”，并不是像对象一样，不使用了就必然会回收。是否对类进行回收在HotSpot虚拟机通过-Xnoclassgc参数控制。
	在大量使用反射、动态代理、CGLib等ByteCode框架、动态生成JSP等等场景中虚拟机需具备类卸载的功能，以保证方法区不会内存溢出。

	标记-清除算法
	标记-清除算法分为“标记”和“清除”两个阶段，首先标记出所有需要回收的对象，标记完成后系统统一回收所有被标记的对象。该方法主要有两点不足：
	1、效率问题，标记和清除两个过程效率都不高；
	2、空间问题，该算法之后会产生大量内存碎片，导致在后续需要分配较大对象是，无法找到足够的连续内存而不得不提前出发另一次垃圾收集。

	复制算法
	复制算法将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当一块用完，将该块上存活的对象复制到另一块上，然后将该块已使用过的内存空间一次清理掉。
	该方法主要不足是将内存缩小为原来的一半。
	商业虚拟机都采用此算法，但并非按1：1的比例划分空间，而是将内存划分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中的一块Survivor。当回收时，将Eden和Survivor中还存活的对象一次性复制到另一块Survivor空间上，最后清理掉Eden和使用过的Survivor空间。HotSpot虚拟机默认Eden、From Survivor、To Survivor比例是8：1：1，当To Survivor空间不足时，需要依赖其他内存进行分配担保（针对新生代，老年代进行担保）。

	标记-整理算法
	标记过程与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存（针对老年代）。

	分代收集算法
	一般把java堆划分为新生代和老年代，根据各个年代的特点采用最适当的收集算法。新生代选用复制算法；老年代选用标记-清除或标记-整理算法。

### - 类加载机制



