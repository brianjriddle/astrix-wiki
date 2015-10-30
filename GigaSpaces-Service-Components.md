> This is a very early draft

The `astrix-gs` module includes three `ServiceComponent` implementations that can be used to provide services implemented using [GigaSpaces](http://www.gigaspaces.com/developers). 

* GS
* GS_REMOTING
* GS_LOCAL_VIEW


Each servcie component uses the same service-properties, namely the url to the space exporting the service. Only the ServiceComponent name part differs:

ServiceComponent | ServiceUri Format | Example ServiceUri
:------------------ |:------------|:--------------
GS                  | gs:[spaceUrl] |  gs-remoting:jini://*/*/my-space?groups=myGroupName
GS_REMOTING         | gs-remoting:[spaceUrl] |  gs-remoting:jini://*/*/my-space?groups=myGroupName
GS_LOCAL_VIEW       | gs-local-view:[spaceUrl] |  gs-remoting:jini://*/*/my-space?groups=myGroupName
