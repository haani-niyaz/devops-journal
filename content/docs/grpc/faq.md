---
title: FAQ
---

# GRPC FAQ

### Do I need to run each method in a goroutine?

No, See [this](https://github.com/grpc/grpc-go/blob/master/Documentation/concurrency.md#servers):

> Each RPC handler attached to a registered server will be invoked in its own goroutine. For example, SayHello will be invoked in its own goroutine. The same is true for service handlers for streaming RPCs, as seen in the route guide example here. Similar to clients, multiple services can be registered to the same server.

### How do I handle errors?

See [A handy guide to GRPC errors](http://avi.im/grpc-errors/#go). More specifically, the github repo [link](https://github.com/avinassh/grpc-errors/tree/master/go).

### What are Deadlines?

> Deadlines allow gRPC clients to specify how long they are willing to wait for an RPC to complete before the RPC is terminated with the error DEADLINE_EXCEEDED . By default this deadline is a very large number, dependent on the language implementation.

The docs recommend setting a deadline. See [deadlines](http://grpc.io/blog/deadlines) for detailed information.