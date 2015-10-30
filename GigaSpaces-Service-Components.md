> This is a very early draft

The `astrix-gs` module includes three `ServiceComponent` implementations that can be used to provide services implemented using [GigaSpaces](http://www.gigaspaces.com/developers). 

* GS
* GS_REMOTING
* GS_LOCAL_VIEW


Each `ServiceComponent` uses the same service-properties, which is the url to the space exporting the service. Only the ServiceComponent name part differs as explained by this table:

ServiceComponent  | ServiceUri Format | Example ServiceUri
:------------------ |:------------|:--------------
gs            | gs:[spaceUrl]            |  gs-remoting:jini://*/*/my-space?groups=myGroupName
gs-remoting   | gs-remoting:[spaceUrl]   |  gs-remoting:jini://*/*/my-space?groups=myGroupName
gs-local-view | gs-local-view:[spaceUrl] |  gs-remoting:jini://*/*/my-space?groups=myGroupName
