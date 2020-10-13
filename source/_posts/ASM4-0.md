title: ASM4.0
author: Wtli
tags:
  - ASM
  - Java
categories:
  - 后端
date: 2020-09-23 14:26:00
---
ASM库的目标是生成、转换和分析已编译的Java类，这些类以字节数组的形式表示(因为它们存储在磁盘上并加载到Java虚拟机中)。

本文将对ASM进行深入学习。
<!--more-->

### Overview

ASM库提供两类API：
1. the core API  ---  类基于事件的表示
2. the tree API  ---  基于对象的表示

分别的优缺点：

- 基于事件的API比基于对象的API更快，需要的内存更少，因为不需要创建表示类的对象树并在内存中存储(SAX和DOM之间也存在相同的差异)。  

-  然而，使用基于事件的API实现类转换可能更加困难，因为在任何给定时间类中只有一个元素可用(对应于当前事件的元素)，而整个类在内存中使用基于对象的API可用。

**架构**

实际上，基于事件的API是围绕事件生产者(类解析器)、事件消费者(类编写器)和各种预定义的事件过滤器组织的，用户定义的生产者、消费者和过滤器可以添加到这些过滤器中。使用这个API需要两个步骤:

1. 将事件生成器、过滤器和使用者组件组装到可能复杂的体系结构中，
2. 然后启动事件生成器来运行生成或转换过程。

基于对象的API还有一个架构方面:实际上可以组合操作对象树的类生成器或转换器组件，它们之间的链接表示转换的顺序。


### Core API

本章解释了如何用core ASM API生成和转换已编译的Java类。


#### 概述

编译后的类包含:

- 描述修饰符(如public或private)、名称、父类、接口和类的注释的部分。
- 该类中声明的每个字段一个部分。每个部分描述字段的修饰符、名称、类型和注释。
- 每个方法和构造函数在这个类中声明一个部分。每个部分描述修饰符、名称、返回和参数类型以及方法的注释。它还以Java字节码指令序列的形式包含方法的编译代码。

然而，源类和编译类之间有一些区别:

- 编译后的类只描述一个类，而源文件可以包含几个类。例如，描述带有内部类的类的源文件在两个类文件中编译:一个用于主类，另一个用于内部类。然而，主类文件包含对其内部类的引用，而在方法内部定义的内部类包含对其外部方法的引用。 

- 当然，编译后的类不包含注释，但是可以包含类、字段、方法和代码属性，这些属性可用于将附加信息关联到这些元素。自从在Java 5中引入了注释(可以用于相同的目的)以来，属性几乎变得毫无用处。

- 编译后的类不包含包和导入部分，因此所有类型名必须是完全限定的。

另一个非常重要的结构差异是，编译后的类包含常量池部分。此池是一个数组，包含类中出现的所有数字、字符串和类型常量。这些常量只定义了一次，在常量池部分中，并由它们在类文件的所有其他部分中的索引引用。 ASM隐藏了与常量池相关的所有细节，因此您不必为此烦恼。下图总结了一个已编译类的总体结构。Java虚拟机规范第4节描述了确切的结构。

