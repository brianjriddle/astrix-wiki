> This is a very early draft

### Configuration
Astrix ships with a small standalone configuration framework called [DynamicConfig](https://github.com/AvanzaBank/astrix/tree/master/astrix-config). Every instance of `AstrixContext` has an associated instance of `DynamicConfig` which is used to read configuration at runtime. A configuration property is resolved in the following order:

1. Custom ConfigurationSource's
2. System properties
3. Programmatic configuration set on `AstrixConfigurer`
4. `META-INF/astrix/settings.properties`
5. Default values

Astrix uses the first value found for a given setting. Hence the Custom ConfigurationSource's takes precedence over system properties, which takes precedence over programmatic configuration and so on. The custom configuration is plugable by implementing the `ConfigSource` and/or `DynamicConfigSource` spi. By default Astrix will not use any external configuration.

### Configuration Bootstrap
Astrix uses configuration source 2 - 5 in the table above to bootstrap the configuration. What Astrix does is that it looks for a  `com.avanza.astrix.context.AstrixDynamicConfigFactory` property in these configuration sources. If found, it instantiates the defined `AstrixDynamicConfigFactory` and uses it to create an instance of DynamicConfig which contains all custom configuration sources. All configuration sources used by the DynamicConfig instance returned by the `AstrixDynamicConfigFactory` will take precedence over configuration source 2 - 5. The effective configuration hierarchy used by the given AstrixContext will therefore look as follows:

1. Custom ConfigurationSource's used by the `DynamicConfig` instance returned by `AstrixDynamicConfigFactory` (if present) 
2. System properties
3. Programmatic configuration set on `AstrixConfigurer`
4. `META-INF/astrix/settings.properties`
5. Default values

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
The `settings.properties` provides a convenient way to override the default values provided by Astrix by adding a `META-INF/astrix/settings.properties` file to the classpath. It could be used to set corporate wide default values by sharing a jar containing `settings.properties` file. For instance it could be used to say that `"com.mycorp"` should be scanned for api-providers, avoiding the need to duplicate such configuration on every instance of AstrixConfigurer throughout an enterprise. 

At Avanza we share a single "corporate" jar containing the default settings used at Avanza:

#### Example content of META-INF/astrix/settings.properties 
```properties
# Custom ConfigurationFactory used to add custom configuration sources
com.avanza.astrix.context.AstrixDynamicConfigFactory=se.avanzabank.aza.astrix.integration.AvanzaAstrixDynamicConfigFactory
# Always scan se.avanzabank,nu.placera for @AstrixApiProvider annotated classes
AstrixApiProviderScanner.basePackage=se.avanzabank,nu.placera
```


### List of AstrixSettings
This is a list of the most common `AstrixSettings`

AstrixSetting (configuration property name)  | Default Value | Description 
:------------------------------------------ | -------------:|:--------------
BEAN_BIND_ATTEMPT_INTERVAL (StatefulAstrixBeanInstance.beanBindAttemptInterval) | 10 000        | The intervall (in milliseconds) between consecutive bind attemps when a ServicBeanInstance is in UNBOUND state
SERVICE_REGISTRY_URI  (AstrixServiceRegistry.serviceUri)      | (none) | ServiceUri used to bind to the service-registry.
DYNAMIC_CONFIG_FACTORY (com.avanza.astrix.context.AstrixDynamicConfigFactory) | (none) | 


### List of AstrixBeanSettings

### Overriding Default AstrixBeanSettings (@DefaultBeanSettings)