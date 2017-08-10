Intent即意图，是Android中连接四大组件的枢纽，Android中的Activity、Service和BroadcastReceiver都依靠Intent来启动。

Intent对象的属性大致包含七种，分别是Component、Action、Category、Data、Type、Extra、Flag。

---

**Component**

Component用于明确指定需要启动的目标组件，使用方法如下：

```java
Intent intent = new Intent();  
// 方法一：传入上下文参数与class参数  
intent.setComponent(new ComponentName(context, XXX.class));  
// 方法二：传入包名与类名  
intent.setComponent(new ComponentName(pkg, cls));  
```

指定Component属性的Intent已经明确了它将要启动的组件，因此这种Intent也被称为显示Intent，没有指定Component属性的Intent被称为隐试Intent。

---

**Action、Category**

Action代表Intent所要完成的一个抽象“动作”，而Category则用于为Action增加额外的附加类别信息，它们的使用方法如下:

```java
Intent intent = new Intent();  
// 设置一个字符串代表Action  
intent.setAction(action);  
// 添加一个字符串代表category  
intent.addCategory(category1);  
intent.addCategory(category2);
```

值得注意的是Action属性是唯一的，但Category属性可以有多个。通常设置了Action和Category来启动组件的Intent就不指定Component属性了，因此这种Intent被称为隐试Intent。

---

**Data、Type**

Data属性接受一个Uri对象，Data属性通常用于向Action属性提供操作的数据。Type属性用于指定该Data属性所指定Uri对应的MIME类型。它们的使用方法如下：

```java
Intent intent = new Intent();  
// 设置Data属性  
intent.setData(new Uri());  
// 设置Type属性  
intent.setType(type);  
// 同时设置Data和Type属性  
intent.setDataAndType(data, type);  
```

---

**Flag**

Intent的Flag属性用于为该Intent添加一些额外的控制旗标，Intent可调用addFlags方法来添加控制旗标。

---

### **可以传递的数据类型**：

**Serializable**:

将 Java 对象序列化为二进制文件的 Java 序列化技术是 Java系列技术中一个较为重要的技术点，在大部分情况下，开发人员只需要了解被序列化的类需要实现 Serializable 接口，使用ObjectInputStream 和 ObjectOutputStream 进行对象的读写。

**Charsequence**:

在JDK1.4中，引入了CharSequence接口，实现了这个接口的类有：CharBuffer、String、StringBuffer、StringBuilder这个四个类。

CharBuffer为nio里面用的一个类，String实现这个接口理所当然，StringBuffer也是一个CharSequence，StringBuilder是Java抄袭C\#的一个类，基本和StringBuffer类一样，效率高，但是不保证线程安全，在不需要多线程的环境下可以考虑。

提供这么一个接口，有些处理String或者StringBuffer的类就不用重载了。但是这个接口提供的方法有限，只有下面几个：charat、length、subSequence、toString这几个方法，感觉如果有必要，还是重载的比较好，避免用instaneof这个操作符。

**Parcelable**:

Android提供了一种新的类型：Parcel。本类被用作封装数据的容器，封装后的数据可以通过Intent或IPC传递。 除了基本类型以

外，只有实现了Parcelable接口的类才能被放入Parcel中。

是GOOGLE在安卓中实现的另一种序列化,功能和Serializable相似,主要是序列化的方式不同

**Bundle**:

Bundle是将数据传递到另一个上下文中或保存或回复你自己状态的数据存储方式。它的数据不是持久化状态。

