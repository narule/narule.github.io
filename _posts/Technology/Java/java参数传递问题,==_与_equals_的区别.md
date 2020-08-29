>java中方法参数传值问题, 基本数据传递值,对象传引用(内存地址)

基本类型参数传递值(数值内容)
引用类型参数传递址(内存地址)

## ==与equals区别

对象而言

### ==

比较的是两位数据的引用值(内存地址)

### equals 

比较两个对象内容是否相同，String比较的是本身字符串
并且不同类如果重写了equals方法具体需要看重写equals方法内部逻辑。

```java

String A = new String("123");

String B = "12" + "3";

System.out.printl(A == B);

System.out.printl(A.equals(B));

```
结果：
A == B : false     A 与 B 是两个对象，有自己唯一的内存地址，地址不同，所以返回false
A.equals(B) : true      A对象的字符串内容与B对象完全相等，String类的equals方法是比较字符串内容，所以返回true
这里很明确，String 是类对象，是引用类型。

如果是：
int a = 123;
int b = 120 + 3;
那a和b是相等的 a==b 返回true  int 是基础数据类型，==比较的是两个int的值

https://www.cnblogs.com/whcwkw1314/p/8051581.html