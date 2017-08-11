在JAVA语言中，字符串数据实际上由String类所实现的。Java字符串类分为两类：一类是在程序中不会被改变长度的不变字符串；二类是在程序中会被改变长度的可变字符串。Java环境为了存储和维护这两类字符串提供了 String和StringBuffer两个类。

### String的一些常用操作 {#toc_12}

#### &lt;1&gt;字符串创建 {#toc_13}

```
String　str="This is a String";
```

或者

```
String　str=new String（"This is a String");
```

#### &lt;2&gt;字符串长度 {#toc_14}

```
String str= 
"This is a String"
;

int
len
 =str.length();

```

#### &lt;3&gt;指定字符或子字符串的位置 {#toc_15}

```java
String str="Thisis a String";
Int index1 =str.indexOf(
"i"
);   
//2

Int index２＝str.lastIndexOf(
"i"
)； 
//12
```

#### &lt;4&gt;判断两个字符串是否相等 {#toc_16}

```
String
 str=
"This　is a String"
;

Boolean
 result=str.
equals
(
"This is another String"
);

```

#### &lt;5&gt;得到指定位置的字符 {#toc_17}

```
String
 str=
"This　is a String"
;

char
 chr=str.charAt(
3
);

```

#### &lt;6&gt;截取子字符串 {#toc_18}

```
str＝str.substring(
int
 beginIndex);

```

截取掉str从首字母起长度为beginIndex的字符串，将剩余字符串赋值给str；

```
str＝str.substring(
int
 beginIndex，
int
 endIndex);

```

#### &lt;6&gt;字符串合并 {#toc_19}

```
String
 str=
"This　is a String"
;

String
 str1=str.concat(
"Test"
); 
//str1="This　is a String Test"
```



