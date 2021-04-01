title: gRPC入门
categories:
  - 网络
  - ''
tags:
  - gRPC
date: 2020-08-02 18:03:00
---
新项目采用gRPC的通信模式，gRPC入门学习
- 4种rpc定义方法与3种stub介绍

接触gRPC通信，对gRPC进行深入的了解与学习。包括Proto文件的格式，与编译方法的使用。

<!-- more -->



### gRPC What

有关[gRPC官方介绍](https://www.grpc.io/docs/what-is-grpc/introduction/)：

![upload successful](/images/pasted-28.png)

gRPC是一个能在不同语言不同平台中进行高效通信的服务。gRPC默认使用Protocol Buffers数据格式：

``` bash
Protocol Buffers：Google成熟的用于序列化结构化数据的开源机制（类似于JSON并且能与JSON一起使用）
```

Protocol Buffers以.proto作为拓展名，是一系列以name-value键值对的形式存储的数据格式。

```bash
message  Person{

    string  name=1;

    int32  id=2;

    bool  has_ponycopter=3;    

}
```

Protocol Buffers从开源到现在已经经过很长时间，目前已经到了proto3版本，有着更加简化的语法，更加有用的特性，能够支持更多的语言。你可以从proto3 language guide和 reference documentation看到更多有用的东西。另外.proto的文件格式能够从formal specification获取到更详细的讲解。

```bash
As you can see, each field in the message definition has a unique number. These field numbers are used to identify your fields in the message binary format, and should not be changed once your message type is in use. Note that field numbers in the range 1 through 15 take one byte to encode, including the field number and the field's type (you can find out more about this in Protocol Buffer Encoding). Field numbers in the range 16 through 2047 take two bytes. So you should reserve the numbers 1 through 15 for very frequently occurring message elements. Remember to leave some room for frequently occurring elements that might be added in the future.

The smallest field number you can specify is 1, and the largest is 229 - 1, or 536,870,911. You also cannot use the numbers 19000 through 19999 (FieldDescriptor::kFirstReservedNumber through FieldDescriptor::kLastReservedNumber), as they are reserved for the Protocol Buffers implementation - the protocol buffer compiler will complain if you use one of these reserved numbers in your .proto. Similarly, you cannot use any previously reserved field numbers.
```

每个参数带有一个唯一的标识符，这些标识符被用来在message的二进制中被识别出来。不是代表每个数据的数值。

### Why use gRPC? 

有了gRPC，我们可以在.proto文件中定义我们的服务，并用gRPC支持的任何语言生成客户端和服务器，它可以在从大型数据中心内的服务器到您自己的平板电脑的各种环境中运行，不同语言和环境之间的所有复杂通信都由gRPC为您处理。我们还获得了使用协议缓冲区的所有优点，包括高效的序列化、简单的IDL和容易的接口更新。

### (JAVA).proto格式

声明定义proto使用3版本，如果不生命默认2版本号。

``` bash
syntax = "proto3";
```

在proto文件中定义java_package，指定了我们要用于生成的Java类的包。如果.proto文件中没有显式的java_package选项，那么默认情况下将使用proto包（使用“package”关键字指定）。

```bash
option  java_package  =  "io.grpc.examples.routeguide";
```

import导入其他proto文件报错时，记得将proto文件夹，右键设置选择Mark Dirctionary as，设置成资源文件夹哦。

使用service定义服务。然后在service中使用rpc方法定义，gRPC允许使用4种不同的定义方式，定义方法。
```bash

service  RouteGuide {

    rpc   GetFeature(Point)   returns (Feature) {}

    rpc   ListFeatures(Rectangle)   returns (stream Feature) {}

    rpc   RecordRoute(stream Point)   returns (RouteSummary) {}

    rpc   RouteChat(stream RouteNote)   returns (stream RouteNote) {}

	}
```
### 三种Stub使用和方法区别
区别在客户端调用Grpc中Stub发送请求方法时：

#### newStub

```bash
当Grpc.Stub.GetFeature：

//双一元请求方法，在newStub中是没有返回值，在参数中虽然是使用StreamObserver，但是进一步调用的call方法为asyncUnaryCall，在这个方法中声明了boolean streamingResponse为false，进而返回值不是数据流。

返回值：void

参数：Request request,   StreamObserver<Response>  responseObserver
```
```bash
当Grpc.Stub.ListFeatures：

//在参数中使用StreamObserver，进一步调用的call方法为asyncServerStreamingCall，在这个方法中声明了boolean streamingResponse为true，返回值是数据流。

返回值：void

参数：Request request, StreamObserver<Response> responseObserver
```
```bash
当Grpc.Stub.RecordRoute：

返回值：StreamObserver<Request>

参数：streamResponse responseObserver
```
```
当Grpc.Stub.RouteChat：双向流式请求，

返回值：StreamObserver<Request>

参数：StreamObserver<Response> responseObserver
```
#### newBlockingStub
```
当Grpc.Stub.GetFeature：

返回值：Response

参数：Request request
```
```
当Grpc.Stub.ListFeatures：

返回值：java.util.Iterator<Response>

参数：Request request
```
```
当Grpc.Stub.RecordRoute：没有rpc对应的方法。。

返回值：

参数：	
```
```
当Grpc.Stub.RouteChat：双向流式请求，没有对应方法

返回值：

参数：
```
#### newFutureStub
```
当Grpc.Stub.GetFeature：

返回值：ListenableFuture<streamResponse>

参数：streamRequest request
```
```
当Grpc.Stub.ListFeatures：没有rcp对应的方法。。

返回值：

参数：
```
```
当Grpc.Stub.RecordRoute：没有rpc对应的方法。。

返回值：

参数：
```
```
当Grpc.Stub.RouteChat：双向流式请求，没有对应方法

返回值：

参数：
```
可以使用message定义所有的request and response types，在service中使用到的数据格式。
```
message    Point  {

    int32    latitude = 1;

    int32    longitude = 2;

}    
```