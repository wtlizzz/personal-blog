title: Java ASM使用
author: Wtli
tags:
  - ASM
categories:
  - 后端
date: 2020-11-27 10:48:00
---
使用Java ASM，根据应用学习。
<!--more-->

ASM提供了三个基于ClassVisitor API的核心组件来生成和转换类，分别是ClassReader、ClassWriter、ClassVisitor:

ClassReader类解析以字节数组形式给出的已编译类，并对作为参数传递给它的accept方法的ClassVisitor实例调用相应的visitXxx方法。它可以被看作是一个事件生成器。
ClassWriter类是ClassVisitor抽象类的子类，该类直接以二进制形式构建已编译类。它产生一个包含编译类的字节数组作为输出，可以使用toByteArray方法检索。它可以被视为事件使用者。
ClassVisitor类将它接收到的所有方法调用委托给另一个ClassVisitor实例。它可以被看作是一个事件过滤器。
接下来的部分将通过具体示例展示如何使用这些组件来生成和转换类。

### ClassVisitor

```
/**
 * A visitor to visit a Java class. The methods of this class must be called in
 * the following order: <tt>visit</tt> [ <tt>visitSource</tt> ] [
 * <tt>visitOuterClass</tt> ] ( <tt>visitAnnotation</tt> |
 * <tt>visitTypeAnnotation</tt> | <tt>visitAttribute</tt> )* (
 * <tt>visitInnerClass</tt> | <tt>visitField</tt> | <tt>visitMethod</tt> )*
 * <tt>visitEnd</tt>.
 */
```
ClassVisitor类：一个Java Class的访问者。


类里方法调用顺序为：

<tt>visit</tt> \[ <tt>visitSource</tt> ] \[
  <tt>visitOuterClass</tt> ] ( <tt>visitAnnotation</tt> |
  <tt>visitTypeAnnotation</tt> | <tt>visitAttribute</tt> )\* (
  <tt>visitInnerClass</tt> | <tt>visitField</tt> | <tt>visitMethod</tt> )\* <tt>visitEnd</tt>



ClassReader结合ClassVisitor使用，查看类的运行情况。

```
    public static void main(String[] args) throws IOException {
        TestAsmVisitor testAsmVisitor = new TestAsmVisitor();
        ClassReader cr = new ClassReader("com/example/asmDemo/Test/Test");
        cr.accept(testAsmVisitor, 0);

        Test test = new Test();
        System.out.println(test.add(1, 2));
    }
```


```
public class TestAsmVisitor extends ClassVisitor {

    public TestAsmVisitor() {
        super(ASM4);
    }

    @Override
    public void visit(int version, int access, String name, String signature, String superName, String[] interfaces) {
        System.out.println(name + " extends " + superName + " {");
    }

    @Override
    public void visitSource(String source, String debug) {
        System.out.println(" visitSource: " + source + " " + debug);
    }

    @Override
    public void visitOuterClass(String owner, String name, String desc) {
        System.out.println(" visitOuterClass: " + owner + " " + name + " " + desc);
    }

    @Override
    public AnnotationVisitor visitAnnotation(String desc, boolean visible) {
        return null;
    }

    @Override
    public void visitAttribute(Attribute attr) {
        System.out.println(" visitAttribute: " + attr);
    }

    @Override
    public void visitInnerClass(String name, String outerName,
                                String innerName, int access) {
        System.out.println(" visitInnerClass: " + name + " " + outerName + " " + innerName + " " + access);
    }

    @Override
    public FieldVisitor visitField(int access, String name, String desc,
                                   String signature, Object value) {
        System.out.println(" visitField: " + access + " " + desc + " " + name + " " + signature + " " + value);
        return null;
    }

    @Override
    public MethodVisitor visitMethod(int access, String name,
                                     String desc, String signature, String[] exceptions) {
        System.out.println(" visitMethod: " + name + " " + desc + " " + signature + " " + exceptions);
        return null;
    }

    @Override
    public void visitEnd() {
        System.out.println("}");
    }
}
```

控制台打印结果

```
com/example/asmDemo/Test/Test extends java/lang/Object {
 visitSource: Test.java null
 visitField: 2 I num1 null null
 visitField: 9 I NUM1 null null
 visitMethod: <init> ()V null null
 visitMethod: func (II)I null null
 visitMethod: add (II)I null null
 visitMethod: sub (II)I null null
 visitMethod: <clinit> ()V null null
}
```

### ClassReader

```
/**
 * A Java class parser to make a {@link ClassVisitor} visit an existing class.
 * This class parses a byte array conforming to the Java class file format and
 * calls the appropriate visit methods of a given class visitor for each field,
 * method and bytecode instruction encountered.
 */
```

ClassReader:使用现有的ClassVisitor类访问现有的class类。把字段、方法、字节码指令绑定到ClassVisitor中。


### ClassWriter

```
/**
 * A {@link ClassVisitor} that generates classes in bytecode form. More
 * precisely this visitor generates a byte array conforming to the Java class
 * file format. It can be used alone, to generate a Java class "from scratch",
 * or with one or more {@link ClassReader ClassReader} and adapter class visitor
 * to generate a modified class from one or more existing Java classes.
 *
 */
```

生成字节码形式的类。更准确地说，这个访问者生成一个符合Java类文件格式的字节数组byte\[]类型，它可以单独使用，以“从头开始”生成Java类，也可以与一个或多个{@link ClassReader ClassReader}和适配器类访问者一起从一个或多个现有Java类生成修改后的类。

使用
```
        ClassWriter cw = new ClassWriter(0);
        cw.visit(V1_8, ACC_PUBLIC, "com/example/asmDemo/Test/Test", null, "java/lang/Object", null);
        cw.visitField(ACC_PUBLIC + ACC_FINAL + ACC_STATIC, "add", "(II)I", null, null).visitEnd();
        cw.visitEnd();
        byte[] b = cw.toByteArray();
        try {
            FileOutputStream fos = new FileOutputStream(new File("./Comparable.class"));
            fos.write(b);
            fos.flush();
            fos.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
```
























