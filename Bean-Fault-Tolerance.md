> This is a draft

Astrix includes a fault-tolerance layer implemented on top of Hystrix. Astrix protects all invocations to a service-bean by default, and libraries might be protected by explicitly marking the bean definition with @AstrixFaultToleranceProxy.

![Design FaultToleranceProxy](images/bean-fault-tolerance-design.png)

The mapping between a Astrix bean, and the name of the Hystrix command used to protect the invocation, is resolved using a HystrixCommandNamingStrategy. The HystrixCommandNamingStrategy allows users of the framework to implement whatever Hystrix naming scheme they prefer for their Astrix beans, see "Extending Astrix" for information about how to apply a custom strategy.

```java
interface HystrixCommandNamingStrategy {
     String getCommandKeyName(PublishedAstrixBean<?> beanDefinition);
	String getGroupKeyName(PublishedAstrixBean<?> beanDefinition);
}
```
Astrix applies two different types of proction using either HystrixCommand or HystrixObservableCommand. Synchronous invocations will be protected by a regular HystrixCommand using a thread-pool for isolation, see HystrixCommandSettings for default values. For reactive service invocations, see xxx, Astrix uses a HystrixObservableCommand and semaphores to protect the service invocation.

### Configuration
Astrixs allows setting the initial values of many configuration parameters on the HystrixCommand. At runtime though, Astrix cant change the values of the Hystrix settings. Rather such settings are updated using Archauis, see the Hystrix documentation on configuration for details (https://github.com/Netflix/Hystrix/wiki/Configuration).

### AstrixBeanSettings
AstrixBeanSetting  | Default Value | Description 
:-------------------------- | -------------:|:--------------
INITIAL_TIMEOUT  | 1000 [ms]        | The initial timeout
INITIAL_MAX_CONCURRENT_REQUESTS  | 20 | Defines the default "maxConcurrentRequests" when semaphore isolation is used to protect invocations to the associated bean, i.e. the maximum number of concurrent requests before the fault-tolerance layer starts rejecting invocations
INITIAL_CORE_SIZE  | 10 | Defines the default "coreSize" when thread isolation is used to protect invocations to the associated bean, i.e. the number of threads in the bulkhead associated with a synchronous service invocation.
INITIAL_QUEUE_SIZE_REJECTION_THRESHOLD  | 10 |  Defines the default "queueSizeRejectionThreshold" for the queue when thread isolation is used to protect invocations to the associated bean, i.e. number of pending service invocations allowed in the queue to a thread-pool (bulkhead) before starting to reject invocations.

### Overriding DefaultBeanSettings in service definition
The default settings in the table above might be overriden in the bean definition for individul Astrix beans by using the `@DefaultBeanSettings`, see the javadoc for `DefaultBeanSettings`.



