> This is a draft

## Configuration
Astrix ships with a small standalone configuration framework called [DynamicConfig](https://github.com/AvanzaBank/astrix/tree/master/astrix-config). Each `AstrixContext` has an associated instance of `DynamicConfig` which is used to read configuration properties at runtime. A configuration property is resolved in the following order:

1. Custom [ConfigSource](http://avanzabank.github.io/astrix/com/avanza/astrix/config/ConfigSource.html)'s
2. System properties
3. Programmatic configuration set on `AstrixConfigurer`
4. `META-INF/astrix/settings.properties`
5. Default values

Astrix uses the first value found for a given [Setting](http://avanzabank.github.io/astrix/com/avanza/astrix/config/Setting.html). Hence the custom configuration sources takes precedence over system properties, which takes precedence over programmatic configuration and so on. The custom configuration is plugable by implementing the `ConfigSource` and/or `DynamicConfigSource` spi. By default Astrix will not use any external configuration.

### Custom Configuration Sources
By default Astrix only reads the well-known configuration sources (2 - 5 above). You can get an `AstrixContext` to read your custom configuration sources by passing an instance of `DynamicConfig` to the `AstrixContext`. There are two ways to do this:

* Set the instance using `AstrixConfigurer.setConfig`
* Define a `com.avanza.astrix.context.AstrixDynamicConfigFactory` property in any of the well-known configuration sources

#### AstrixConfigurer.setConfig
The `AstrixConfigurer.setConfig` property allows passing an instance of `DynamicConfig` to the `AstrixContext`. Any property defined in this `DynamicConfig` instance will take precedence over all of the well-known configuration sources above.

##### Example
```java
DynamicConfig customConfig = DynamicConfig.create(new ArchaiusConfigSource());
astrixConfigurer.setConfig(customConfig);
```

#### AstrixDynamicConfigFactory
The other way to define your custom configuration sources is to implement `AstrixDynamicConfigFactory`. Astrix queries all well-known configuration sources for a `"com.avanza.astrix.context.AstrixDynamicConfigFactory"` property. If that property is defined, Astrix creates an instance of the defined factory and uses it to create a DynamicConfig instance with the custom configuration sources. Note that a `AstrixDynamicConfigFactory` will be totally ignored if a `DynamicConfig` instance is set using `AstrixConfigurer.setConfig` as described above.

##### Example
```java
public class CustomConfigFactory implements AstrixDynamicConfigFactory {
	public DynamicConfig create() {
		return DynamicConfig.create(new ArchaiusConfigSource());
	}
}
```

```properties
# content of META-INF/astrix/settings.properties
com.avanza.astrix.context.AstrixDynamicConfigFactory=com.mycorp.astrix.ext.CustomConfigFactory
```

### System Properties
Astrix allows overriding/defining any setting using system properties. For instance you can define what service registry to use from the command line:

##### Example
```bash
> java -jar my-service.jar -DAstrixServiceRegistry.serviceUri=gs-remoting:jini://*/*/service-registry-space?groups=my-group
```


### Programmatic configuration
All settings may be set programmatically on a AstrixConfigurer/AstrixFramworkBean:

##### Example
```java 
@Bean
public AstrixFrameworkBean astrix() {
	AstrixFrameworkBean astrix = new AstrixFrameworkBean();
	astrix.set(AstrixSettings.SERVICE_REGISTRY_URI, "gs-remoting:jini://*/*/service-registry-space?groups=my-group");
	return astrix;
}
```

### META-INF/astrix/settings.properties
The `settings.properties` file provides a convenient way to override the default value for each setting in Astrix by adding a `META-INF/astrix/settings.properties` file to the classpath. It could be used to set corporate wide default values by sharing a jar containing a `settings.properties` file. For instance it could be used to say that `"com.mycorp"` should be scanned for api-providers, avoiding the need to duplicate such configuration on every instance of AstrixConfigurer throughout an enterprise. 

At Avanza we share a single "corporate" jar containing the default settings used at Avanza:

##### Example content of META-INF/astrix/settings.properties 
```properties
# Custom ConfigurationFactory used to add custom configuration sources
com.avanza.astrix.context.AstrixDynamicConfigFactory=se.avanzabank.aza.astrix.integration.AvanzaAstrixDynamicConfigFactory
# Always scan se.avanzabank,nu.placera for @AstrixApiProvider annotated classes
AstrixApiProviderScanner.basePackage=se.avanzabank,nu.placera
```

## Settings
The `DynamicConfig` instance associated with a `AstrixContext` is used to read _settings_ at runtime. Astrix distinguishes between two types of settings. The first one are `AstrixSettings` which are global to a `AstrixContext`, and the other one are `AstrixBeanSettings` which applies to individual Astrix beans. 

### List of AstrixSettings
This is a list of the most common [AstrixSettings](http://avanzabank.github.io/astrix/com/avanza/astrix/beans/core/AstrixSettings.html)

AstrixSetting (configuration property name)  | Default Value | Description 
:------------------------------------------ | -------------:|:--------------
BEAN_BIND_ATTEMPT_INTERVAL (StatefulAstrixBeanInstance.beanBindAttemptInterval) | 10 000        | The intervall (in milliseconds) between consecutive bind attemps when a ServicBeanInstance is in UNBOUND state
SERVICE_REGISTRY_URI  (AstrixServiceRegistry.serviceUri)      | (none) | ServiceUri used to bind to the service-registry.
DYNAMIC_CONFIG_FACTORY (com.avanza.astrix.context.AstrixDynamicConfigFactory) | (none) | 


### List of AstrixBeanSettings

### Overriding Default AstrixBeanSettings (@DefaultBeanSettings)