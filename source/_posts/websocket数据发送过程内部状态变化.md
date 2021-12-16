title: websocket数据发送过程内部状态变化
author: Wtli
tags: []
categories: []
date: 2021-05-15 14:42:00
---
本文研究websocekt数据发送过程中，各个内部重要状态（属性）变化。

- state
- opCode
- fragmented
- text

<!--more-->

#### State

State是websocket内一个重要的状态变量，通过state来判断数据的发送完成状态，确定下一个数据的发送。

贯穿于websocekt数据发送的整个流程。

![upload successful](/images/pasted-86.png)

主要的属性有如下几条。

```
private enum State {
    OPEN,
    STREAM_WRITING,
    WRITER_WRITING,
    BINARY_PARTIAL_WRITING,
    BINARY_PARTIAL_READY,
    BINARY_FULL_WRITING,
    TEXT_PARTIAL_WRITING,
    TEXT_PARTIAL_READY,
    TEXT_FULL_WRITING
}
```

websocket内部的State有自己独立的类StateMachine来对State进行判断、管理。

state的流程图如下：

![upload successful](/images/pasted-87.png)

StateMachine管理器中checkState，如果状态state不对就会抛出异常。

```
private void checkState(State... required) {
    for (State state : required) {
        if (this.state == state) {
            return;
        }
    }
    throw new IllegalStateException(
            sm.getString("wsRemoteEndpoint.wrongState", this.state));
}
```

这也能看出websocket在多线程中是没法使用的，一个数据在发送过程中，state就会切换到非OPEN状态，下一条数据如果此时进来，就会跑出状态不对的异常。所以在调用websocket发送数据时是需要加上synchronized关键字。

#### opcode
opcode是用来表示websocket中发送的数据类型。

websocket中对不同类型数据有不同的发送方法，如图所示：


![upload successful](/images/pasted-88.png)

有sendBinary、sendObject、sendText、sendPing、sendPong。

对应每种数据类型，有不同的opcode来表示。

```
// OP Codes
public static final byte OPCODE_CONTINUATION = 0x00;
public static final byte OPCODE_TEXT = 0x01;
public static final byte OPCODE_BINARY = 0x02;
public static final byte OPCODE_CLOSE = 0x08;
public static final byte OPCODE_PING = 0x09;
public static final byte OPCODE_PONG = 0x0A;
```

顺着不同的数据类型来继续往下研究：

调用发送数据方法，进入到WsRemoteEndpointBasic类中，然后到WsRemoteEndpointImplBase方法实现类中。

- sendBinary ==> sendBytes
- sendText ==> sendString
- sendObject ==> sendBytes || sendString （使用instanceof解析Object）
- sendPing、sendPong

然后无论哪种数据类型，最终进入的方法都是sendMessageBlock方法，就是通过opcode进行不同数据类型区分。

![upload successful](/images/pasted-89.png)












