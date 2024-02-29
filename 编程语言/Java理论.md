# Java理论

## Java基础

### 面向对象特性

    封装：封装是将数据和操作数据的方法捆绑在一起，形成一个独立的单元。

    继承：允许一个类继承另一个类的属性和方法，子类可以重用父类的代码并添加新的功能。

    多态：指同一个方法可以根据调用它的对象不同而表现出不同的行为。

    抽象：是将对象的共同特征提取出来形成类或接口
    
### public,private，protected以及默认之前的区别

    public：对所有类可见。
    private：仅对声明它的类可见。
    protected：对同一包内和子类可见。
    默认：对同一包内可见。

### 抽象类和接口的异同

相同点：

    都可以用于实现多态和都不能被实例化。

不同点：

     一个类只能继承一个抽象类但能实现多个接口
     抽象类可以有构造函数，且可以有参数；接口不能包含构造函数，因为接口中都是抽象方法，没有方法体
     抽象类中可以包含普通方法的实现，而接口中所有方法都是抽象的


### Java异常

异常主要分为三类：受检异常、运行时异常和Error。

受检异常是在编译时必须进行处理的异常，否则会报错。
使用try-catch块或throws关键字来处理或者在方法签名处使用throws声明该异常。

运行时异常是在运行时可能抛出的异常，编译器不会强制要求处理这类异常，开发者可以选择是否处理。
使用try-catch块或者在需要的情况下处理这些异常，一般来说，运行时异常是由程序逻辑错误导致的，应该尽量避免发生这类异常。

Error是指Java运行时系统的内部错误和资源耗尽等严重问题，一般情况下无法通过代码来捕获和处理Error。
开发者通常不需要处理Error，而是应该尽量避免引起Error的情况。
   
### 常用的集合类

    ArrayList：
        ArrayList是基于数组实现的动态数组，可以根据需要动态增长或缩减容量。
        特点：随机访问速度快，但插入和删除元素效率较低。

    LinkedList：
        LinkedList是基于双向链表实现的集合类，可以高效地进行插入和删除操作。
        特点：插入和删除元素效率高，但随机访问元素的效率较低。

    HashMap：
        HashMap是基于哈希表实现的键值对集合，可以快速查找、插入和删除元素。
        特点：通过键来存储和访问值，查找效率高，无序。

    TreeMap：
        TreeMap是基于红黑树实现的有序键值对集合，可以按照键的自然顺序或自定义顺序进行排序。
        特点：按照键的顺序有序存储，支持范围查找等操作。

    HashSet：
        HashSet是基于HashMap实现的无重复元素集合，不保证元素的顺序。
        特点：不能包含重复元素，查找元素效率高，无序。

    TreeSet：
        TreeSet是基于TreeMap实现的有序集合，可以按照元素的自然顺序或自定义顺序进行排序。
        特点：集合中的元素有序排列，支持范围查找等操作。

    LinkedHashMap：
        LinkedHashMap是基于HashMap和双向链表实现的有序键值对集合，可以按照插入顺序或访问顺序进行排序。
        特点：保持元素插入顺序或访问顺序不变，具有HashMap和LinkedHashMap的特点。

### ArrayList和LinkedList

ArrayList和LinkedList内部的实现方式有所不同：

    ArrayList的内部实现：
        ArrayList是基于数组实现的动态数组，当数组容量不足时会自动扩容。
        在添加元素时，如果数组已满，会创建一个新的更大容量的数组，并将原数组中的元素复制到新数组中。
        ArrayList支持随机访问，可以通过索引直接访问元素，因为它内部使用数组来存储元素。

    LinkedList的内部实现：
        LinkedList是基于双向链表实现的集合，每个节点都包含元素值和指向前一个和后一个节点的指针。
        在添加或删除元素时，只需修改相邻节点的指针，无需像ArrayList那样进行元素的移动和复制操作。
        LinkedList不支持随机访问，需要从头或尾开始遍历链表才能访问特定位置的元素。

ArrayList和LinkedList之间的区别和优缺点如下：

    访问效率：
        ArrayList支持高效的随机访问，通过索引可以直接访问元素，时间复杂度为O(1)。
        LinkedList在访问时需要遍历节点，时间复杂度为O(n)，不适合频繁的随机访问操作。

    插入和删除效率：
        ArrayList在中间或头部插入/删除元素时，需要移动后续元素，时间复杂度为O(n)。
        LinkedList在中间插入/删除元素时，只需修改相邻节点的指针，时间复杂度为O(1)。

    空间占用：
        ArrayList在元素较少时可能会浪费一些内存，因为需要预留一定的容量空间。
        LinkedList每个节点都需要额外的指针空间，可能会占用更多的内存。

    适用场景：
        如果需要频繁进行随机访问操作，应该选择ArrayList。
        如果需要频繁进行插入和删除操作，尤其是在中间位置，应该选择LinkedList。

