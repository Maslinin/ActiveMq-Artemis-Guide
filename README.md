# ActiveMq Artemis

You can read this documentation in the following languages:
- [Russian](https://github.com/Maslinin/ActiveMq-Artemis-Guide/blob/master/locales/ActiveMq-Artemis-Guide-RU.md)

## Basic information
**ActiveMq Artemis** - is highly efficient message broker designed to process a large number of messages in real time. Artemis is based on JVM and supported on Windows, Linux и macOS.

ActiveMq Artemis features:
- Supports various OS - Windows, Linux и macOS;
- Supports various protocols - OpenWire, STOMP, AMQP, MQTT и HornetQ;
- High efficiency - broker can proceed up to 10 million messages per second;
- Scalability - broker can be launched in a cluster and horizontaly scaled to proceed a larger number of messages.

## ActiveMq Artemis operation principle

### Address
One of basic conceptions in ActiveMQ Artemis is address. Addresses are used to identify destination points, where messages can be sent. Each address is a unique string consisting of two parts: *Address name* and *Routing type (RoutingType)*. Address name can contain any sequence of characters, and RoutingType defines, how messages sent to this address will be processed.

### RoutingType
There are two types of routing(RoutingType) in ActiveMQ Artemis: **Anycast** and **Multicast**. Anycast means that the message will be put into one single queue, even if the address specifies multiple queues. Multicast means that the message will be put into every queue specified in the address. Thus the address can specify one queue(Anycast) or multiple queues(Multicast).

### Queues
Queues - are the names for identification of destination points where messages will be stored and await for their processsing. Queues are FIFO data structures which allow you to process messages in the order in which they are arrived.

There are several types of queues in ActiveMQ Artemis including *Persistent Queue* and *Non-Persistent Queue*. Persistent Queue stores messages on disk until they are delivered or until they expire; Non-Persistent Queue, in turn, stores messages in memory, while deleting them only after they are confirmed for delivery or when they expire.

## Working with Mq Artemis using C#
Unfortunately, Apache doesn't have an official ActiveMq Artemis client for .NET; 
But there is an unofficial **ArtemisNetClient** package by Havret for ActiveMq Artemis on .NET, giving simple and intuitive interface for writing, sending and receiving messages.

> This paragraph is relevent for **ArtemisNetClient** of version **2.12.0**

To start working with ArtemisNetClient, you need to install packages with *NuGet package manager* or *dotnet CLI*. In package manager you can use following command:
```
Install-Package ArtemisNetClient
```
and for dotnet CLI:
```
dotnet add package ArtemisNetClient
```

Then to use ArtemisNetClient in your project, you need to add following namespace into your project:
```
using ActiveMQ.Artemis.Client;
```

Now you can start creating factory to create connections to Artemis elements, such as *ProducerClient* (to send messages) and *ConsumerClient* (to receive messages).

To create factories you can use following code:
```
var endpoint = Endpoint.Create(
    "192.168.0.1", 
    61616, 
    "User", 
    "Password",
    Scheme.Amqp
);

var connection = await new ConnectionFactory().CreateAsync(endpoint);
```

### Sending messages
To send a message you need to create a copy of **IProducer** from **connection** object with **CreateProducerAsync()** method, as shown in the code below:
```
var producer = connection.CreateProducerAsync(address, RoutingType.Multicast);
```

Now you can send messages using the following codeТеперь:
```
producer.SendAsync(new Message("foo"))
```

Take a note that when creating a producer we set the equal parameter **RoutingType**, **Multicast** to equal, meaning that the message will be sent to every available queue at once.

### Receiving messages
To receive a message you need to create a copy of **IConsumer** from **connection** object with **CreateConsumerAsync()** method, as show in the code below:
```
var consumer = connection.CreateConsumerAsync(address, queue);
```

The consumer object we created will be subscribed to a specific address in the queue, from where we can receive the message using the **ReceiveAsync()** method:
```
var message = await consumer.ReceiveAsync();
await consumer.AcceptAsync(message);
```

Take a note, that in this example we have confirmed the delivery of the message with the **AcceptAsync()** method after receiving it, after which the received message will be immediately removed from the queue.
