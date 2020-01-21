---
title: 对String、StringBuilder以及StringBuffer的简单源码分析
date: 2018-04-13 00:02:52
tags: JAVA
description: 对源码进行了阅读以及简单地分析
---
## String StringBuilder StringBuffer的主要数据组织及功能实现

### String类
- 关键内部成员
```JAVA
private final char value[];
private final int offset;
private final int count;
//部分成员变量已省略
```
value[]就是用来指向字符数组,offset是指String类中第一个被用到的字符的偏移量(成员方法中有用到),count
就是指字符的总数。
- 无参数初始化

```JAVA
public String() {
    this.offset = 0;
    this.count = 0;
    this.value = new char[0];
}
```

源代码处有以下注释:
>Note that use of this constructor is unnecessary since Strings are immutable.

从源码中可以看出String类初始化在没有参数时是新建了一个长度为0的字符数组(空字符数组)。

- 以另一个String作为参数初始化
```JAVA
public More String(String original) {
    int size = original.count;
    char[] originalValue = original.value;
    char[] v;
    if (originalValue.length > size) {
        int off = original.offset;
        v = Arrays.copyOfRange(originalValue, off, off+size);
    } else {
        v = originalValue;
    }
    this.offset = 0;
    this.count = size;
    this.value = v;
}
```
其他都比较好理解,比较难以理解的地方在于if这一判断代表什么含义。其实可以结合String的成员函数substring
来理解。
```JAVA
        public String More ...substring(int beginIndex, int endIndex) {//越界判断
            return ((beginIndex == 0) && (endIndex == count)) ? this :
                    new String(offset + beginIndex, endIndex - beginIndex, value);
```
然后我们可以看看String(int, int, char[])这一成员函数
```JAVA
String(int offset, int count, char value[]) {
    this.value = value;
    this.offset = offset;
    this.count = count;
}
```
可以看到调用substring是重新够造了一个String对象,value和原String对象指向同一字符数组,但offset和count不
同。所以就有我们在上面看到的以一个String对象为参数构造一个新的String对象时的代码。需要先判断才能进行
赋值,如果参数String的字符数组的长度就是count,就直接把新的String的value指向参数String的value即可,否
则需要进行拷贝操作。从这里也可以看出,当写表达式String str1 = str2时,是新构造了一个对象。

- equals成员函数
```JAVA
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

可以非常明显的看出来,在进行equals判断操作时,如果是同一个对象的引用就直接return true,否则是对两个字
符数组的内容进行一个字符一个字符的比对,所以说如果两个String的内容相同或是同一个引用都会返回true。

### StringBuilder类
- 关键成员变量 
- 
因为StringBuider继承了AbstractStringBuilder类,所以我们可以查看AbstractStringBuider的源
码:
```JAVA
char value[];
int count;
```
其中value的实际占用空间其实是compacity,在StringBuider中有很重要的地位,会动态发生变化,而count是指
value实际已经使用的空间。

-初始化 

同样可以在AbstractStringBuilder中先看一下他的初始化函数
```JAVA
AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}
```
然后我们可以看下StringBuider中的初始化函数
```JAVA
public StringBuilder(int capacity) {
    super(capacity);
}
public StringBuilder() {
    super(16);
}
```
可以看到,在没有参数的情况下,StringBuider会默认new一个长度为16的字符数组,而如果提供参数,则会根据
参数进行初始化。 当参数为一个字符串时:
```JAVA
public StringBuilder(String str) {
    super(str.length() + 16);
    append(str);
}
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
public AbstractStringBuilder append(String str) {
    if (str == null) str = "null";
    int len = str.length();
    if (len == 0) return this;
    int newCount = count + len;
    if (newCount > value.length)
        expandCapacity(newCount);
    str.getChars(0, len, value, count);
    count = newCount;
    return this;
}
```
其中str.getChars完成了从str的value拷贝到StringBuilder的value的工作。即当用String去初始化StringBuider时,value申请的空间会比String的value的字符数组长度再大16的空间,然后把字符数组拷贝到这个空间中去。如果直接执行append的话还得注意扩大compacity。还有要注意的是,如果str是空字符串,那么会append
"null"这一字符数组,这也是StringBuider的一个特点。 我们顺便看一下append另一个StringBuider对象时发生了什么:
```JAVA
private StringBuilder append(StringBuilder sb) {
    if (sb == null)
        return append("null");
    int len = sb.length();
    int newcount = count + len;
    if (newcount > value.length)
        expandCapacity(newcount);
    sb.getChars(0, len, value, count);
    count = newcount;
    return this;
}
void expandCapacity(int minimumCapacity) {
    int newCapacity = (value.length + 1) * 2;
    if (newCapacity < 0) {
        newCapacity = Integer.MAX_VALUE;} else if (minimumCapacity > newCapacity) {
        newCapacity = minimumCapacity;
    }
    value = Arrays.copyOf(value, newCapacity);
}
```
getChars的方法定义在AbsrtactStringBuilder中,执行的是arraycopy,所以说是先判断value的空间是否足够,足
够的情况下将sb的value内容拷贝到需要append的对象后方即可。(copyOf其实是为value新申请一块更大的空
间,这里不做详细展开)

### StringBuffer类
在阅读源码时,我们发现StringBuffer和StringBuilder几乎完全一样,最大的不同就是在许多成员方法前有
synchronized,即线程锁,就是两个并发的线程同时访问同一个object时,只有一个能执行,另一个必须等待,具
体细节不作展开,后续内容也都将主要比较StringBuilder和String类。
String, StringBuilder, StringBuffer比较
```JAVA
    public String toString() {
// Create a copy, don't share the array
        return new String(value, 0, count);
    }
