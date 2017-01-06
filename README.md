Vertx-RPC
=====

[![Build Status](https://circleci.com/gh/MaxLeap/vertx-rpc.svg?style=shield&circle-token=67793c816897b2aa0dc59dda6a4b4079939b3eb7)](https://circleci.com/gh/organizations/MaxLeap)	

Wrap eventBus of vert.x 3 as transport layer for RPC invoking, and arguments of method could be POJO or primitive.
You just have to define `interface` of service, and then implements it with server end.
Expose interface as client for invoker, say separation of interface and implementation.


```xml
<dependency>
	<groupId>as.leap</groupId>
	<artifactId>vertx-rpc</artifactId>
	<version>3.3.8</version>
</dependency>
```

How to using
=======

1.Define service interface.

```java
public interface MyService {
	void hello(String what, Handler<AsyncResult<String>> handler)
}
```


2.Implements service with interface.

```java
public ExampleService implements MyService {
  	public void hello(String what, Handler<AsyncResult<String>> handler) {
		handler.handle(Future.succeededFuture("echo " + what));
    }
}
```		

3.Startup service on the server end.

```java
RPCServerOptions serverOption = new RPCServerOptions(vertx).setBusAddress("Address").addService(new ExampleService());
RPCServer rpcServer = new VertxRPCServer(serverOption);
```

4.Invoke service from client.

```java
RPCClientOptions<MyService> rpcClientOptions = new RPCClientOptions<MyService>(vertx).setBusAddress("Address")
.setServiceClass(MyService.class);

MyService myService = new VertxRPCClient(rpcClientOptions).bindService();

//invoking service
myService.hello("world", result -> {
	//TODO
});
```

We also support vertx's future as return type, you can specific return type with Future.

```java

Future<User> getUserById(int id);

```

full example could be found [here](https://github.com/stream1984/vertx-rpc-example).


Custom exception
=========
You can custom exception as normal, the exception will be passed from server side to client side.  
but the exception your defined that have to a `Constructor with a parameter which type have to be String`.  
you can find example [here](https://github.com/MaxLeap/vertx-rpc/blob/master/src/test/java/as/leap/rpc/example/spi/MyException.java#L18)


Hook method
=========
You can also hook your method either client or server or both.  
just implement interface `RPCHook` and then put it into Option.  

```java
RPCClientOptions<SampleHandlerSPI> rpcClientHandlerOptions = new RPCClientOptions<SampleHandlerSPI>(vertx).setRpcHook(new ClientServiceHook())
```

hook method running in a worker thread, so you can running with block method, we using it add `reqId` for identify method in invoking chain, and could also make metric for performance monitor and logs. 


The more detail
=========
We also support `Reactive` as return type, so you can define your interface as

`Observable<String> hello(String what)` instead of `void hello(String what, Handler<AsyncResult<String>> handler)`

or CompletableFuture

`CompletableFuture<String> hello(String what)` instead of `void hello(String what, Handler<AsyncResult<String>> handler)`

One more thing that about `timeout and retry`.
you can make annotation on your interface to define timeout for specify method and retry times.

```java
    @RequestProp(timeout = 1, timeUnit = TimeUnit.SECONDS, retry = 2)
    void hello(String what, Handler<AsyncResult<String>> handler);
```
or
```java
	@RequestProp(timeout = 1000) //default timeUnit is milliseconds
    void hello(String what, Handler<AsyncResult<String>> handler)
```
You can specify default timeout parameter in RPCClientOptions, if there are no RequestProp annotation on method, vertx-rpc will using
default timeout, annotation @RequestProp have highest priority.

`retry = 2` meaning that will repeat request at most 2 times after found Timeout Exception,
this is not include original request, so this would be throw Timeout Exception after try 3 times.

