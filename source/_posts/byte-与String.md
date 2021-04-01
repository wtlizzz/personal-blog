title: 'byte[]与String'
author: Wtli
tags:
  - Java
categories:
  - 后端
date: 2020-11-19 16:59:00
---
记录一次byte\[]与String转换遇到的坑。

<!--more-->

### byte[]简述

使用Java Nio将byte\[]转换成String。主流的转换方式有如下三种：
```
byte[] bytes = {'1', '2', '3'};
log.debug(new String(bytes));
log.debug(String.valueOf(bytes));
log.debug(bytes.toString());
```
输出：
```
17:06:15.300 [main] DEBUG com.example.nioChannel.server.SocketClientServer - 123
17:06:15.303 [main] DEBUG com.example.nioChannel.server.SocketClientServer - [B@14514713
17:06:15.303 [main] DEBUG com.example.nioChannel.server.SocketClientServer - [B@14514713
```

调试，查看byte\[]中的数据：

![upload successful](/images/pasted-7.png)


在byte\[]中存储的是字符的ascii码。

**String的本质是 private final char value\[];
例如调用String.indexOf方法，就会根据char\[] value来进行数组的查找。**

#### 类别一：new String(bytes)
```
    public String(byte bytes[], int offset, int length) {
        checkBounds(bytes, offset, length);
        this.value = StringCoding.decode(bytes, offset, length);
    }
```
new String(bytes)方法调用了一个解码的操作，StringCoding.decode()。
```
    static char[] decode(byte[] ba, int off, int len) {
        String csn = Charset.defaultCharset().name();
        try {
            // use charset name decode() variant which provides caching.
            return decode(csn, ba, off, len);
        } catch (UnsupportedEncodingException x) {
            warnUnsupportedCharset(csn);
        }
        try {
            return decode("ISO-8859-1", ba, off, len);
        } catch (UnsupportedEncodingException x) {
            // If this code is hit during VM initialization, MessageUtils is
            // the only way we will be able to get any kind of error message.
            MessageUtils.err("ISO-8859-1 charset not available: "
                             + x.toString());
            // If we can not find ISO-8859-1 (a required encoding) then things
            // are seriously wrong with the installation.
            System.exit(1);
            return null;
        }
    }
```
其中，默认的Charset.defaultCharset().name()是UTF-8。之后就根据UTF-8的编码方式，将byte\[]进行解码，使用这种方式将byte\[]转化成string是正确的。

#### 类别二：String.valueOf(bytes)

String.valueOf是调用的object.toString的方法。

```
    public static String valueOf(Object obj) {
        return (obj == null) ? "null" : obj.toString();
    }
```
object.toString方法是输出的类名+hashCode。所以如果生成String类型，尽量不要用这个方法。
```
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```
#### 类别三：bytes.toString()

同类别二，直接调用的object.toString的方法。

### 其他
当使用boolean、char、int、long、float、double时，可以直接使用toString方法，他们分别调用的是Integer、Long、Float、Double封装类的toString()方法。
#### Integer

```
        Integer integer = 11;
        log.debug(integer.toString());
************************************************************        
17:28:08.635 [main] DEBUG com.example.nioChannel.server.SocketClientServer - 11
```
Integer.toString的方法时直接调用的new String(buf, true)。通过这种方式能够直接生成正确的String。
```
    public static String toString(int i) {
        if (i == Integer.MIN_VALUE)
            return "-2147483648";
        int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
        char[] buf = new char[size];
        getChars(i, size, buf);
        return new String(buf, true);
    }
```
另外使用String.valueOf(Integer)，也是调用的Integer.toString()。