### 内存溢出

内存溢出是指程序在申请内存时，未能获得所需的内存空间，导致程序无法正常运行或崩溃的情况。内存溢出通常发生在以下场景：

    无限递归：
        当一个函数无限递归调用自身，且没有合适的终止条件时，会导致栈空间不断增长，最终导致栈溢出。

    内存泄漏：
        当程序中动态分配的内存未被正确释放，导致程序长时间运行后累积的未释放内存达到系统上限，引发内存溢出。

    大对象数组：
        当程序试图创建一个过大的数组对象，超出了系统所能提供的内存空间，导致内存溢出。

    资源耗尽：
        在多线程或并发编程中，当请求的资源（如线程、数据库连接、文件句柄等）超出系统所能提供的限制时，也会导致内存溢出。

    内存泄漏：
        在使用Java等语言开发时，如果程序中存在内存泄漏，即使程序运行期间没有大量分配内存，但由于未能正确释放之前分配的内存，最终也会导致内存溢出。

    过度递归：
        当算法设计中使用了过多的递归操作，导致函数调用栈层级过深，最终引发栈溢出。

### ==和equals

    "=="用于比较两个对象的引用地址是否相同。
    "equals"方法用于比较两个对象的内容（值）是否相等。

### hashCode方法

hashCode()方法是Java中Object类的一个方法，它返回对象的哈希码值，用于确定对象在哈希表中的存储位置。hashCode()方法的作用包括：

    在哈希表中存储和查找对象：
        当一个对象要被存储到哈希表（如HashMap、HashSet等）中时，会根据对象的hashCode值确定存储位置。
        在查找对象时，首先根据对象的hashCode值确定可能存在的位置，然后再进行实际比较以确认是否为目标对象。

    提高查找效率：
        哈希表通过hashCode值将数据分散到不同的存储桶中，可以减少查找的时间复杂度，提高查找效率。

    判断两个对象是否相等：
        hashCode()方法通常与equals()方法一起使用，用于判断两个对象是否相等。
        如果两个对象的hashCode值相等，但equals方法返回false，则认为这两个对象不相等；如果两个对象的hashCode值不等，则这两个对象一定不相等。

    在集合中去重：
        在使用HashSet或HashMap等集合类存储对象时，会调用对象的hashCode()方法来确定对象在集合中的存储位置。
        如果要确保集合中不存储重复对象，则需要正确重写hashCode()和equals()方法，以确保相等的对象具有相同的hashCode值。

### HashMap

HashMap是基于哈希表实现的Map集合，它使用键值对存储数据，通过键来快速查找值。HashMap的实现原理包括以下几个关键点：

    内部数据结构：
        HashMap内部使用数组和链表（或红黑树）结合的方式来存储键值对。
        数组称为“桶”，每个桶存储一个链表或红黑树，用于解决哈希冲突（多个键映射到同一个桶）的问题。

    计算哈希值：
        当向HashMap中插入键值对时，首先计算键的哈希值，然后根据哈希值确定存储位置。
        如果多个键具有相同的哈希值（哈希冲突），它们会被存储在同一个桶中，并通过链表或红黑树来解决碰撞。

    扩容机制：
        当HashMap中的元素数量达到负载因子（load factor）与桶的容量的乘积时，HashMap会进行扩容操作，即重新调整桶的大小并重新分配元素，以保持性能。

    遍历和查找：
        HashMap通过计算键的哈希值来快速定位存储位置，从而实现快速的插入、查找和删除操作。
        在遍历HashMap时，可以通过迭代桶中的链表或红黑树来访问所有键值对。

要保证HashMap的线程安全，可以采取以下几种方式：

    使用Collections.synchronizedMap()方法：
        可以通过Collections类提供的synchronizedMap()方法将HashMap包装成线程安全的Map对象，实现简单粗暴的线程安全。

    使用ConcurrentHashMap：
        ConcurrentHashMap是Java提供的线程安全的Map实现，采用了分段锁（Segment）的机制，不仅能保证高并发环境下的线程安全，还能提高并发性能。

    使用锁机制：
        可以使用显式的锁（如ReentrantLock）来保护HashMap的操作，确保在对HashMap进行修改时只有一个线程可以访问，从而实现线程安全。

    使用并发工具类：
        可以使用Java并发框架提供的并发工具类来处理HashMap的线程安全，如使用AtomicInteger来保证计数器的原子性操作。

