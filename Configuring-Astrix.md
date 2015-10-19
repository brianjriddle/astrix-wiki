# Configuration
Astrix ships with a small standalone configuration framework called [DynamicConfig](https://github.com/AvanzaBank/astrix/tree/master/astrix-config). A configuration property is resolved in the following order:

1. Custom ConfigurationSource's
2. System properties
3. Programmatic configuration set on `AstrixConfigurer`
4. `META-INF/astrix/settings.properties`
5. Default values

Astrix will use the first value found for a given setting. Hence the Custom ConfigurationSource's takes precedence over the Programatic configuration and so on. The custom configuration is plugable by implementing the `ConfigSource` and/or `DynamicConfigSource` spi. By default Astrix will not use any external configuration. The `settings.properties` provides a convenient way to override the default values provided by Astrix. It could be used to set corporate wide default-values by sharing a single `settings.properties` file. For instance it could be used to say that `"com.mycorp"` should be scanned for api-providers, avoiding the need to dupplicate such configurations on every instance of AstrixConfigurer throughout an enterprise.

# AstrixSettings
AstrixSetting  | Default Value | Description 
:-------------------------- | -------------:|:--------------
BEAN_BIND_ATTEMPT_INTERVAL  | 10 000        | The intervall (in milliseconds) between consecutive bind attemps when a ServicBeanInstance is in UNBOUND state




# AstrixBeanSettings