JVM把class文件加载的内存，并对数据进行校验、转换解析和初始化，最终形成JVM可以直接使用的Java类型的过程就是加载机制。  
类从被加载到虚拟机内存中开始，到卸载出内存为止，它的生命周期包括了：加载\(Loading\)、验证\(Verification\)、准备\(Preparation\)、解析\(Resolution\)、初始化\(Initialization\)、使用\(Using\)、卸载\(Unloading\)七个阶段，其中验证、准备、解析三个部分统称链接。  
**1.加载**  
在加载阶段，虚拟机需要完成以下事情：

1. 通过一个类的权限定名来获取定义此类的二进制字节流
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
3. 在java堆中生成一个代表这个类的java.lang.Class对象，作为方法去这些数据的访问入口

**2.验证**  
在验证阶段，虚拟机主要完成：

1. 文件格式验证：验证class文件格式规范
2. 元数据验证：这个阶段是对字节码描述的信息进行语义分析，以保证起描述的信息符合java语言规范要求
3. 字节码验证：进行数据流和控制流分析，这个阶段对类的方法体进行校验分析，这个阶段的任务是保证被校验类的方法在运行时不会做出危害虚拟机安全的行为
4. 符号引用验证：符号引用中通过字符串描述的全限定名是否能找到对应的类、符号引用类中的类，字段和方法的访问性\(private、protected、public、default\)是否可被当前类访问

**3.准备**  
准备阶段是正式为类变量（被static修饰的变量）分配内存并设置变量初始值（0值）的阶段，这些内存都将在方法区中进行分配

**4.解析**  
解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程  
常见的解析有四种：

1. 类或接口的解析
2. 字段解析
3. 类方法解析
4. 接口方法解析

**5.初始化**  
初始化阶段才真正开始执行类中定义的java程序代码，初始化阶段是执行类构造器\(\)方法的过程
