> This is a very early draft

Astrix provides a number of extension points, where some of the more important are:

* `ServiceComponent`
* `ServiceDiscoveryFactoryPlugin`
* `BeanPublisherPlugin`

This part describes how you plug-in your custom extensions into Astrix at runtime.

### Plugins vs Strategies
Astrix allows for two different types of extensions, plugins and strategy. Put simply, there might be an arbritrary number of a given plugin type (for instance `ServiceComponent` plugins), as opposed to a strategy where Astrix assumes exactly one instance at runtime. 

### Implementing a plugin
A plugin is created by implementing the `AstrixContextPlugin` interface. It contains one non-default method: 
```java
public interface AstrixContextPluign {
	void prepare(ModuleContext moduleContext);
}
```

```java
@MetaInfServices(AstrixContextPlugin.class)
public class CustomRemotingModule implements AstrixContextPlugin {

	@Override
	public void prepare(ModuleContext moduleContext) {
		// 1.
		moduleContext.bind(ServiceComponent.class, MyCustomRemotingComponent.class);
		
		// 2.
		moduleContext.importType(AstrixServiceActivator.class);
		moduleContext.importType(ObjectSerializerFactory.class);
		moduleContext.importType(RemotingProxyFactory.class);
		moduleContext.importType(AstrixConfig.class);
		
		// 3.
		moduleContext.export(ServiceComponent.class);
	}
}
```

1. Setup internal bindings used for dependency injection within the component, see [astrix-modules](https://github.com/AvanzaBank/astrix/tree/master/astrix-modules)
2. Import components from the AstrixContext
3. Export the `ServcieComponent` type from this module

### Loading the plugin
Astrix uses the java [ServiceLoader](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html) mechanism to load plugins at runtime.

#### Content of META-INF/services/com.avanza.astrix.context.AstrixContextPlugin
```
com.mycorp.astrix.ext.CustomRemotingModule
```


### Implementing a Stratgey
```java
@MetaInfServices(AstrixContextPlugin.class)
public class MyCustomHystrixCommandNamingStrategy implements AstrixContextPlugin {
	
	@Override
	public void registerStrategies(AstrixStrategiesConfig strategiesConfig) {
		strategiesConfig.registerStrategy(HystrixCommandNamingStrategy.class, MyCustomHystrixCommandNamingStrategy.class);
	}
	
	@Override
	public void prepare(ModuleContext moduleContext) {
	}
}
```

### Loading a Strategy
Strategies are loaded in the same way as a plugin.


