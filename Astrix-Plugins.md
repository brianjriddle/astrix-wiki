Astrix provides a number of extension points.

* ServiceComponent
* ServiceDiscoveryFactoryPlugin

This part will describes how you plugin your custom extensions into astrix at runtime.



```
META-INF/services/com.avanza.astrix.context.AstrixContextPlugin
com.mycorp.astrixext.CustomRemotingModule
```


```java
@MetaInfServices(AstrixContextPlugin.class)
public class NettyRemotingModule implements AstrixContextPlugin {

	@Override
	public void prepare(ModuleContext moduleContext) {
		moduleContext.bind(ServiceComponent.class, NettyRemotingComponent.class);
		
		moduleContext.importType(AstrixServiceActivator.class);
		moduleContext.importType(ObjectSerializerFactory.class);
		moduleContext.importType(RemotingProxyFactory.class);
		moduleContext.importType(AstrixConfig.class);
		
		moduleContext.export(ServiceComponent.class);
	}

}
```