title: Java Nio ServerSocketChannel原理
author: Wtli
tags:
  - Java NIO
  - ''
categories:
  - 后端
date: 2021-01-05 17:11:00
---
SelectionKey在编程过程中实际使用的并不多，但是这个抽象类确是在Java NIO Selector中很重要，在其他Java NIO中并没有太多的解释，下面来详细看一下。

<!-- more -->

#### Selector

**是什么？**

```
A token representing the registration of a SelectableChannel with a Selector.
```
Selector是一种类似于token一样的记录数据。当SelectableChannel在Selector中注册时，会生成一个Selector。

```
A selection key is created each time a channel is registered with a selector.  A key remains valid until it is cancelled by invoking its cancel method, by closing its channel, or by closing its selector. 
```
当channel在selector上注册时，每次都会生成一个selection key，知道channel被关闭时才会销毁这个对应的key。

```
A channel may be registered at most once with any particular selector
```

一个channel只能绑定一个特定的selector。

#### SelectionKeyImpl

SelectionKeyImpl继承类SelectionKey，用来绑定channel和selector对象。

通过保存两个主要的数据channel和selector来实现绑定功能：
```
    final SelChImpl channel;                            // package-private
    public final SelectorImpl selector;
```


#### 流程图

有点复杂，简单画一下流程图来直观的看一下。

![sAoDXj.png](https://s3.ax1x.com/2021/01/06/sAoDXj.png)

下面来看一下从刚开始创建Channel到Selector.select()怎么个过程：

首先是创建channel和selector：
```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.configureBlocking(false);
serverSocketChannel.socket().bind(new InetSocketAddress(SERVER_PORT));
Selector selector = Selector.open();
serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
```

好，重点就是serverSocketChannel.register方法，将channel绑定到selector中，下面是ServerSocketChannel的继承关系：

![sATfat.png](https://s3.ax1x.com/2021/01/06/sATfat.png)

好第一个重点类：

#### AbstractSelectableChannel

serverSocketChannel.register就是调用的AbstractSelectableChannel.register：

```
            SelectionKey k = findKey(sel);
            if (k != null) {
                k.interestOps(ops);
                k.attach(att);
            }
            if (k == null) {
                // New registration
                synchronized (keyLock) {
                    if (!isOpen())
                        throw new ClosedChannelException();
                    k = ((AbstractSelector)sel).register(this, ops, att);
                    addKey(k);
                }
            }
            return k;
```

AbstractSelectableChannel类中有一个数据为：

**SelectionKey\[] keys**

首先是findKey通过查找SelectionKey\[] keys是否有key，如果有的话直接返回这个key。
第一次传入这个Channel时，没有key的话就到了SelectorImpl这个类。

这就是第二个重要的类

#### SelectorImpl

Base Selector implementation class。  
Selector基础的实现类。 

如果创建一个SelectionKey，就通过这个方法：
```
        SelectionKeyImpl k = new SelectionKeyImpl((SelChImpl)ch, this);
        k.attach(attachment);
        synchronized (publicKeys) {
            implRegister(k);
        }
        k.interestOps(ops);
        return k;
```

在创建SelectionKey的同时，还调用了implRegister(k)这个方法。这个方法在SelectorImpl中做了keys.add(SelectionKeyImpl ski)操作。

下面来详细介绍一下SelectorImpl这个类：

```
    // The set of keys with data ready for an operation
    protected Set<SelectionKey> selectedKeys;

    // The set of keys registered with this Selector
    protected HashSet<SelectionKey> keys;

    // Public views of the key sets
    private Set<SelectionKey> publicKeys;             // Immutable
    private Set<SelectionKey> publicSelectedKeys;     // Removal allowed, but not addition
```

这个类首先声明了四个Set，上面有英文的注释，但是我们直接看这几个Set被操作的时间点来进行对它的理解：
- selectedKeys：当连接传入时（selector.select()触发时），会去判断里面包含了对应的key，如果没有就添加进去，也就是说这个数据是根据连接的channel最先改变的。
- keys：当channel在selector中注册时（serverSocketChannel.register()）添加，当注册方法触发时，会在SelectorImpl和AbstractSelectableChannel两个类中进行保存。
- publicKeys：= Collections.unmodifiableSet(keys);
- publicSelectedKeys：= Util.ungrowableSet(selectedKeys);

这里有两个挺有意思的方法：  
Collections.unmodifiableSet和Util.ungrowableSet()

unmodifiableSet：不能修改的Set。当调用add，remove，clear时会直接报错。  
**但是，由于publicKeys = keys，也就是说，当keys发生改变时publicKeys才能改变。**

```
        public boolean add(E e) {
            throw new UnsupportedOperationException();
        }
        public boolean remove(Object o) {
            throw new UnsupportedOperationException();
        }
        public void clear() {
            throw new UnsupportedOperationException();
        }
        ······
```

ungrowableSet：和上面的unmodifiableSet类似，但是区别在于ungrowableSet能够进行删除，不能添加，代码如下。
**如上面所说的，publicSelectedKeys = selectedKeys，publicSelectedKeys是随着selectedKeys而变化，不能单独变化。**

```
public boolean remove(Object o)   { return s.remove(o); }
public boolean add(E o){
    throw new UnsupportedOperationException();
}
public boolean addAll(Collection<? extends E> coll) {
    throw new UnsupportedOperationException();
}
```

这么做的原因，上面源码中说的是想，把publicKeys和publicSelectedKeys暴漏出来，但不能修改，这样保护了keys和selectedKeys。（再深层次的原因不知道，可能只有写这代码的人知道吧～～）

#### 小结

上文中介绍了AbstractSelectableChannel和SelectorImpl两个类，都存储了SelectionKey，但是区别在哪里呢？

**区别：**

注册时：serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

AbstractSelectableChannel  SelectionKey\[] keys

| 类名 | 数据| 注册	| 连接建立	|  
|  -----  | -----  | ---- |----|
|AbstractSelectableChannel	|SelectionKey\[] keys|	add(key) ||
|SelectorImpl	|Set\<SelectionKey> publicKeys	|add(key)|
|SelectorImpl	|Set\<SelectionKey> publicSelectedKeys	||add(key)|

第一次注册创建的channel是@797，连接建立使用的channel也是@797，使用的是一条通道。
![sZIvan.png](https://s3.ax1x.com/2021/01/07/sZIvan.png)











































