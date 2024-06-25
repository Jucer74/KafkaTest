# KafkaTest
Using the current project complete the Producer and Consumer.

This Exercise uses the [Confluent Kafka Library](https://github.com/confluentinc/confluent-kafka-dotnet/) and [Confluent Kafka Cluster](https://www.confluent.io/) to implement the Kafka broker.


# Producer
Using the Api-Key File and the global values for users and items, generate 10 random messages to the Kafka broker.

These messages must to have a random numeric Key between 1 to 100 and the message value must be a concatenation  for the random user and item.

Example:

```code
...Producer Started...
Produced event to topic purchases: key = 99  value = awhite-eraser
Produced event to topic purchases: key = 72  value = rdowney-pencil
Produced event to topic purchases: key = 53  value = bgriffin-notebook
Produced event to topic purchases: key = 71  value = awhite-pencil
Produced event to topic purchases: key = 99  value = jsmith-ruler
Produced event to topic purchases: key = 11  value = cgarcia-notebook
Produced event to topic purchases: key = 68  value = mjordan-notebook
Produced event to topic purchases: key = 76  value = awhite-notebook
Produced event to topic purchases: key = 17  value = awhite-notebook
Produced event to topic purchases: key = 74  value = cgarcia-pencil
10 messages were produced to topic purchases
...Producer Stoped...
```

## Producer Configuration
Use the bellow example to configure the producer

```csharp
var config = new ProducerConfig
{
    // User-specific properties that you must set
    BootstrapServers = "<BOOTSTRAP SERVERS>",
    SaslUsername     = "<CLUSTER API KEY>",
    SaslPassword     = "<CLUSTER API SECRET>"

    // Fixed properties
    SecurityProtocol = SecurityProtocol.SaslSsl
    SaslMechanism    = SaslMechanism.Plain
    Acks             = Acks.All
};
```
## Producer Sample
Use the belllow example to generte mesages
```csharp
using System;
using Confluent.Kafka;

var conf = new ProducerConfig { BootstrapServers = "localhost:9092" };

Action<DeliveryReport<Null, string>> handler = r =>
    Console.WriteLine(!r.Error.IsError
        ? $"Delivered message to {r.TopicPartitionOffset}"
        : $"Delivery Error: {r.Error.Reason}");

using (var p = new ProducerBuilder<Null, string>(conf).Build())
{
    for (int i = 0; i < 100; ++i)
    {
        p.Produce("my-topic", new Message<Null, string> { Value = i.ToString() }, handler);
    }

    // wait for up to 10 seconds for any inflight messages to be delivered.
    p.Flush(TimeSpan.FromSeconds(10));
}
```

# Consumer
Using the Api-Key File consume the messages previously generate by the consumer.

## Consumer Configuration
use the bello example to configure the consumer

```csharp
var config = new ConsumerConfig
{
    // User-specific properties that you must set
    BootstrapServers = "<BOOTSTRAP SERVERS>",
    SaslUsername     = "<CLUSTER API KEY>",
    SaslPassword     = "<CLUSTER API SECRET>"

    // Fixed properties
    SecurityProtocol = SecurityProtocol.SaslSsl
    SaslMechanism    = SaslMechanism.Plain
    GroupId          = "kafka-test",
    AutoOffsetReset  = AutoOffsetReset.Earliest
};
```

## Consumer Sample
Use the bellow sample to consume the messages

```csharp
using System;
using System.Threading;
using Confluent.Kafka;

var conf = new ConsumerConfig
{
    GroupId = "test-consumer-group",
    BootstrapServers = "localhost:9092",
    // Note: The AutoOffsetReset property determines the start offset in the event
    // there are not yet any committed offsets for the consumer group for the
    // topic/partitions of interest. By default, offsets are committed
    // automatically, so in this example, consumption will only start from the
    // earliest message in the topic 'my-topic' the first time you run the program.
    AutoOffsetReset = AutoOffsetReset.Earliest
};

using (var c = new ConsumerBuilder<Ignore, string>(conf).Build())
{
    c.Subscribe("my-topic");

    CancellationTokenSource cts = new CancellationTokenSource();
    Console.CancelKeyPress += (_, e) => {
        // Prevent the process from terminating.
        e.Cancel = true;
        cts.Cancel();
    };

    try
    {
        while (true)
        {
            try
            {
                var cr = c.Consume(cts.Token);
                Console.WriteLine($"Consumed message '{cr.Value}' at: '{cr.TopicPartitionOffset}'.");
            }
            catch (ConsumeException e)
            {
                Console.WriteLine($"Error occured: {e.Error.Reason}");
            }
        }
    }
    catch (OperationCanceledException)
    {
        // Ensure the consumer leaves the group cleanly and final offsets are committed.
        c.Close();
    }
}
```