### 字节占用

在Java中，一个字符占用2个字节（16位），即16个比特。Java中的字符是使用Unicode编码表示的，每个字符占用16位空间。

至于Java中其他基本数据类型的大小（占用字节数）如下所示：

    int：整型数据类型int在Java中占用4个字节（32位）。范围为-2^31到2^31-1。
    long：长整型数据类型long在Java中占用8个字节（64位）。范围为-2^63到2^63-1。
    double：双精度浮点数类型double在Java中占用8个字节（64位）。范固定的8个字节，用来存储双精度浮点数。

### 创建一个类的实例的办法

在Java中，创建一个类的实例（对象）有以下几种方式：

    使用new关键字：
        最常见的方式是使用new关键字直接实例化对象，如：ClassName obj = new ClassName();

    通过反射机制：
        使用Java的反射机制，可以通过Class类的newInstance()方法来动态创建对象：ClassName obj = (ClassName) Class.forName("ClassName").newInstance();

    通过clone()方法：
        如果目标类实现了Cloneable接口，并且重写了clone()方法，就可以使用clone()方法创建对象：ClassName obj = (ClassName) originalObj.clone();

    使用工厂方法：
        可以定义一个工厂类，通过工厂方法创建对象：ClassName obj = Factory.createInstance();

    使用反序列化：
        可以通过反序列化从文件或网络中恢复一个对象：ObjectInputStream ois = new ObjectInputStream(new FileInputStream("file.ser")); ClassName obj = (ClassName) ois.readObject();

### final/finally/finalize的区别

在Java中，final、finally和finalize是三个不同的关键字，它们在语法和作用上有着不同的功能和含义。

    final：
        final 是一个修饰符（关键字），用于修饰类、方法和变量。它表示被修饰的类不能被继承、被修饰的方法不能被重写、被修饰的变量是一个常量（只能被赋值一次）。
        当一个类被声明为final时，表示该类是最终的，不能再被其他类继承。
        当一个方法被声明为final时，表示该方法不能被子类重写。
        当一个变量被声明为final时，表示该变量是一个常量，只能被赋值一次。

    finally：
        finally 是异常处理中的一个关键字，用于定义在try-catch语句块中始终会被执行的代码块。
        无论是否发生异常，finally中的代码块都会被执行。通常用于释放资源、关闭连接等操作。

    finalize：
        finalize 是Object类的一个方法，用于在对象被垃圾回收之前执行一些清理操作。
        在Java中，垃圾回收器会在对象即将被回收时调用其finalize()方法（如果该方法被覆盖的话）。
        finalize 方法在Java 9中已经被标记为过时（deprecated），不建议使用，因为无法保证何时会被调用，且可能导致性能问题。

### String/StringBuffer/StringBuilder

在Java中，String、StringBuffer和StringBuilder是用来操作字符串的类，它们之间有以下区别：

    String：
        String 类是不可变的（immutable）的，即一旦创建了String对象，就不能修改其内容。当对String对象进行操作时，实际上是创建了一个新的String对象。
        字符串拼接操作会在内存中创建新的字符串对象，导致频繁的对象创建和销毁，可能会影响性能。
        适用于字符串内容不经常变化的场景。

    StringBuffer：
        StringBuffer 是可变的（mutable）的字符串序列，可以通过调用其方法对其中的字符内容进行修改。
        StringBuffer是线程安全的，所有的方法都是同步的（synchronized）。
        适用于多线程环境或需要频繁进行字符串拼接操作的场景。

    StringBuilder：
        StringBuilder 也是可变的字符串序列，与StringBuffer类似，但不是线程安全的。
        StringBuilder的方法不是同步的，因此在单线程环境下性能更好。
        适用于单线程环境下需要频繁进行字符串操作的场景。

总结：

    如果需要频繁对字符串进行修改操作，并且在单线程环境下，推荐使用StringBuilder。
    如果在多线程环境下需要对字符串进行修改操作，推荐使用StringBuffer。
    如果字符串内容基本不变，并且只涉及到少量的拼接操作，可以使用String来提高代码的简洁性和可读性。

### ava序列化

Java序列化是指将Java对象转换为字节序列的过程，以便可以将其保存到文件、数据库中，或通过网络传输。反之，反序列化则是将字节序列转换回Java对象的过程。

要实现Java序列化，需要遥标记Serializable接口。Serializable接口是一个标记接口（marker interface），没有任何方法需要实现，只是用来标识一个类可以被序列化。一旦一个类实现了Serializable接口，它就可以被Java序列化机制序列化。

