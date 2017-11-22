Dsm.Common.Messaging
===

This package incluses _MassTransit_ and _RabbitMQ_ core implementation for messaging architecture.

Features:
--------
- Provide create message Producer
- Provide create message Consumer
- Provide create message Request/Response client
- Include auto retry policies
- Provide circuit breaker
- Provide rate limiter

Usage:
-----

Initializing RabbitMQ BUS instance for _Producer_:

```cs
ISendEndpoint bus = await BusInitializer.Instance.UseRabbitMq(string rabbitMqUri, string rabbitMqUserName, string rabbitMqPassword)
													.InitializeProducer(string queueName);
```


after RabbitMQ BUS instance initializing then you can use _Send_ method with your queues channel _TCommand_ type.

```cs
bus.Send<TCommand>(new
			{
				SomeProperty = SomeValue
			}
		);
```


_RabbitMQ BUS instance_ using for Consumer:

```cs
static async void Main(string[] args)
{
	BusHandle bus = await BusInitializer.Instance.UseRabbitMq(string rabbitMqUri, string rabbitMqUserName, string rabbitMqPassword)
							.InitializeConsumer<TCommandConsumer>(string queueName)
							.Start();

	bus.Stop();

	Console.ReadLine();
}
```


_TCommandConsumer_ could like below:

```cs
public class TCommandConsumer : IConsumer<TCommand>
{
    public async Task Consume(ConsumeContext<TCommand> context)
    {
        var command = context.Message;

		//do something...
        await Console.Out.WriteAsync($"{command.SomeProperty}");
    }
}
```

Initializing RabbitMQ BUS instance for Request/Response operation:

```cs
IRequestClient<TRequest, TResponse> client = BusInitializer.Instance.UseRabbitMq(string rabbitMqUri, string rabbitMqUserName, string rabbitMqPassword)
                                                                    .InitializeRequestClient<TRequest, TResponse>(string queueName);

TResponse result = await client.Request(new TRequest
{
    Command = "Say hello!"
});
```

and _TCommandConsumer_ for Request/Response operation could like below:

```cs
public class TCommandConsumer : IConsumer<TRequest>
{
    public async Task Consume(ConsumeContext<TRequest> context)
    {
        var command = context.Message;

		//do something...
        await Console.Out.WriteAsync($"{command.SomeProperty}");

		//and
		context.Respond(new TRequest
            {
                Command = "Hello!"
            });
    }
}
```


**PS**: _Publisher_ and _Consumer_ services must be used same _TCommand_ interface. This case important for MassTransit integration. Also one other thing is _rabbitMqUri_ parameter must start with "rabbitmq://" prefix.


There are several options you can set via fluent interface:

- `.UseRetryPolicy(int retryLimit, int initialIntervalFromMinute, int intervalIncrementFromMinute, params Exception[] retryOnSpecificExceptionType)`
- `.UseCircuitBreaker(int tripThreshold, int activeThreshold, int resetInterval)`
- `.UseRateLimiter(int rateLimit, int interval)`