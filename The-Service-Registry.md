> This is a very early draft

The service registry is the most common way to do service discovery. It is a service that allows dynamic service registration. Astrix implements "self service registration" using the service registry.

The service registry is a service shipped with the framework. It is provided using the same ServiceComponent framework as any other service implemented using Astrix. The biggest difference is that all consumers of the service registry uses config-discovery to find the service registry (to avoid a bootstrap problem).

You never interact with the service registry directly. Its used by the framework behind the scenes by service consumers to do service discovery, and by service providers to make its services available.

### How a consumer discovers a provider using the service registry
![ServiceBeanInstance](images/Astrix-Design-ServiceRegistry.png) 

Its important to note that Astrix does use the service registry to discover "service"-endpoints in the registry. Rather it discovers a binding-mechanism and associated properties.


The Astrix view of a service:

```java
interface OrderValidation {
    ValidationResult validateOrder(Order order);
}
```


### Running a Service Registry





