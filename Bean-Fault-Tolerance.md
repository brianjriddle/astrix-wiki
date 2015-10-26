> This is a draft

Astrix includes a fault-tolerance layer implemented on top of [Hystrix](https://github.com/Netflix/Hystrix) which can be used to protect invocations to an Astrix bean with fault isolation. The fault tolerance layer is "activated" by adding the `astrix-ft` module to the classpath. When the `astrix-ft` module is present, then all invocations to a service bean will be protected by the fault-tolerance layer without further configuration, and library beans might be protected by explicitly annotating the bean definition with @AstrixFaultToleranceProxy.

![Design FaultToleranceProxy](images/bean-fault-tolerance-design.png)

The mapping between an Astrix bean, and the name of the `HystrixCommand` used to protect an invocation on the given Astrix bean, is resolved using a `HystrixCommandNamingStrategy`. The `HystrixCommandNamingStrategy` allows users of the framework to implement whatever `HystrixCommand` naming scheme they prefer for their Astrix beans, see "Extending Astrix" for information about how to apply a custom strategy.

```java
interface HystrixCommandNamingStrategy {
    String getCommandKeyName(PublishedAstrixBean<?> beanDefinition);
	String getGroupKeyName(PublishedAstrixBean<?> beanDefinition);
}
```
Astrix applies two different types of protection using either `HystrixCommand` or `HystrixObservableCommand` depending on the return type of the invoked Astrix bean method. Most invocations (so called "synchronous" invocations) will be protected by a regular HystrixCommand using a thread-pool for isolation. Astrix may also handle a service invocations as "reactive", see below for definition. For reactive service invocations Astrix uses a HystrixObservableCommand and semaphores for protection.

### Reactive invocations
Astrix may treat a bean invocation as "reactive" depending on the return type of the invoked method. All methods returning a `java.util.concurrent.CompletableFuture`, `rx.Observable` and `com.gigaspaces.async.AsyncFuture` are treated as reactive by default. Its possible to extends the reactive type handling to include more types by implementing a `ReactiveTypeHandlerPlugin`.

### Configuration
Astrixs allows setting the initial values of many configuration parameters on the HystrixCommand. At runtime though, Astrix cant change the values of the Hystrix settings. Rather such settings are updated using Archaius, see the [Hystrix documentation on configuration](https://github.com/Netflix/Hystrix/wiki/Configuration) for details.

### AstrixBeanSettings related to fault tolerance
AstrixBeanSetting           | Default Value | Description 
:-------------------------- | -------------:|:--------------
INITIAL_TIMEOUT  | 1000 [ms]        | The initial timeout
INITIAL_MAX_CONCURRENT_REQUESTS  | 20 | Defines the default "maxConcurrentRequests" when semaphore isolation is used to protect invocations to the associated bean, i.e. the maximum number of concurrent requests before the fault-tolerance layer starts rejecting invocations
INITIAL_CORE_SIZE  | 10 | Defines the default "coreSize" when thread isolation is used to protect invocations to the associated bean, i.e. the number of threads in the bulkhead associated with a synchronous service invocation.
INITIAL_QUEUE_SIZE_REJECTION_THRESHOLD  | 10 |  Defines the default "queueSizeRejectionThreshold" for the queue when thread isolation is used to protect invocations to the associated bean, i.e. number of pending service invocations allowed in the queue to a thread-pool (bulkhead) before starting to reject invocations.

### Overriding DefaultBeanSettings in service definition
The default settings in the table above might be overriden in the bean definition for an individual Astrix bean by using the `@DefaultBeanSettings` annotation, see the javadoc for `DefaultBeanSettings`.