```
在StringBuilder中有一个toString操作,本质上还是新建了一个String对象,然后将value拷贝。而在String的成员函数中我们可以看到对应的构造方法:
```JAVA
public String(StringBuilder builder) {
    String result = builder.toString();
    this.value = result.value;
    this.count = result.count;
    this.offset = result.offset;
}
```
再结合之前的StringBuilder构造以String为参数时的情况,两者的差异还是非常明显的。其实在很多别的成员函数
中都可以找到类似的差异,但因为构造函数是最为直观的,所以其他的也就不在这边额外展开了。

在之前分析String类时,我们在注释中常常可以看到immutable,即不可更改的为什么说String类是immutable呢?
下面有一个例子:
```JAVA
String s1 = "abc";
String s2 = "def";
String s3 = s1 + s2;
```
这个代码编译成class文件并反编译,得到反编译后的代码如下:
```JAVA
String s = "abc";
String s1 = "def";
String s2 = (new StringBuilder()).append(s).append(s1).toString();
```

通过阅读StringBuider的toString()方法后我们就知道每次执行这种操作都会new一个String对象,即原来的对象不是被修改而是直接新建了一个对象。 而StringBuilder就不一样了,在之前的操作中我们可以看到,除了在
expandCompacity时有对value进行一个new的操作,其他都是直接对字符数组进行操作而完成的,所以说
StringBuffer以及StringBuilder都是mutable的。

## 什么这样设计,这么设计对String, StringBuilder及StringBuffer的影响?他们适合哪些场景?
### 为什么在有String的情况下还会有StringBuilder呢? 
其实原因在之前的分析过程中已经提到了,String的一个

很重要的特点就是不可变,每当调用String中的方法时,都会新建一个String的object,这会大大影响程序运
行的效率,因为新建需要资源,原先的object回收同样要利用系统的资源才能完成。所以才会有StringBuilder
这样更加灵活的类,在调用append时,直接对字符数组层面进行操作,可以大大提高效率。而StringBuffer则
在StringBuilder的基础上保证了多线程的安全,牺牲了部分速度和性能来保障安全性。 所以在很多情况下,
我们可以使用StringBuilder来进行字符串的相关操作来提高效率,如果设计多线程的相关编程的话则可以使用
StringBuffer。 当然String类也有其优点,因为其不可变的特性,才能实现String常量池,节约很多heap的空
间。同时因为其不可变,在map时响应也会更快,所以在Hashmap时我们也往往使用String。 同时,因为其
不可变的特性,它还能保证一定的安全性。

再来看看这个问题
```JAVA
String s1 = "Welcome to Java";
String s2 = new String("Welcome to Java");
String s3 = "Welcome to Java";
System.out.println("s1 == s2 is " + (s1 == s2));
System.out.println("s1 == s3 is " + (s1 == s3));
```
为什么第一个输出是false,而第二个输出是true呢? 这里就要讲一下常量池(string constant pool)的概念了。
其实这个概念也比较好理解,指的是在编译期被确定,并被保存在已编译的.class文件中的一些数据。它包括了关
于类、方法、接口等中的常量,也包括字符串常量和符号引用。所以在创建s1时,因为编译阶段已经将"Welcome
to Java"放入了常量池,所以JVM在check时找到了,返回了他的引用赋值给s1,s3也是同一个道理,所以s1 ==
s2会返回true。 而s2在创建过程中,根据我们之前的源码分析,会根据字符串常量再创建一份复制,所以自然和
s1不一样。