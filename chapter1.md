# Node Core Concepts

Small Ecosystem

* small modules which contain their own dependencies
* expose only minimal set of functionality
* Reactor pattern

I/O is slow,

The process for doing input / output can be a very time intesive process. This is mainly due to the fact that the RAM access is a slow process.

Why is it slow?

It is slow because accessing RAM is 10e^-9 seconds, or on the order of nanoseconds \(the RAM is optimized for larger metal gates, faster connections\) while accessing files on a disk is an order of milliseconds \(10^e^-3\). For network operations it is also the same story, RAM has transfer rate in GB while network transfer rates are in the realm of MB. The moment a request is sent and completes there is a time delay. Moreover, the click of a mouse or a user input is inherently slower than machine operations so this adds to the length of an operation.

## Blocking I/O

In traditional blocking programming, function call to an io request will block execution of the thread until the operation completes. This can vary tremendously, you are basically in an idle or waiting state and more operations could be accomplished. In a grocery store, you are waiting in line for the next cart to be emptied, processed and placed into grocery bags back into the cart.

```
// traditional blocking
data = socket.read();
print(data);
```

thread is blocked until data is available. Traditional way of dealing with this blocking is to distribute the other tasks to another thread. For example, in a grocery store a store manager will direct a store clerk to open up another line to process more customers and create a new queue. However, the user still waits in line for a

## Non-Blocking I/O

Whereas blocking IO waited for system resources to become available and then allocate a new thread, non-blocking IO the system call returns immediately without waiting to be finished. This would be analogous to ordering your food and receiving a ticket with your name and number on it to receive your food when your oder has been completed.

In unix operating systems, the fcntl\(\) function is used to change the file descriptor to O\_NONBLOCK signifiying non-blocking. Once in this mode, any read operation will fail if there isn't any data to read yet.

### Busy-Waiting

polling or busy-waiting. In busy-waiting, the thread will continue to try and access the resources at given intervals checking to see if the data is ready for consumption. This is analogous to going to customer going to the chef every five minutes to check if the food is ready to be eaten. As you can imagine, this would result in a huge amount of wasted time and resources for the customer.

### Event Notification Interface

This component collects and queues I/O events that come from a set of watched resources and block until those events are ready to be processed. 

```
var watchList = [];
watchList.push({data: pipeA, operation: FOR_READ});
watchlist.push({data: pipeB, operation: FOR_READ});
var events = demultiplexer.watch(watchList);
if (!events.length) return null; // finished

for (event in events) {
    var data = event.data.read();
    if (data === CLOSED) demultiplexer.unwatch(event);
    else consume(data);
}
```

Here the resource pipeA and pipeB are associated with operations like FOR\_READ which will then be added to the demultiplexer. This call is synchronous and blocks until any of the resources is ready to be read. When this happens, a new set of events it returned and ready to be processed. This flow is called the event loop --&gt; event enters queue, listeners are notified, event queue is updated. Repeat.

## Reactor Pattern

So you have a handler which in node is a callback function which is associated with each I/O operation and will be invoked as soon as an event is produced and processed by the event loop. 

```js
var demultiplexer = require('demultiplexer'); // is such a thing exists like this
var watchList = [];
watchList.push({data: pipeA, operation: FOR_READ, handler: handleRead});
watchlist.push({data: pipeB, operation: FOR_READ, handler: handleRead});
var events = demultiplexer.watch(watchList);

if (!events.length) return null; // finished

for (event in events) {
    var data = event.data.handler(); //HANDLER is used
    if (data === CLOSED) demultiplexer.unwatch(event);
    else consume(data);
}
```





