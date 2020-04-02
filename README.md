# Generator Instructions

This template generates a Spring Cloud Stream (SCSt) application.
It uses version 3 of SCSt which uses function names rather than annotations to configure the channels. See the [reference](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.1.RELEASE/reference/html/spring-cloud-stream.html).

This document makes references to annotation types from the java.util.function package, in particular Supplier, Consumer and Function. It is assumed that the reader has some familiarity with those.

When generating an application, you can set the parameter ```binder={binderType}``` to specify which binder will get pulled in via a Maven dependency. Supported values are kafka (the default), rabbit and solace.

The template works as follows: for each channel, if there is a publisher operation, a Supplier method will be generated. If there is a subscriber operation, a Provider method will get generated. It is possible to have a Function method as well.

Note that this template interprets the AsyncAPI document from the perspective of an application, not an API. This means that when the template sees a subscribe operation, it generates code to subscribe to a topic, not publish to one.

The methods are named as follows. For each operation (i.e. publish or subscribe in each channel), the template looks for the specification extention `x-scs-function-name```. If present, it uses that to name the function. Of one publish operation and one subscribe operation both share the same x-scs-function-name value then a Function method is generated that takes a message from the subscribe operation and publishes it to the publisher operation.

It is an error for more than one subscribe operation or more than one publish operation to share the same x-scs-function-name attribute.

If there is no x-scs-function-name attribute, the generator checks the operation's operationId value. If set, that is used as the function name. Otherwise a name will be generated by taking the channel name, removing non-alphabetic characters, converting to camel case, and appending 'Supplier' or 'Producer.'

with the name ```{operationName}Supplier```.

For example, if there is a channel called store/process with a publisher operation with a payload called Order, This method will get generated:

 ```
@Bean
public Supplier<Order> storeProcessSupplier () {
  // Add business logic here.
  return null;
}
```

The template generates an application.yaml file that has bindings configured corresponding to the functions in the Java code.

## Destination Overrides

There are two specification extentions you can use to shape how the bindings are configured. You can add the following to a subscription operation:

```x-scs-destination``` : This overrides the destination value in a binder. This is useful when you are using the Solace binder and you are following the Solace pattern of publishing to topics but subscribing from queues. In this case the s-scs-destination value would be treated as the name of a queue.

```x-scs-group``` : This will add the group value on a binding.

## Parameters

Parameters can be passed to the generator using command line arguments in the form ```-p param=value```. Here is a list of the parameters that can be used with this template. In some cases these can be put into the AsyncAPI documents using the specification extensions feature. In those cases, the 'info' prefix means that it belongs in the info section of the document.

Parameter | Extension | Default | Required | Description
----------|-----------|---------|----------|--
actuator    |              | false | | If present, it adds the dependencies for spring-boot-starter-web, spring-boot-starter-actuator and micrometer-registry-prometheus.
artifactId  |  info.x-artifact-id | | Yes | The Maven artifact id.
artifactType | | application | | The type of project to generate, application or library. When generating an application, the pom.xml file will contain the complete set of dependencies required to run an app, and it will contain an Application class with a main function. Otherwise the pom file will include only the dependencies required to compile a library.
binder | | kafka | | The name of the binder implementation, one of kafka, rabbit or solace. Default: kafka. If you need other binders to be supported, please let us know!
groupId | info.x-group-id | | Yes | The Maven group id.
host | | tcp://localhost:55555 | |The host connection property. Currently this only works with the Solace binder. When other binders are used this parameter is ignored.
javaClass | info.x-java-class | Application | | The name of the main class. Only used when artifactType is 'application'.
javaPackage | info.x-java-package | | | The Java package of the generated classes. If not set then the classes will be in the default package.
msgVpn | | default | | The message vpn connection property. Currently this only works with the Solace binder. When other binders are used this parameter is ignored.
password | | default | | The client password connection property. Currently this only works with the Solace binder. When other binders are used this parameter is ignored.
reactive | | false | | If true, the generated functions will use the Reactive style and use the Flux class.
solaceSpringCloudVersion | info.x-solace-spring-cloud-version | | Only when artifactType is 'application' and binder is 'solace' | The version of the solace-spring-cloud-bom dependency used when generating an application. Example: 1.0.0
springCloudVersion | info.x-spring-cloud-version | | Only when artifactType is 'application' | The version of the spring-cloud-dependencies BOM dependency used when generating an application. Example: Hoxton.RELEASE.
springCloudStreamVersion | info.x-spring-cloud-stream-version | | Only when artifactType is 'library' | The version of the spring-cloud-stream dependency specified in the Maven file, when generating a library. When generating an application, the spring-cloud-dependencies BOM is used instead. Example: 3.0.1.RELEASE.
username | | default | |The client username connection property. Currently this only works with the Solace binder. When other binders are used this parameter is ignored.

## Specification Extensions

The following specification extensions are supported. In some cases their value can be provided as a command line parameter. The 'info' prefix means that it belongs in the info section of the document.

Extension | Parameter | Default | Required | Description
----------|-----------|---------|----------|-------------
info.x-artifact-id | artifactId | | Yes | The Maven artifact id.
info.x-group-id | groupId | | Yes | The Maven group id.
info.x-java-class | javaClass | Application | | The name of the main class. Only used when artifactType is 'application'.
info.x-java-package | javaPackage | | | The Java package of the generated classes. If not set then the classes will be in the default package.
info.x-solace-spring-cloud-version | solaceSpringCloudVersion | | Only when artifactType is 'application' and binder is 'solace' | The version of the solace-spring-cloud BOM dependency used when generating an application. Example: 1.0.0
info.x-spring-cloud-version | info.x-spring-cloud-version | | Only when artifactType is 'application' | The version of the spring-cloud-dependencies BOM dependency used when generating an application. Example Hoxton.SR1
info.x-spring-cloud-stream-version | springCloudStreamVersion | | Only when artifactType is 'library' | The version of the spring-cloud-stream dependency specified in the Maven file, when generating a library. When generating an application, the spring-cloud-dependencies BOM is used instead. Example 3.0.1.RELEASE.
operation.x-scs-function-name | | | | This specifies the base function name to use on a publish or subscribe operation. For example, if it is set to myFunction, then the template will create functions myFunctionSupplier and/or myFunctionConsumer. If the same name is used on one subscribe operation and one publish operation, a processor function will be generated.
channel.subscription.x-scs-destination | | | | This overrides the destination on an incoming binding. It can be used to specify, for example, the name of a queue to subscribe to instead of a topic.
channel.subscription.x-scs-group | | | | This is used to specify the group property of an incoming binding.