以下是实现Java序列化的步骤：

    让需要序列化的类实现Serializable接口。
    创建一个输出流，比如ObjectOutputStream，将对象写入到输出流中，这样对象就会被序列化为字节序列。
    如果想要将序列化后的字节序列保存到文件中，可以将输出流包装在FileOutputStream中。
    在反序列化时，创建一个输入流，比如ObjectInputStream，从输入流中读取字节序列，并将其转换回对象。

以下是一个简单的示例代码，演示如何实现Java序列化：

    import java.io.*;

    // 实现Serializable接口
    class Person implements Serializable {
        String name;
        int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }

    public class SerializationExample {
        public static void main(String[] args) {
            try {
                // 序列化对象
                Person person = new Person("Alice", 30);
                FileOutputStream fileOut = new FileOutputStream("person.ser");
                ObjectOutputStream out = new ObjectOutputStream(fileOut);
                out.writeObject(person);
                out.close();
                fileOut.close();
    
                // 反序列化对象
                FileInputStream fileIn = new FileInputStream("person.ser");
                ObjectInputStream in = new ObjectInputStream(fileIn);
                Person serializedPerson = (Person) in.readObject();
                in.close();
                fileIn.close();
    
                System.out.println("Deserialized Person: " + serializedPerson.name + ", " + serializedPerson.age);
            } catch (IOException | ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
    }

在这个示例中，Person类实现了Serializable接口，然后通过ObjectOutputStream和ObjectInputStream来实现对象的序列化和反序列化。

## JVM

### JVM结构

JVM（Java虚拟机）的内存结构可以分为以下几个部分：

    方法区（Method Area）：
        用于存储类的结构信息、静态变量、常量等数据。
        在JDK 8及之前，方法区中也包含永久代（Permanent Generation），用于存储类型信息、方法信息等。但在JDK 8之后，永久代被元空间（Metaspace）所取代。

    堆内存（Heap）：
        用于存储对象实例及数组，是Java程序中动态分配内存的主要区域。
        可以进一步分为新生代（Young Generation）和老年代（Old Generation）等区域，用于进行对象的新建、存活和老化过程。

    栈内存（Stack）：
        用于存储局部变量、方法调用、操作数栈等信息，每个线程都有自己的栈空间。
        包括栈帧（Stack Frame）和操作数栈（Operand Stack）。

    本地方法栈（Native Method Stack）：
        用于支持Java调用本地（Native）方法的栈空间。

    程序计数器（Program Counter Register）：
        当前线程执行的字节码指令地址。

GC（Garbage Collection，垃圾回收）是因为在JVM的堆内存中，Java程序创建的对象会占用一定的内存空间。随着程序的执行，会产生大量的对象实例，部分对象可能会变成垃圾，即不再被程序所引用的对象。如果这些垃圾对象不得到清理，将会导致内存的浪费和内存泄漏。

因此，GC的作用就是定期地对堆内存中的对象进行检查，识别出不再被引用的垃圾对象，并将其进行清理和回收，释放内存空间以便给新的对象使用。通过GC，可以保证程序运行过程中的内存空间的合理利用，避免内存泄漏和内存溢出等问题，提高了程序的健壮性和性能。

### JVM堆的基本结构

JVM堆是Java虚拟机中用于存储对象实例和数组的内存区域，是Java程序中动态分配内存的主要区域。JVM堆的基本结构可以分为以下几个部分：

    新生代（Young Generation）：
        新生代是堆内存的一部分，用于存放新创建的对象。
        又可以分为Eden区和两个Survivor区（通常是From区和To区），用于对象的存活周期管理。

    老年代（Old Generation）：
        老年代用于存放经过多次垃圾回收仍然存活下来的对象。
        大多数情况下，老年代中存放的对象具有较长的生命周期。

    永久代/元空间（Permanent Generation / Metaspace）：
        在JDK 8及之前，永久代用于存放类的结构信息、静态变量、常量等数据。
        在JDK 8之后，永久代被元空间所取代，元空间使用本地内存而不是堆内存，用于存放类的元数据、字符串池等信息。

JVM堆的不同区域在对象的分配、存活检测、垃圾回收等方面有不同的作用和算法。新生代和老年代之间通过对象晋升的方式进行数据交换，当新生代的对象经过多次存活检测后仍然存活时，会被晋升到老年代。

对于堆内存的管理和优化，可以通过调整堆的大小、选择不同的垃圾回收器、调整垃圾回收策略等手段来提高程序的性能和健壮性。

### JVM的垃圾算法

JVM的垃圾回收算法有以下几种：

    标记-清除算法（Mark and Sweep）：
        首先标记出所有活动对象，然后清除所有未被标记的对象，释放它们所占用的内存空间。
        缺点是会产生内存碎片，影响内存分配的效率。

    复制算法（Copying）：
        将堆内存分为两个区域（一般是Eden区和两个Survivor区的其中一个），每次只使用其中一个区域来存放对象，当该区域满时，将存活对象复制到另一个区域中，并进行清除。
        优点是避免了内存碎片问题，但需要额外的内存空间。

    标记-整理算法（Mark and Compact）：
        首先标记出所有活动对象，然后将活动对象向一端移动，清理掉端边界以外的内存空间，使内存连续，减少碎片。
        相比标记-清除算法，标记-整理算法能够避免内存碎片问题。

    增量式垃圾回收算法（Incremental Garbage Collection）：
        将整个垃圾回收过程分解为多个小步骤，在执行程序的同时逐步完成垃圾回收，减少暂停时间。

CMS（Concurrent Mark-Sweep）垃圾回收器是一种基于标记-清除算法的并发垃圾回收器，其基本流程如下：

    初始标记阶段（Initial Marking）：
        暂停所有应用线程，标记根对象和直接与根对象有关联的对象，并标记这些对象为活动对象。

    并发标记阶段（Concurrent Marking）：
        同时运行垃圾回收线程和应用线程，标记所有活动对象及其可达对象，不会暂停应用线程。

    重新标记阶段（Remark）：
        暂停所有应用线程，处理在并发标记阶段有可能产生的新的引用关系，对未标记的对象进行标记。

    并发清除阶段（Concurrent Sweeping）：
        同时运行垃圾回收线程和应用线程，清除所有未标记的对象，释放内存空间，不会暂停应用线程。

CMS垃圾回收器采用了并发标记和并发清除的方式，可以减少垃圾回收暂停时间，适合对响应时间要求高的应用场景。

### JVM常用启动参数调整

JVM有许多常用的启动参数，可以用来调整堆内存大小、选择垃圾回收器、设置线程栈大小等。以下是一些常用的启动参数以及它们的描述：

    -Xms 和 -Xmx：
        -Xms 用于设置JVM堆的初始大小，例如 -Xms512m 表示初始堆大小为512MB。
        -Xmx 用于设置JVM堆的最大大小，例如 -Xmx1024m 表示最大堆大小为1GB。

    -Xss：
        用于设置每个线程的堆栈大小，默认值因平台而异。例如 -Xss1m 表示设置线程堆栈大小为1MB。

    -XX:NewRatio 和 -XX:SurvivorRatio：
        -XX:NewRatio 用于设置新生代与老年代的比例，默认值为2，表示新生代占整个堆的1/3。
        -XX:SurvivorRatio 用于设置Eden区和Survivor区的比例，默认值为8，表示Eden区与一个Survivor区的比例。

    -XX:+UseConcMarkSweepGC 和 -XX:+UseG1GC：
        -XX:+UseConcMarkSweepGC 用于启用CMS垃圾回收器。
        -XX:+UseG1GC 用于启用G1垃圾回收器。

    -XX:MaxPermSize 和 -XX:MaxMetaspaceSize：
        -XX:MaxPermSize 用于设置永久代的最大大小（仅在JDK 8及之前有效）。
        -XX:MaxMetaspaceSize 用于设置元空间的最大大小（JDK 8及之后有效）。

这些启动参数可以根据具体应用的需求进行调整，以优化JVM的性能和资源利用。

### 查看JVM的内存使用情况

要查看JVM的内存使用情况，可以使用以下几种方法：

    jstat命令：
    使用 jstat 命令可以监控JVM内存使用情况，包括堆内存、非堆内存等。例如，可以使用以下命令查看JVM的堆内存使用情况：

jstat -gc <pid>

其中 <pid> 是Java进程的进程ID。

jcmd命令：
使用 jcmd 命令可以向运行中的Java进程发送诊断命令，包括查看内存使用情况。例如，可以使用以下命令查看堆内存情况：

    jcmd <pid> GC.heap_info

VisualVM：
VisualVM 是一款Java应用程序性能分析工具，可以用来监控和分析JVM的内存使用情况。通过连接到运行中的Java进程，可以查看堆内存、非堆内存、线程、类加载等信息。

JConsole：
JConsole 是JDK自带的监控工具，可以监控本地或远程的Java应用程序。通过连接到运行中的Java进程，可以查看内存使用情况、垃圾回收情况等。

