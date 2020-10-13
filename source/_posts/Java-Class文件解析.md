title: Java Class文件解析
author: Wtli
tags:
  - Java
categories:
  - 后端
date: 2020-09-23 08:56:00
---
为了更好的学习Java ASM，对Java编译后的Class文件进行解析。

Javac是用来编译.java文件的。  
Javap 主要用于帮助开发者深入了解 Java 编译器的机制。
<!--more-->

新建一个Test类
```
public class Test {
    private int num1 = 1;
    public static int NUM1 = 100;

    public int func(int a, int b) {
        return add(a, b);
    }

    public int add(int a, int b) {
        return a + b + num1;
    }

    public int sub(int a, int b) {
        return a - b - NUM1;
    }
}
```
在该目录下使用两条命令，在终端中查看class文件源代码：
```
$ javac -g Test.java 
$ javap -verbose Test.class 
```
源代码：
```
Last modified 2020-9-23; size 706 bytes
  MD5 checksum 4f1181fc0507aaf1484076210dfd6b63
  Compiled from "Test.java"
public class com.example.nettydemo.test.Test
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#26         // java/lang/Object."<init>":()V
   #2 = Fieldref           #5.#27         // com/example/nettydemo/test/Test.num1:I
   #3 = Methodref          #5.#28         // com/example/nettydemo/test/Test.add:(II)I
   #4 = Fieldref           #5.#29         // com/example/nettydemo/test/Test.NUM1:I
   #5 = Class              #30            // com/example/nettydemo/test/Test
   #6 = Class              #31            // java/lang/Object
   #7 = Utf8               num1
   #8 = Utf8               I
   #9 = Utf8               NUM1
  #10 = Utf8               <init>
  #11 = Utf8               ()V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               LocalVariableTable
  #15 = Utf8               this
  #16 = Utf8               Lcom/example/nettydemo/test/Test;
  #17 = Utf8               func
  #18 = Utf8               (II)I
  #19 = Utf8               a
  #20 = Utf8               b
  #21 = Utf8               add
  #22 = Utf8               sub
  #23 = Utf8               <clinit>
  #24 = Utf8               SourceFile
  #25 = Utf8               Test.java
  #26 = NameAndType        #10:#11        // "<init>":()V
  #27 = NameAndType        #7:#8          // num1:I
  #28 = NameAndType        #21:#18        // add:(II)I
  #29 = NameAndType        #9:#8          // NUM1:I
  #30 = Utf8               com/example/nettydemo/test/Test
  #31 = Utf8               java/lang/Object
{
  public static int NUM1;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC

  public com.example.nettydemo.test.Test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: iconst_1
         6: putfield      #2                  // Field num1:I
         9: return
      LineNumberTable:
        line 9: 0
        line 10: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      10     0  this   Lcom/example/nettydemo/test/Test;

  public int func(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=3, args_size=3
         0: aload_0
         1: iload_1
         2: iload_2
         3: invokevirtual #3                  // Method add:(II)I
         6: ireturn
      LineNumberTable:
        line 14: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       7     0  this   Lcom/example/nettydemo/test/Test;
            0       7     1     a   I
            0       7     2     b   I

  public int add(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=3
         0: iload_1
         1: iload_2
         2: iadd
         3: aload_0
         4: getfield      #2                  // Field num1:I
         7: iadd
         8: ireturn
      LineNumberTable:
        line 18: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/example/nettydemo/test/Test;
            0       9     1     a   I
            0       9     2     b   I

  public int sub(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=3
         0: iload_1
         1: iload_2
         2: isub
         3: getstatic     #4                  // Field NUM1:I
         6: isub
         7: ireturn
      LineNumberTable:
        line 22: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       8     0  this   Lcom/example/nettydemo/test/Test;
            0       8     1     a   I
            0       8     2     b   I

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: bipush        100
         2: putstatic     #4                  // Field NUM1:I
         5: return
      LineNumberTable:
        line 11: 0
}
SourceFile: "Test.java"

```
细细品尝：

**头部标示header**
```
  Last modified 2020-9-23; size 706 bytes
  MD5 checksum 4f1181fc0507aaf1484076210dfd6b63
  Compiled from "Test.java"
public class com.example.nettydemo.test.Test
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
```
Last modified：修改时间  
MD5 checksum：MD5加密  
minor version && major version：JDK的版本（52=1.8）  
flags：类的标示

**常量池**
```
Constant pool:
   #1 = Methodref          #6.#26         // java/lang/Object."<init>":()V
   #2 = Fieldref           #5.#27         // com/example/nettydemo/test/Test.num1:I
   #3 = Methodref          #5.#28         // com/example/nettydemo/test/Test.add:(II)I
   #4 = Fieldref           #5.#29         // com/example/nettydemo/test/Test.NUM1:I
   #5 = Class              #30            // com/example/nettydemo/test/Test
   #6 = Class              #31            // java/lang/Object
   #7 = Utf8               num1
   #8 = Utf8               I
   #9 = Utf8               NUM1
  #10 = Utf8               <init>
  #11 = Utf8               ()V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               LocalVariableTable
  #15 = Utf8               this
  #16 = Utf8               Lcom/example/nettydemo/test/Test;
  #17 = Utf8               func
  #18 = Utf8               (II)I
  #19 = Utf8               a
  #20 = Utf8               b
  #21 = Utf8               add
  #22 = Utf8               sub
  #23 = Utf8               <clinit>
  #24 = Utf8               SourceFile
  #25 = Utf8               Test.java
  #26 = NameAndType        #10:#11        // "<init>":()V
  #27 = NameAndType        #7:#8          // num1:I
  #28 = NameAndType        #21:#18        // add:(II)I
  #29 = NameAndType        #9:#8          // NUM1:I
  #30 = Utf8               com/example/nettydemo/test/Test
  #31 = Utf8               java/lang/Object
```
static变量
```
  public static int NUM1;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC
```

Test()方法：
类在编译时，自动生成默认构造方法
```
  public com.example.nettydemo.test.Test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: iconst_1
         6: putfield      #2                  // Field num1:I
         9: return
      LineNumberTable:
        line 9: 0
        line 10: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      10     0  this   Lcom/example/nettydemo/test/Test;
```
descriptor: ()V---返回void  
Code: Java代码编译成的字节码指令  
LineNumberTable: Java源码的行号与字节码指定的对应关系  
LocalVariableTable: 方法的局部变量描述  



func()方法：
```
  public int func(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=3, args_size=3
         0: aload_0
         1: iload_1
         2: iload_2
         3: invokevirtual #3                  // Method add:(II)I
         6: ireturn
      LineNumberTable:
        line 14: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       7     0  this   Lcom/example/nettydemo/test/Test;
            0       7     1     a   I
            0       7     2     b   I
```
其他方法：
```
  public int add(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=3
         0: iload_1
         1: iload_2
         2: iadd
         3: aload_0
         4: getfield      #2                  // Field num1:I
         7: iadd
         8: ireturn
      LineNumberTable:
        line 18: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/example/nettydemo/test/Test;
            0       9     1     a   I
            0       9     2     b   I

  public int sub(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=3
         0: iload_1
         1: iload_2
         2: isub
         3: getstatic     #4                  // Field NUM1:I
         6: isub
         7: ireturn
      LineNumberTable:
        line 22: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       8     0  this   Lcom/example/nettydemo/test/Test;
            0       8     1     a   I
            0       8     2     b   I
```
static{}:
```
  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: bipush        100
         2: putstatic     #4                  // Field NUM1:I
         5: return
      LineNumberTable:
        line 11: 0

```