![wjzAud.png](https://s1.ax1x.com/2020/09/23/wjzAud.png)

在许多情况下，类型被限制为类或接口类型。例如，类的超类、类实现的接口或方法抛出的异常不能是基本类型或数组类型，而必须是类或接口类型。这些类型在具有内部名称的已编译类中表示。类的内部名称只是该类的完全限定名，其中点被替换为斜线。例如，字符串的内部名称是java/lang/String。


**基本类型**的描述符是单个字符:Z是布尔型，C是char型，B是byte型，S是short型，I是int型，F是float型，J是long型，D是double型。类类型的描述符是这个类的内部名称，前面是L，后面是分号。例如，String的类型描述符是Ljava/lang/String;。最后，数组类型的描述符是一个方括号，后面跟着数组元素类型的描述符。

![wjzHat.png](https://s1.ax1x.com/2020/09/23/wjzHat.png)

**方法描述符**是用一个字符串描述方法的参数类型和返回类型的类型描述符列表。方法描述符与左括号开始,其次是每个形式参数的类型描述符,紧随其后的是一个右括号,紧随其后的类型描述符返回类型,或者V如果方法返回void方法描述符(不包含方法的名称或参数名称)。

![wjzOG8.png](https://s1.ax1x.com/2020/09/23/wjzOG8.png)

一旦了解了类型描述符的工作原理，理解方法描述符就很容易了。例如(I)I描述了一个方法，它接受一个int类型的参数并返回一个int。图2.3给出了几个方法描述符示例。

### Interfaces and components

用于生成和转换已编译类的ASM API基于ClassVisitor抽象类。使用一个方法调用访问简单的部分，该方法调用的参数描述了它们的内容，并且返回void。内容可以是任意长度和复杂性的部分，通过返回一个辅助visitor类的初始方法调用进行访问。

 visitAnnotation、visitField和visitMethod方法就是这样，它们分别返回一个AnnotationVisitor、一个FieldVisitor和一个MethodVisitor。

![wvpl0s.png](https://s1.ax1x.com/2020/09/23/wvpl0s.png)

返回一个辅助的AnnotationVisitor，如在ClassVisitor中。这些辅助访问器的创建和使用将在下一章进行解释:实际上，本章仅限于可以单独使用ClassVisitor类解决的简单问题。
![wvpUcF.png](https://s1.ax1x.com/2020/09/23/wvpUcF.png)

ClassVisitor类的方法必须按照以下顺序调用，在这个类的Javadoc中指定:

![wvpg1O.png](https://s1.ax1x.com/2020/09/23/wvpg1O.png)

这意味着必须首先调用visit，然后最多调用一个visitSource，然后最多调用一个visitOuterClass，然后对visitAnnotation和visitAttribute的任意数量的调用，然后是任意数量的调用，以任何顺序访问visitInnerClass、visitField和visitMethod，并通过访问visitEnd的单个调用终止。

ASM提供了三个基于ClassVisitor API的核心组件来生成和转换类，分别是**ClassReader、ClassWriter、ClassVisitor**:
- ClassReader类解析以字节数组形式给出的已编译类，并对作为参数传递给它的accept方法的ClassVisitor实例调用相应的visitXxx方法。它可以被看作是一个事件生成器。
- ClassWriter类是ClassVisitor抽象类的子类，该类直接以二进制形式构建已编译类。它产生一个包含编译类的字节数组作为输出，可以使用toByteArray方法检索。它可以被视为事件使用者。
-  ClassVisitor类将它接收到的所有方法调用委托给另一个ClassVisitor实例。它可以被看作是一个事件过滤器。

接下来的部分将通过具体示例展示如何使用这些组件来生成和转换类。

#### 解析类

解析现有类所需的惟一组件是ClassReader组件。让我们举一个例子来说明这一点。假设我们希望以类似于javap工具的方式打印类的内容。第一步是编写ClassVisitor类的子类，它打印有关它访问的类的信息。例如：
```
public class ClassPrinter extends ClassVisitor {

    public ClassPrinter() {
        super(ASM4);
    }

    public void visit(int version, int access, String name, String signature, String superName, String[] interfaces) {
        System.out.println(name + " extends " + superName + " {");
    }

    public void visitSource(String source, String debug) {
    }

    public void visitOuterClass(String owner, String name, String desc) {
    }

    public AnnotationVisitor visitAnnotation(String desc, boolean visible) {
        return null;
    }

    public void visitAttribute(Attribute attr) {
    }

    public void visitInnerClass(String name, String outerName,
                                String innerName, int access) {
    }

    public FieldVisitor visitField(int access, String name, String desc,
                                   String signature, Object value) {
        System.out.println(" " + desc + " " + name);
        return null;
    }

    public MethodVisitor visitMethod(int access, String name,
                                     String desc, String signature, String[] exceptions) {
        System.out.println(" " + name + desc);
        return null;
    }

    public void visitEnd() {
        System.out.println("}");
    }
}
```
第二步是将这个ClassPrinter和一个ClassReader组件结合起来，这样由ClassReader产生的事件就会被我们的ClassPrinter使用:
```
ClassPrinter cp = new ClassPrinter();
ClassReader cr = new ClassReader("java.lang.Runnable");
cr.accept(cp, 0);
```
第二行创建了一个ClassReader来解析Runnable类。最后一行调用的accept方法解析可运行类字节码，并在cp上调用相应的ClassVisitor方法，结果如下:
```
java/lang/Runnable extends java/lang/Object { 
    run()V 
}
```
请注意，有几种方法可以构造ClassReader实例。必须读取的类可以按名称(如上所述)指定，也可以按值指定，作为字节数组或InputStream。读取类内容的输入流可以通过类加载器(ClassLoader)的getResourceAsStream方法获得:
```
cl.getResourceAsStream(classname.replace(’.’, ’/’) + ".class");
```
#### 生成类

生成类所需的唯一组件是ClassWriter组件。让我们举一个例子来说明这一点。考虑以下接口:
```
package pkg;

public interface Comparable extends Mesurable {
    int LESS = -1;
    int EQUAL = 0;
    int GREATER = 1;
    int compareTo(Object o);
}
```
它可以通过对一个ClassVisitor的六个方法调用来生成:
```
    ClassWriter cw = new ClassWriter(0);
    cw.visit(V1_5, ACC_PUBLIC + ACC_ABSTRACT + ACC_INTERFACE,"pkg/Comparable", null, "java/lang/Object",new String[] { "pkg/Mesurable" });
    cw.visitField(ACC_PUBLIC + ACC_FINAL + ACC_STATIC, "LESS", "I",null, new Integer(-1)).visitEnd();
    cw.visitField(ACC_PUBLIC + ACC_FINAL + ACC_STATIC, "EQUAL", "I",null, new Integer(0)).visitEnd();
    cw.visitField(ACC_PUBLIC + ACC_FINAL + ACC_STATIC, "GREATER", "I",null, new Integer(1)).visitEnd();
    cw.visitMethod(ACC_PUBLIC + ACC_ABSTRACT, "compareTo","(Ljava/lang/Object;)I",null,null).
    visitEnd(); cw.visitEnd();
    byte[] b = cw.toByteArray();
```
 第一行创建了一个ClassWriter实例，它将实际构建类的字节数组表示(构造器参数将在下一章中解释)。

对visit方法的调用定义了类头部。V1_5参数是在ASM操作码接口中定义的常量，就像所有其他ASM常量一样。它指定了类版本，Java 1.5。ACC_XXX常量是对应于Java修饰符的标志。这里我们指定这个类是一个接口，并且它是公共的和抽象的(因为它不能被实例化)。下一个参数以内部形式指定类名。还记得吗，已编译的类不包含包或导入部分，因此所有类名必须是完全限定的。下一个论点是相关的

接下来对visitField的三个调用类似，它们用于定义这三个接口字段。第一个参数是一组与Java修饰符对应的标志。这里我们指定字段为公共、final和静态。第二个参数是字段的名称，就像它在源代码中出现的那样。第三个参数是字段的类型，以类型描述符的形式。这里的字段是int字段，描述符是i。第四个参数对应于泛型。在我们的例子中，它是null，因为字段类型没有使用泛型。最后一个参数是字段的常数值:这个参数visitMethod调用用于定义compareTo方法。这里第一个参数也是一组与Java修饰符对应的标志。第二个参数是方法名，就像它在源代码中出现的那样。第三个参数是方法的描述符。第四个参数对应于泛型。在我们的例子中，它是null，因为方法没有使用泛型。最后一个参数是可以由方法抛出的异常数组，由异常的内部名称指定。这里是null，因为方法没有声明任何异常。visitMethod方法返回一个MethodVisitor。
最后，使用对visitEnd的最后一个调用来通知cw类已经完成，并使用对toByteArray的调用来检索它作为字节数组。

##### 使用生成的类

前面的字节数组可以存储在Comparable.class文件中以供将来使用。或者，也可以用类加载器动态加载它。一种方法是定义一个类加载器子类，它的defineClass方法是public的:
```
class MyClassLoader extends ClassLoader { 
	public Class defineClass(String name, byte[] b) { 
		return defineClass(name, b, 0, b.length); 
	} 
}
```
然后生成的类可以直接加载:
```
Class c = myClassLoader.defineClass("pkg.Comparable", b);
```
另一种加载生成类的方法是定义一个类加载器子类，它的findClass方法被重写，以便动态生成所请求的类:
```
class StubClassLoader extends ClassLoader {

    @Override
    protected Class findClass(String name) throws ClassNotFoundException {
        if (name.endsWith("_Stub")) {
            ClassWriter cw = new ClassWriter(0); 
            ...
            byte[] b = cw.toByteArray();
            return defineClass(name, b, 0, b.length);
        } return super.findClass(name);

    }

}
```
实际上，使用生成的类的方式取决于上下文超出ASM API的范围。如果你在写一个编译器，类的抽象语法树将驱动生成过程要编译的程序，生成的类将存储在磁盘上。如果您正在编写将使用的动态代理类生成器或aspect weaver，以这样或那样的方式，一个类加载器。

#### 转换类

到目前为止，ClassReader和ClassWriter组件是单独使用的。事件是“手工”生成并由ClassWriter直接使用的，或者，同样地，它们是由ClassReader生成并“手工”使用的，即由自定义ClassVisitor实现。当这些组件一起使用时，事情开始变得非常有趣。第一步是将ClassReader生成的事件定向到ClassWriter。其结果是类阅读器解析的类被类编写器重构:

```
        byte[] b1 = ...;
        ClassWriter cw = new ClassWriter(0);
        ClassReader cr = new ClassReader(b1);
        cr.accept(cw, 0);
        byte[] b2 = cw.toByteArray(); // b2 represents the same class as b1
```
这本身并不是很有趣(复制字节数组有更简单的方法!)，但是请稍等。下一步是在类阅读器和类编写器之间引入一个类访问器:
```
        byte[] b1 = ...;
        ClassWriter cw = new ClassWriter(0); // cv forwards all events to cw
        ClassVisitor cv = new ClassVisitor(ASM4, cw) {};
        ClassReader cr = new ClassReader(b1);
        cr.accept(cv, 0);
        byte[] b2 = cw.toByteArray(); // b2 represents the same class as b1
```
下图描述了与上述代码相对应的架构，其中组件用正方形表示，事件用箭头表示(在序列图中使用垂直时间线)。
![wzpbjO.png](https://s1.ax1x.com/2020/09/24/wzpbjO.png)

但是，结果不会改变，因为ClassVisitor事件过滤器不会过滤任何东西。但是现在，通过覆盖一些方法，过滤一些事件以便能够转换类就足够了。例如，考虑下面的ClassVisitor子类:

```

public class ChangeVersionAdapter extends ClassVisitor {

    public ChangeVersionAdapter(ClassVisitor cv) {
        super(ASM4, cv);
    }

    @Override
    public void visit(int version, int access, String name,
                      String signature, String superName, String[] interfaces) {
        cv.visit(V1_5, access, name, signature, superName, interfaces);
    }

}
```

这个类只覆盖ClassVisitor类的一个方法。因此，除了对visit方法的调用外，所有的调用都被不加更改地转发给传递给构造函数的类访问器cv。其序列图如图2.7所示。

![wz9FKS.png](https://s1.ax1x.com/2020/09/24/wz9FKS.png)

通过修改visit方法的其他参数，您可以实现不仅仅是更改类版本的其他转换。例如，您可以将接口添加到已实现接口列表中。也可以更改类的名称，但这需要的不仅仅是更改visit方法中的name参数。实际上，类的名称可以出现在编译后的类中的许多不同位置，而且必须更改所有这些出现的名称才能真正重命名类。

##### 最优化


前面的转换只改变了原始类中的四个字节。但是，通过上面的代码，b1被完全解析，相应的事件被用来从头构造b2，这不是很有效。复制b1中没有直接转换为b2的部分，而不解析这些部分，也不生成相应的事件，这样效率会高得多。ASM自动执行这种优化方法:


- 如果ClassReader组件检测到作为参数传递给它的accept方法的ClassVisitor返回的MethodVisitor来自ClassWriter，这意味着这个方法的内容将不会被转换，实际上应用程序甚至不会看到它。

- 在这种情况下，ClassReader组件没有解析此方法的内容，没有生成相应的事件，只需在ClassWriter中复制此方法的字节数组表示。

这个优化是由ClassReader和ClassWriter组件执行的，如果它们彼此有一个引用，可以这样设置:
```
        byte[] b1 = ...;
        ClassReader cr = new ClassReader(b1);
        ClassWriter cw = new ClassWriter(cr, 0);
        ChangeVersionAdapter ca = new ChangeVersionAdapter(cw);
        cr.accept(ca, 0);
        byte[] b2 = cw.toByteArray();
```
由于这种优化，上面的代码比之前的代码快两倍，因为ChangeVersionAdapter不转换任何方法。对于转换部分或所有方法的普通类转换，其加速速度要小一些，但仍然是显而易见的:它确实是10到20%的量级。不幸的是，这种优化需要将原始类中定义的所有常量复制到转换后的类中。对于添加字段、方法或指令的转换来说，这不是问题，但是对于转换来说，与未优化的情况相比，这会导致更大的类文件。因此，只能对“加法”转换使用这种优化。


#####  使用转换类

转换后的类b2可以存储在磁盘上，也可以用类加载器加载，如前一节所述。但是在类装入器内完成的类转换只能转换由这个类装入器装入的类。如果您想转换所有的类，那么您必须将您的转换放到一个ClassFileTransformer中，正如java.lang.instrument中定义的那样。

```
    public static void premain(String agentArgs, Instrumentation inst) {
        inst.addTransformer(new ClassFileTransformer() {
            public byte[] transform(ClassLoader l, String name, Class c, ProtectionDomain d, byte[] b) throws IllegalClassFormatException {
                ClassReader cr = new ClassReader(b);
                ClassWriter cw = new ClassWriter(cr, 0);
                ClassVisitor cv = new ChangeVersionAdapter(cw);
                cr.accept(cv, 0);
                return cw.toByteArray();
            }
        });
    }
```
#### 删除类成员

在前一节中用于转换类版本的方法可以应用于ClassVisitor类的其他方法。通过更改visitField中的access或name参数visitMethod方法，您可以更改修饰符或字段或方法的名称。此外，没有转发一个方法调用与modified  参数，您可以选择根本不转发此调用。结果是  相应的类元素被删除。

例如，下面的类适配器删除了关于外部类和内部类的信息，以及编译类的源文件的名称(生成的类仍然是功能完整的，因为这些元素仅用于调试目的)。这是通过不转发任何东西在适当的访问方法:

```
    public class RemoveDebugAdapter extends ClassVisitor {

        public RemoveDebugAdapter(ClassVisitor cv) {
            super(ASM4, cv);
        }

        @Override
        public void visitSource(String source, String debug) {
        }

        @Override
        public void visitOuterClass(String owner, String name, String desc) {
        }

        @Override
        public void visitInnerClass(String name, String outerName,
                                    String innerName, int access) {
        }

    }
```
此策略不适用于字段和方法，因为visitField和visitMethod方法必须返回一个结果。为了删除字段或方法，您必须不转发方法调用，并向调用者返回null。例如，下面的类适配器通过它的名称和描述符(名称不足以识别一个方法，因为一个类可以包含几个相同名称但具有不同参数的方法)来移除一个方法:

```
    public class RemoveMethodAdapter extends ClassVisitor {
        private String mName;
        private String mDesc;

        public RemoveMethodAdapter(ClassVisitor cv, String mName, String mDesc) {
            super(ASM4, cv);
            this.mName = mName;
            this.mDesc = mDesc;
        }

        @Override
        public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
            if (name.equals(mName) && desc.equals(mDesc)) { // do not delegate to next visitor -> this removes the method
                return null;
            }
            return cv.visitMethod(access, name, desc, signature, exceptions);
        }
    }
```

#### 添加类成员
您可以“转发”更多的调用，而不是转发比接收到的更少的调用，这将产生添加类元素的效果。新调用可以插入到原始方法调用之间的多个位置，前提是必须遵守调用各个visitXxx方法的顺序

例如，如果您想向类添加一个字段，就必须在原始方法调用之间插入对visitField的新调用，并且必须将这个新调用放入类适配器的一个访问方法中。例如，您不能在visit方法中这样做，因为这可能导致调用visitField，然后调用visitSource、visitOuterClass、visitAnnotation或visitAttribute，这些都是无效的。出于同样的原因，不能将这个新调用放到visitSource、visitOuterClass、visitAnnotation或visitAttribute方法中。唯一可能的是visitInnerClass、visitField、visitMethod或visitEnd方法。

如果您将新的调用放入visitEnd方法中，字段将始终被添加(除非您添加显式条件)，因为该方法始终被调用。如果您将它放在visitField或visitMethod中，将会添加几个字段:原始类中的每个字段或每个方法一个。两种解决方案都有道理;这取决于你需要什么。例如，您可以添加一个计数器字段来计算对象上的调用，或者为每个方法添加一个计数器来分别计算每个方法的调用。

Note:实际上，唯一真正正确的解决方案是通过在visitEnd方法中进行额外调用来添加新成员。事实上，一个类不能包含重复的成员，确保新成员唯一的唯一方法是将它与所有现有成员进行比较，只有在所有成员都被访问后才能进行比较，即在visitEnd方法中这是相当有限的。使用生成的不太可能被程序员使用的名称，例如_counter$或_4B7F_，在实践中足以避免重复成员，而不必在visitEnd中添加它们。请注意，正如在第一章中所讨论的，tree API没有这个限制:可以在使用该API的转换中随时添加新成员。

为了说明上面的讨论，这里有一个类适配器，它向类添加一个字段，除非这个字段已经存在:
```
    public class AddFieldAdapter extends ClassVisitor {
        private int fAcc;
        private String fName;
        private String fDesc;
        private boolean isFieldPresent;
        public AddFieldAdapter(ClassVisitor cv, int fAcc, String fName, String fDesc) {
            super(ASM4, cv);
            this.fAcc = fAcc;
            this.fName = fName;
            this.fDesc = fDesc;
        }

        @Override
        public FieldVisitor visitField(int access, String name, String desc, String signature, Object value) {
            if (name.equals(fName)) {
                isFieldPresent = true;
            }
            return cv.visitField(access, name, desc, signature, value);
        }

        @Override
        public void visitEnd() {
            if (!isFieldPresent) {
                FieldVisitor fv = cv.visitField(fAcc, fName, fDesc, null, null);
                if (fv != null) {
                    fv.visitEnd();
                }
            }
            cv.visitEnd();
        }
    }
```
字段被添加到visitEnd方法中。visitField方法不是为了修改现有字段或删除字段而重写的，而是为了检测要添加的字段是否已经存在。注意 fv!=null 在fv.viditEnd()调用前进行判断，这是因为，正如我们在上一节中所看到的，类访问者可以在visitField中返回null。

#### 转换链

到目前为止，我们已经看到了由类读取器、类适配器和类编写器组成的简单转换链。当然，也可以使用更复杂的链，将几个类适配器链接在一起。链接几个适配器允许您组合几个独立的类转换，以便执行复杂的转换。还要注意，变换链不一定是线性的。你可以编写一个ClassVisitor，它可以同时将所有接收到的方法调用转发给几个ClassVisitor:
```
    public class MultiClassAdapter extends ClassVisitor {
        protected ClassVisitor[] cvs;
        public MultiClassAdapter(ClassVisitor[] cvs) {
            super(ASM4);
            this.cvs = cvs;
        }

        @Override
        public void visit(int version, int access, String name,
                          String signature, String superName, String[] interfaces) {
            for (ClassVisitor cv : cvs) {
                cv.visit(version, access, name, signature, superName, interfaces);
            }
        } ...
    }
```

对称地，几个类适配器可以委托给同一个类访问器(这需要一些预防措施来确保，例如，在这个类访问器上只调用了一次visit和visitEnd方法)。

#### 工具

除了ClassVisitor类和相关的ClassReader和ClassWriter组件之外，ASM还在org.objectweb.asm中提供。util包，这是几个在类生成器或适配器开发期间有用的工具，但在运行时不需要它们。ASM还提供了一个实用程序类，用于在运行时操作内部名称、类型描述符和方法描述符。下面介绍了所有这些工具。

![wzPKnU.png](https://s1.ax1x.com/2020/09/24/wzPKnU.png)

##### 类型

正如您在前面几节中看到的，ASM API公开了存储在编译类中的Java类型，即作为内部名称或类型描述符。当它们在源代码中出现时，可以将它们公开，以使代码更具可读性。但是这需要在ClassReader和ClassWriter中的两种表示之间进行系统转换，这会降低性能。这就是ASM不透明地将内部名称和类型描述符转换为它们等效的源代码形式的原因。不过，它提供了类型类，以便在必要时手动完成该操作。

类型对象表示Java类型，可以从类型描述符或类对象构造。类型类还包含表示基本类型的静态变量。例如类型。INT_TYPE是表示int类型的类型对象。

getInternalName方法返回类型的内部名称。例如Type.getType(String.class). getinternalname()提供了字符串类的内部名称，即。“java / lang / String”。此方法只能用于类或接口类型。

getDescriptor方法返回类型的描述符。因此，例如，在代码中可以使用Type.getType(String.class). getdescriptor()而不是"Ljava/lang/String;"。或者，您可以使用Type.INT_TYPE.getDescriptor()来代替I。

类型对象也可以表示方法类型。这样的对象可以从方法描述符或方法对象构造。getDescriptor方法然后返回与此类型对应的方法描述符。另外，可以使用getArgumentTypes和getReturnType方法来获取方法的参数类型和返回类型对应的类型对象。例如Type.getArgumentTypes("(I)V")返回值包含单个元素的数组Type.INT_TYPE。 类似地，调用Type.getReturnType("(I)V")返回Type.VOID_TYPE对象。

##### TraceClassVisitor

为了检查生成的或转换的类是否符合您的期望，ClassWriter返回的字节数组并不是真正有用的，因为它是不可读的。文本表示更容易使用。这就是TraceClassVisitor类所提供的内容。这个类，顾名思义，扩展了ClassVisitor类，并构建被访问类的文本表示。因此，您可以使用TraceClassVisitor，而不是使用ClassWriter来生成类，以获得实际生成内容的可读跟踪。或者，更好的是，你可以同时使用这两种方法。实际上，除了默认行为之外，TraceClassVisitor还可以将对其方法的所有调用委托给另一个访问者，例如ClassWriter:
```
		ClassWriter cw = new ClassWriter(0);
        TraceClassVisitor cv = new TraceClassVisitor(cw, printWriter);
        cv.visit(...); 
        ...
        cv.visitEnd();
        byte b[] = cw.toByteArray();
```
这段代码创建了一个TraceClassVisitor，它将接收到的所有调用委托给cw，并将这些调用的文本表示输出给printWriter。例如，在2.2.3节的示例中使用TraceClassVisitor会给出:

```
    // class version 49.0 (49)
    //access flags 1537
    public abstract interface pkg/Comparable implements pkg/Mesurable
    // access flags 25
    public final static I LESS = -1
    //access flags 25
    public final static I EQUAL = 0
    // access flags 25
    public final static I GREATER = 1
    // access flags 1025
    //public abstract compareTo(Ljava/lang/Object;)I
```

 请注意，您可以在生成链或转换链的任何点上使用TraceClassVisitor，而不仅仅是在ClassWriter之前，以便查看在链的这个点上发生了什么,还请注意，通过String.equals()，这个适配器生成的类的文本表示可以方便地用于比较类。
 
##### CheckClassAdapter

ClassWriter类不检查它的方法是否以适当的顺序和有效的参数被调用。因此，有可能生成将被Java虚拟机验证器拒绝的无效类。为了尽快检测其中一些错误，可以使用CheckClassAdapter类。与TraceClassVisitor类似，这个类扩展了ClassVisitor类，并将对其方法的所有调用委托给另一个ClassVisitor，例如TraceClassVisitor或ClassWriter。但是，这个类不是打印被访问类的文本表示，而是在委托给下一个访问器之前，检查它的方法是否按适当的顺序调用，并带有有效的参数。在出现错误时，抛出IllegalStateException或IllegalArgumentException。

为了检查一个类，打印这个类的文本表示，最后创建一个字节数组表示，你应该使用如下代码:
```
        ClassWriter cw = new ClassWriter(0);
        TraceClassVisitor tcv = new TraceClassVisitor(cw, printWriter);
        CheckClassAdapter cv = new CheckClassAdapter(tcv);
        cv.visit(...); 
        ...
        cv.visitEnd();
        byte b[] = cw.toByteArray();
```

请注意，如果您将这些类访问者以不同的顺序链接起来，那么它们执行的操作也将以不同的顺序执行。例如，使用以下代码，检查将在跟踪之后发生:
```
        ClassWriter cw = new ClassWriter(0);
        CheckClassAdapter cca = new CheckClassAdapter(cw);
        TraceClassVisitor cv = new TraceClassVisitor(cca, printWriter);
```
与TraceClassVisitor类似，您可以在生成链或转换链的任何点(而不仅仅是在ClassWriter之前)使用CheckClassAdapter，以便在链的这个点上检查类。

##### ASMifier

这个类为TraceClassVisitor工具提供了一个备用的后端（默认情况下，它使用textier后端，产生上面所示的那种输出）。这个后端使TraceClassVisitor类的每个方法打印用于调用它的Java代码。例如，调用visitEnd()方法输出cv.visitEnd();。其结果是，当具有ASMifier后端的TraceClassVisitor访问一个类时，它打印源代码来用ASM生成这个类。如果您使用此访问器访问已经存在的类，这将非常有用。例如，如果您不知道如何用ASM生成一些已编译的类，那么编写相应的源代码，用javac编译它，并用ASMifier访问已编译的类。您将得到ASM代码来生成这个编译类!

ASMifier类可以在命令行中使用。例如使用:
```
java -classpath asm.jar:asm-util.jar \ org.objectweb.asm.util.ASMifier \ java.lang.Runnable
```
产生缩进后的代码:
```
package asm.java.lang;
import org.objectweb.asm.*;

public class RunnableDump implements Opcodes {

    public static byte[] dump() throws Exception {
        ClassWriter cw = new ClassWriter(0);
        FieldVisitor fv;
        MethodVisitor mv;
        AnnotationVisitor av0;
        cw.visit(V1_5, ACC_PUBLIC + ACC_ABSTRACT + ACC_INTERFACE, "java/lang/Runnable", null, "java/lang/Object", null);
        {
            mv = cw.visitMethod(ACC_PUBLIC + ACC_ABSTRACT, "run", "()V", null, null);
            mv.visitEnd();
        }
        cw.visitEnd();
        return cw.toByteArray();
    }
}
```

#### Methods

章解释了如何用核心ASM API生成和转换编译后的方法。首先介绍编译后的方法，然后介绍相应的ASM接口、组件和生成和转换ASM的工具，以及许多说明性示例。

##### Structure

在已编译的类中，方法的代码以字节码指令序列的形式存储。为了生成和转换类，了解这些指令并理解它们是如何工作的是非常重要的。本节概述了这些说明，这些说明对于开始编写简单的类生成器和转换器已经足够了。要获得完整的定义，您应该阅读Java虚拟机规范。


##### 执行模型

在展示字节码指令之前，有必要展示Java虚拟机执行模型。正如您所知道的，Java代码是在线程中执行的。每个线程都有自己的执行堆栈，它是由帧组成的。每一帧代表一个方法调用:每次调用一个方法时，一个新的帧被推入当前线程的执行堆栈。当方法返回时，通常或由于异常，该框架将从执行堆栈中弹出，并在调用方法中继续执行(其框架现在位于堆栈之上)。

每一帧包含两部分:局部变量部分和操作数堆栈部分。局部变量部分包含可以通过其索引访问的变量，并按随机顺序访问。操作数堆栈部分，顾名思义，是字节码指令用作操作数的值堆栈。这意味着该堆栈中的值只能按后进先出顺序访问。不要混淆操作数堆栈和线程的执行堆栈:执行堆栈中的每一帧都包含自己的操作数堆栈。

局部变量和操作数堆栈部分的大小取决于方法的代码。它是在编译时计算的，并与字节码指令一起存储在已编译的类中。因此，对应于调用给定方法的所有帧都具有相同的大小，但是对应于不同方法的帧对于其局部变量和操作数堆栈部分可能具有不同的大小。

![wz3JXD.png](https://s1.ax1x.com/2020/09/24/wz3JXD.png)

图3.1显示了一个包含3帧的示例执行堆栈。第一个帧包含3个局部变量，它的操作数栈的最大大小为4，并且它包含两个值。第二帧包含两个局部变量，以及操作数堆栈中的两个值。最后，在执行堆栈之上的第三帧包含4个局部变量和两个操作数。

在创建框架时，使用空堆栈初始化框架，并使用目标对象this(对于非静态方法)和方法的参数初始化其局部变量。例如，调用方法a.equals(b)将创建一个框架，该框架包含一个空堆栈，并且前两个局部变量初始化为a和b(其他局部变量未初始化)。

局部变量和操作数堆栈部分中的每个槽都可以保存任何Java值，长值和双精度值除外。这些值需要两个槽。这使得局部变量的管理变得复杂:例如第i个方法参数不一定存储在局部变量i中。max(1L, 2L)创建一个帧，其1L值位于前两个局部变量插槽，2L值位于第三和第四插槽。

##### 字节码指令

一个字节码指令是由一个操作码来识别这条指令，和一个固定数量的参数:

- 操作码是无符号字节值—因此是字节码名，并由助记符标识。例如，操作码值0是由助记符号NOP设计的，它对应于不执行任何操作的指令。
- 参数是定义精确指令行为的静态值。它们是在操作码之后给出的。例如，操作码值为167的GOTO标签指令以指定下一条要执行的指令的标签作为参数标签。不能将指令参数与指令操作数混淆:参数值是静态已知的，并存储在编译后的代码中，而操作数值来自操作数堆栈，仅在运行时才知道。

字节码指令可以分为两类:一小组指令被设计用来将值从本地变量转移到操作数堆栈，反之亦然;其他指令仅作用于操作数堆栈:它们从堆栈中弹出一些值，根据这些值计算结果，然后将其推回堆栈。

 ILOAD、LLOAD、FLOAD、DLOAD和ALOAD指令读取局部变量并将其值压入操作数堆栈。它们将必须读取的局部变量的索引i作为参数。ILOAD用于加载布尔型、字节型、char型、short型或int型局部变量。LLOAD、FLOAD和DLOAD分别用于加载一个长值、浮点值或双精度值(LLOAD和DLOAD实际上加载了两个插槽i和i+1)。最后，使用ALOAD加载任何非原始值，如对象和数组引用。对称地，ISTORE, LSTORE, FSTORE, DSTORE和ASTORE指令从操作数堆栈中弹出一个值，并将其存储在由其索引i指定的局部变量中。

正如您所看到的，xLOAD和xSTORE指令是键入的(实际上，正如您将在下面看到的，几乎所有指令都是键入的)。这用于确保没有进行非法转换。实际上，将值存储在局部变量中，然后用另一种类型加载它是非法的。例如，ISTORE 1 ALOAD 1序列是非法的—它允许在本地变量1中存储任意的内存地址，并将该地址转换为对象引用!然而，在局部变量中存储其类型与该局部变量中存储的当前值的类型不同的值是完全合法的。 局部变量的类型，即存储在这个局部变量中的值的类型，可以在方法执行期间改变。

如上所述，所有其他字节码指令仅在操作数堆栈上工作。它们可分为下列类别:

**Stack** &nbsp; &nbsp; 这些指令用于操作堆栈上的值:POP将值弹出到堆栈顶部，DUP将顶部堆栈值的一个副本推送，SWAP弹出两个值并将它们按相反的顺序推送，等等。

**Constants** &nbsp; &nbsp; 这些指令操作数堆栈上推一个常数值:ACONST_NULL把null, ICONST_0推int值0,FCONST_0推动0 f, DCONST_0推动0 d, BIPUSH b将字节值b, SIPUSH推动短期价值年代,LDC春秋国旅将任意整数,浮点数、长,双,字符串,或类1常数cst,等等。

**Arithmetic and logic** &nbsp; &nbsp; 这些指令从操作数堆栈中弹出数值并组合它们并将结果压入堆栈。他们没有任何争论。xADD、xSUB、xMUL、xDIV和xREM对应于+、-、*、/和%操作，其中x是I、L、F或D。类似地，还有其他相应的指令<<, >>, >>>, |, & and ^,用于int和long值。

**Casts** &nbsp; &nbsp; 这些指令从堆栈中弹出一个值，将其转换为另一种类型，然后将结果推回去。它们对应于Java中的强制转换表达式。I2F、F2D、L2D等将数值从一种数值类型转换为另一种数值类型。将引用值转换为类型t。

**Objects** &nbsp; &nbsp; 这些指令用于创建对象、锁定它们、测试它们的类型，等等。例如，NEW type指令将类型为type的新对象推入堆栈(其中type是一个内部名称)。

**Fields** &nbsp; &nbsp; 这些指令读取或写入字段的值。GETFIELD所有者名称desc弹出一个对象引用，并推入其名称字段的值。PUTFIELD所有者名称desc弹出一个值和一个对象引用，并将这个值存储在它的name字段中。在这两种情况下，对象的类型必须是owner，其字段的类型必须是desc. GETSTATIC和PUTSTATIC是类似的指令，但对于静态字段。

**Methods** &nbsp; &nbsp; 这些指令调用一个方法或构造函数。它们弹出与方法参数一样多的值，再加上目标对象的一个值，然后推入方法调用的结果。INVOKEVIRTUAL调用类owner中定义的name方法，其方法描述符为desc。 INVOKESTATIC用于静态方法，invokspecialial用于私有方法和构造函数，INVOKEINTERFACE用于在接口中定义的方法。最后，对于Java 7类，INVOKEDYNAMIC用于新的动态方法调用机制。 

**Arrays** &nbsp; &nbsp; 这些指令用于读取和写入数组中的值。xALOAD指令弹出一个索引和数组，并将数组元素的值压入该索引处。xASTORE指令弹出一个值、一个索引和一个数组，并将该值存储在数组的那个索引处。这里x可以是I L F D或A，也可以是B C或S。 

**Jumps** &nbsp; &nbsp; 如果某些条件为真或无条件，这些指令会跳转到任意指令。它们用于编译if、for、do、while、break和continue指令。例如，IFEQ label从堆栈中弹出一个int值，如果该值为0，则跳转到label设计的指令(否则继续正常执行下一条指令)。还有许多其他跳转指令，如IFNE或IFGE。最后，TABLESWITCH和LOOKUPSWITCH对应于switch的Java指令。

**Return** &nbsp; &nbsp; 最后，xRETURN和RETURN指令用于终止方法的执行，并将结果返回给调用者。RETURN用于返回void的方法，xRETURN用于其他方法。 

##### Examples

<b>未完待续








 
 
 
 
 
 
 
 
 
 
