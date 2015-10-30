> This is a very early draft

### Configuration
Astrix ships with a small standalone configuration framework called [DynamicConfig](https://github.com/AvanzaBank/astrix/tree/master/astrix-config). A configuration property is resolved in the following order:

1. Custom ConfigurationSource's
2. System properties
3. Programmatic configuration set on `AstrixConfigurer`
4. `META-INF/astrix/settings.properties`
5. Default values

Astrix will use the first value found for a given setting. Hence the Custom ConfigurationSource's takes precedence over the Programatic configuration and so on. The custom configuration is plugable by implementing the `ConfigSource` and/or `DynamicConfigSource` spi. By default Astrix will not use any external configuration. The `settings.properties` provides a convenient way to override the default values provided by Astrix. It could be used to set corporate wide default-values by sharing a single `settings.properties` file. For instance it could be used to say that `"com.mycorp"` should be scanned for api-providers, avoiding the need to dupplicate such configurations on every instance of AstrixConfigurer throughout an enterprise.


### System Properties
Astrix allows overriding/defining any setting using system properties. For instance you can define what service registry to use as follows:

```bash
> java -jar my-service -DAstrixServiceRegistry.serviceUri=gs-remoting:jini://*/*/service-registry-space?groups=my-group
```


### Programmatic configuration
All settings may be set programmatically directly on a AstrixConfigurer/AstrixFramworkBean:

```java 
@Bean
public AstrixFrameworkBean astrix() {
	AstrixFrameworkBean astrix = new AstrixFrameworkBean();
	astrix.set(AstrixSettings.SERVICE_REGISTRY_URI, "gs-remoting:jini://*/*/service-registry-space?groups=my-group);
	return astrix;
}
```

### META-INF/astrix/settings.properties
Astrix allows overriding the default-values used for each settings by adding a `META-INF/astrix/settings.properties` to the classpath. At avanza we
share a single "corporate" jar containing the default settings used at Avanza:

#### Example content of META-INF/astrix/settings.properties 
```properties
# Custom ConfigurationFactory used to add custom config sources
com.avanza.astrix.context.AstrixDynamicConfigFactory=se.avanzabank.aza.astrix.integration.AvanzaAstrixDynamicConfigFactory
# Always scan se.avanzabank,nu.placera
AstrixApiProviderScanner.basePackage=se.avanzabank,nu.placera
```


### List of AstrixSettings
AstrixSetting  | Default Value | Description 
:------------------------------------------ | -------------:|:--------------
BEAN_BIND_ATTEMPT_INTERVAL                  | 10 000        | The intervall (in milliseconds) between consecutive bind attemps when a ServicBeanInstance is in UNBOUND state
SERVICE_REGISTRY_URI  (AstrixServiceRegistry.serviceUri)      | null        | ServiceUri used to bind to the service-registry.


### List of AstrixBeanSettings

### Overriding Default AstrixBeanSettings (@DefaultBeanSettings)