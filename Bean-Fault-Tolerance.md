> This is a draft

Astrix includes a fault-tolerance layer implemented on top of [Hystrix](https://github.com/Netflix/Hystrix) which can be used to protect invocations to an Astrix bean with fault isolation. The fault tolerance layer is activated by adding the `astrix-ft` module to the classpath. When the `astrix-ft` module is present, then all invocations to a service bean will be protected by the fault-tolerance layer without further configuration, and library beans might be protected by explicitly annotating the bean definition with @AstrixFaultToleranceProxy.

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
Astrixs allows setting the initial values of many configuration parameters on the HystrixCommand. At runtime though, Astrix can't change the values of the Hystrix settings. Rather such settings are updated using Archaius, see the [Hystrix documentation on configuration](https://github.com/Netflix/Hystrix/wiki/Configuration) for details. Astrix always uses a `maxQueueSize` of 1 000 000 when protecting a service invocation with a `HystrixCommand` to ensure that the `queueSizeRejectionThreshold` configuration have the desired effect, see below for the initial value of the `queueSizeRejectionThreshold` property.

### AstrixBeanSettings related to Hystrix configuration
AstrixBeanSetting                       | Default Value    | Description 
:-------------------------------------- | ----------------:|:--------------
INITIAL_TIMEOUT (astrix.bean.[beanKey].faultTolerance.timeout) | 1000 [ms]        | The initial timeout used when protecting a bean invocation with Hystrix
INITIAL_MAX_CONCURRENT_REQUESTS (astrix.bean.[beanKey].faultTolerance.enabled) | 20               | Defines the initial "maxConcurrentRequests" when semaphore isolation is used to protect invocations to the associated bean, i.e. the maximum number of concurrent requests before the fault-tolerance layer starts rejecting invocations
INITIAL_CORE_SIZE (astrix.bean.[beanKey].faultTolerance.initialMaxConcurrentRequests) | 10               | Defines the initial "coreSize" when thread isolation is used to protect invocations to the associated bean, i.e. the number of threads in the bulkhead associated with a synchronous service invocation.
INITIAL_QUEUE_SIZE_REJECTION_THRESHOLD  (astrix.bean.[beanKey].faultTolerance.initialCoreSize) | 10 |  Defines the initial "queueSizeRejectionThreshold" for the queue when thread isolation is used to protect invocations to the associated bean, i.e. number of pending service invocations allowed in the queue to a thread-pool (bulkhead) before starting to reject invocations.

### Enabling/disabling fault tolerance at runtime
Fault tolerance may be enabled/disabled at runtime for all beans, or individual beans, in a `AstrixContext`. In order for a bean invocation to be protected by a fault-tolerance proxy, fault tolerance bust be enabled both globally and for the individual bean using these settings:


AstrixBeanSetting                       | Default Value    | Description 
:-------------------------------------- | ----------------:|:--------------
FAULT_TOLERANCE_ENABLED  (astrix.bean.[beanKey].faultTolerance.enabled) | true             | Determines whether invocations on the associated bean should be protected by fault tolerance. Applies to all service beans, and also to library beans whose definition are annotated with `@AstrixFaultToleranceProxy`.


AstrixSetting                           | Default Value    | Description 
:-------------------------------------- | ----------------:|:--------------
ENABLE_FAULT_TOLERANCE (AstrixContext.enableFaultTolerance)| true             | Globally disables/enables the fault tolerance proxy for all beans in the associated `AstrixContext`

### Overriding DefaultBeanSettings in service definition
The default settings in the table above might be overriden in the bean definition for an individual Astrix bean by using the `@DefaultBeanSettings` annotation, see the javadoc for `DefaultBeanSettings`.


