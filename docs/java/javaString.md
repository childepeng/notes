# String对象不可变
Java中，String是引用数据类型，String类型的变量中保存的实际上是一个指向一个内存的地址引用，但更改字符串类型的变量时，变量指向的内存地址也在改变，而原来内存地址中存储的字符串是不变的。

```
String a = "a";
String temp = a;
a = "b";
// '==' 比较对象地址
System.out.println(a == temp);    // false
System.out.println(temp == "a");  // true
```

Java虚拟机中有个字符串产量池（String pool）是Java堆内存中特殊的存储区域，当创建一个String对象时，如果字符串已经存在于常量池中，则不会创建一个新的字符串对象，而是引用已经存在的对象。


# String对象为什么不可变
String对象源码

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {

    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0


    public String() {
        this.value = "".value;
    }

    public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }

    ...
}
```
String类final，不可被继承。 String中的字符串以char数组的形式保存在value数组中，value也是final，不可修改。


# String真的不可变？
String类通过private、final等保护String不被修改，但是通过Java的反射机制，任是可以修改String的值。
```
//创建字符串"Hello World"， 并赋给引用s
String s = "Hello world";
String temp = s;

//获取String类中的value属性
Field valueFieldOfString = String.class.getDeclaredField("value");

//改变value属性的访问权限
valueFieldOfString.setAccessible(true);

//获取s对象上的value属性的值
char[] value = (char[]) valueFieldOfString.get(s);

//改变value所引用的数组中的第5个字符
value[5] = '_';
System.out.println("s = " + s);  //Hello_World
System.out.println(s == temp);  // true,  s与temp仍是同一个对象，说明s的地址没变
```
