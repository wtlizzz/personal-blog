title: SpringBoot模拟gRPC通信
categories:
  - 集成
tags:
  - gRPC
  - 模拟通信
  - SpringBoot
date: 2020-08-02 20:40:45
---
使用SpringBoot模拟gRPC通信，包括.proto文件编译、客户端实现、服务端实现、模拟双向流通信学习与实践。
<!-- more -->

### .proto文件编译
在main目录下新建proto目录，创建hello.proto文件。文件内容如下：

```
syntax = "proto3";//指定proto版本

package com.aaaa.monitor;//指定proto文件包名

option java_package ="com.aaaa.monitor";//指定 java 包名

option java_outer_classname ="ServiceProto";//指定proto文件生成java文件后的类名

option java_multiple_files =true;//开启多文件

message HelloRequest {//请求参数

    string name = 1;  //1仅作为该message与其他数据区别的标示

}

message HelloReply {//响应参数

    string message = 1; 

}

//定义rpc服务接口

service Greeter{//服务端接口方法

  rpc SayHello(com.aaaa.monitor.HelloRequest)returns (com.aaaa.monitor.HelloReply);

}

```


使用maven compile   编译之后会在target/generated-sources文件夹中增加protobuf文件夹。编译完之后目录如下：

![aUIkzd.png](https://s1.ax1x.com/2020/08/03/aUIkzd.png)


生成了在proto文件中service参数对应的Grpc类，在类中自动创建了newstub、newBlockingStub、newFutureStub三个方法。分别对应的是：
```
newstub：Creates a new async stub that supports all call types for the service

```
翻译一下就是创建一个异步的stub，支持所有类型的service。

```
newBlockingStub：Creates a new blocking-style stub that supports unary and streaming output calls on the service

```
翻译一下就是创建一个新的阻塞式的stub，能够支持一元和流式输出调用。

```
newFutureStub：Creates a new ListenableFuture-style stub that supports unary calls on the service

```
翻译一下就是创建一个未来监听stub，支持一元服务。

文件中还包含了GreeterImplBase方法，通过继承该抽象类来实现一个service，添加到ServerBuilder中，GreeterImpl实现类中添加服务端处理.proto文件中service参数定义的rpc。


### 客户端实现

使用GreeterGrpc.newBlockingStub(channel)，生成的Grpc文件中的Stub来发送请求。

首先定义

```
ManagedChannel channel = ManagedChannelBuilder.forTarget(target).usePlaintext().build();

```
然后定义一个blockingStub

```

blockingStub = GreeterGrpc.newBlockingStub(channel);

```

在使用blockingStub时，这时候就用到了proto生成的另一个文件夹，在protobuf文件夹中生成的java文件夹，里面包含了在项目中.proto文件中定义的message数据，每一个message数据生成一个接口，一个实现类。

在blockingStub中调用service定义的rpc方法，传递的参数为HelloRequest，在.proto文件中定义的message数据。然后接收到的参数为HelloReply，同样是在.proto文件中定义的message数据。

```
HelloRequest request = HelloRequest.newBuilder().setName(name).build();

HelloReply response;

response =blockingStub.sayHello(request);
```


### 服务端实现

在服务端实现上，重要的部分就是自定义实现.proto文件中定义的service参数中rpc方法。

```
public class GreeterImplextends GreeterGrpc.GreeterImplBase {

@Override

    public void sayHello(HelloRequest req, StreamObserver responseObserver) {

HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName()).build();

        responseObserver.onNext(reply);

        responseObserver.onCompleted();

    }

}
```
定义完方法后，定义端口号，打开服务就能够接收gRPC请求。


```
server = ServerBuilder.forPort(port).addService((BindableService)new  GreeterImpl()).build().start();

```

### 模拟双向流通信


服务端与普通通信差别不大，直接附代码

```
public void start()throws IOException {

        int port =8081;

        server = ServerBuilder.forPort(port).addService(new StreamServiceImpl()).addService(new                         GreeterImpl()).build().start();

        logger.info("Server started, listening on " + port);

        Runtime.getRuntime().addShutdownHook(new Thread() {

        @Override

            public void run() {

                    System.out.println("*** shutting down gRPC server since JVM is shutting down");

                    try {

                    StreamServer.this.stop();

                    }catch (InterruptedException e) {

                        e.printStackTrace(System.err);

                    }    

                        System.out.println("*** server shut down");

               }

        });

    }
```
服务端ServiceImpl实现部分，流传输通过接收response参数，返回request参数来实现的。

```
public class StreamServiceImplextends StreamServiceGrpc.StreamServiceImplBase {

        private Loggerlog = LoggerFactory.getLogger(StreamServiceImpl.class);    

        int count =0;

        @Override

        public StreamObservergetNetInfo(StreamObserver responseObserver) {

                return new StreamObserver() {

                        @Override

                        public void onNext(streamRequest streamRequest) {        

                                log.info("Body：" + streamRequest.getNetBodyInfo() +"\t" +"Head:" +                                     streamRequest.getNetHeadInfo() +"count:" +count++);

                                streamResponse response = streamResponse.newBuilder()

                                                                                .setAcceptInfo("Accept" +count).build();

                            responseObserver.onNext(response);

                        }

                        @Override

                        public void onError(Throwable throwable) {

                            log.error("StreamServiceImpl.onError");

                        }

                        @Override

                        public void onCompleted() {

                                responseObserver.onCompleted();

                        }

                 };

        }

}
```
客户端实现
客户端部分实现了StreamObserver<streamResponse>，用来接收服务端返回的数据。

发送是使用StreamObserver<streamRequest> request.onNext来发送。

通过Stub.getNetInfo(responseStreamObserver)，来调用服务端的ServiceImpl方法，持续接收到responseStreamObserver中onNext方法的数据。

在此用例中增加了，延时发送请求操作，Thread.sleep(3000);

最后增加request.onCompleted();  &&   responseStreamObserver.onCompleted();完成操作。

```
public void sendNetInfo(String info) {

        StreamObserver responseStreamObserver =new StreamObserver() {

                private int cut =0;

                @Override

                public void onNext(streamResponse streamResponse) {

                        logger.info("接收到第" +cut++ +"次streamResponse");

                        logger.info(streamResponse.toString());    

                }

                @Override

                public void onError(Throwable throwable) {

                        logger.warning("onError：" + throwable.getMessage());

                }

                @Override

                public void onCompleted() {

                        logger.warning("onCompleted");

                }

        };

        //发送流式请求

        try {

                StreamObserver request =ssStub.getNetInfo(responseStreamObserver);

                for (int i =0; i <10; i++) {

                        streamRequest streamRequest = com.aircas.monitor.streamRequest.newBuilder().

                        setNetBodyInfo("I am body :" + i).setNetHeadInfo("I am Head :" + i).build();

                        request.onNext(streamRequest);

                        Thread.sleep(3000);

            }

            request.onCompleted();

            responseStreamObserver.onCompleted();

        }catch (StatusRuntimeException | InterruptedException e) {

                logger.log(Level.WARNING, "RPC failed: {0}", e.getMessage());

                return;

        }

}

public static void main(String[] args)throws Exception {

        ManagedChannel channel = ManagedChannelBuilder.forTarget(target).usePlaintext().build();

        try {    

                StreamClient client =new StreamClient(channel);

                client.sendNetInfo(user);

        }finally {

                channel.shutdownNow().awaitTermination(5, TimeUnit.SECONDS);

        }

}
```
最后附上运行图
![aUIEQA.png](https://s1.ax1x.com/2020/08/03/aUIEQA.png)