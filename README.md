# karate-grpc

[![Build Status](https://api.travis-ci.org/thinkerou/karate-grpc.svg)](https://travis-ci.org/thinkerou/karate-grpc)

gRPC Testing Made Simple by [Karate](https://github.com/intuit/karate).

## Testing hello-world

Prefer to use Maven:

```
$ mvn verify
$ # Run the server
$ mvn exec:java -Dexec.mainClass=example.helloworld.HelloWorldServer
$ # In another terminal run the client karate test
$ mvn test -Dtest=HelloWorldRunner
```

## How to write grpc client

You only need two steps:

- Read json file and parse protobuf object to the request of grpc server

- format the response of grpc server to json string and return it as grpc server

Like this:

```java
public class Client {

  // ...

  public static String greet(String input) {
    HelloRequest.Builder requestBuilder = HelloRequest.newBuilder();
    try {
      JsonFormat.parser().merge(input, requestBuilder);
    } catch (ProtocolBufferException e) {
      // ...
    }

    HelloReply response = null;
    try {
      response = blockingStub.sayHello(requestBuilder.build());
    } catch (StatusRuntimeException e) {
      // ...
    }

    String res = "";
    try {
      res = JsonFormat.printer().print(response);
    } catch (ProtocolBufferException e) {
      // ...
    }

    return res;
  }

  // ...
}

```

## How to write karate feature

We need to use [Java interop](https://github.com/intuit/karate#java-interop) of Karate in order to call us define grpc client.

And use `JSON.parse` javascript function parse the response of grpc server return value.

Like this:

```
Feature: grpc hello-world example

  Background:
    * def Client = Java.type('example.helloworld.HelloWorldClient')
    * def config = { host: 'localhost', port: 50051, extra: 'other config information' }
    * def client = new Client(config.host, config.port)

  Scenario: do it
    * def payload = read('helloworld.json')
    * def response = client.greet(payload)
    * eval client.shutdown()
    * def response = JSON.parse(response)
    * match response.message == 'Hello thinkerou'
```

That's all!
