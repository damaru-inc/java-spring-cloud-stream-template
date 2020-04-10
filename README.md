# Java Spring Cloud Stream Generator Template

This template generates a Spring Cloud Stream (SCSt) microservice. It uses version 3 of SCSt which uses function names to configure the channels. See the [reference](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.1.RELEASE/reference/html/spring-cloud-stream.html). The generated microservice is a Maven project so it can easily be imported into your IDE. 

The Spring Cloud Stream microservice generated using this template will be an _almost ready to run_ Spring Boot app. The microservice will contain a java class, by default _Application.java_, which includes methods which publish or subscribe events as defined in the AsyncAPI document. These generated methods include Supplier, Consumer and Function functional interfaces from the [java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html) package. These methods will already be pre-configured to publish to and consume from the channels as defined in the AsyncAPI. This configuration is located in the `spring.cloud.stream` section of the generated application.yml file.


## How it works
Note that this template interprets the AsyncAPI document from the perspective of an application, not an API. This means that when the template sees a subscribe operation, it generates code to subscribe to a topic, not publish to one.

### Which Methods are created
The template works as follows:
* By default for each channel in the AsyncAPI document, if there is a _publish_ operation a _Supplier_ method will be generated, and for a _subscribe_ operation a _Consumer_ method will get generated. 
* To customize this default behavior you can make use of the `x-scs-function-name` extension. If one _publish_ operation and one _subscribe_ operation share a `x-scs-function-name` attribute then a _java.util.function.Function_ method will be created which uses the _subscribe_ operation's message as the input and the _publish_ operation's message as the output to the generated Function method. It will also wire up the proper channel information in the generated application.yaml file.
* Note that at this time the generator does not support the creation of functions that have more than one _publish_ and/or more than one _subscribe_ operation with the same ```x-scs-function-name``` attribute. This scenario will result in error. 


### Method Naming
The generated methods are named as follows: 
* For each operation (i.e. _publish_ or _subscribe_ in each channel), the template looks for the specification extention ```x-scs-function-name```. If present, it uses that to name the function. 
* If using the same ```x-scs-function-name``` on one _publish_ operation and one _subscribe_ operation to create a Function the name of the generated method will be the ```x-scs-function-name```
* If there is no ```x-scs-function-name``` attribute, the generator checks the operation's operationId value. If set, that is used as the function name. 
* If there is no ```x-scs-function-name``` or operationId available, then a name will be generated by taking the channel name, removing non-alphanumeric characters, converting to camel case, and appending 'Supplier' or 'Producer.' For example, if there is a channel called store/process with a publisher operation with a payload called Order, This method will get generated:
 ```
@Bean
public Supplier<Order> storeProcessSupplier () {
  // Add business logic here.
  return null;
}
```
### Property Naming

When converting from property names to Java field names, the property names are first converted to camelCase, removing non-alphanumeric characters in the process. If the resulting name ends up being a Java keyword, it is prepended with an underscore.

### Application vs Library

By default, this will generate a runnable Spring Boot application. If you set the ```artifactType``` parameter to ```library```, then it will generate a project without a main Application class and without the main Spring Boot dependencies. That will produce a library that can be imported into another application as a Maven artifact. It will contain the model classes and the Spring Cloud Stream configuration.

Doing that allows you to hava a clean separation between the generated code and your hand-written code, and ensures that regenerating the library will not overwrite your business logic.

## How to Use This Template

Install the AsyncAPI Generator
```
npm install -g @asyncapi/generator
```

Run the Generator using the Java Spring Cloud Stream Template
```
ag ~/AsyncApiDocument.yaml https://github.com/asyncapi/java-spring-cloud-stream-template.git
```

Run the Generator using the Java Spring Cloud Stream Template with Parameters
```
ag -p binder=solace -p artifactId=ExampleArtifactId -p groupId=com.example -p javaPackage=com.example.foo -p solaceSpringCloudVersion=1.0.0 -p springCloudStreamVersion=Horsham.SR3 -p springCloudVersion=Hoxton.SR3 ~/AsyncApiDocument.yaml https://github.com/asyncapi/java-spring-cloud-stream-template.git
```


## Configuration Options

### Destination Overrides

There are two specification extentions you can use to shape how the bindings are configured. You can add the following to a _subscribe_ operation:

```x-scs-destination``` : This overrides the destination value in a binder. This is useful when you are using the Solace binder and you are following the Solace pattern of publishing to topics and consuming from queues. In this case the x-scs-destination value would be treated as the name of the queue which your microservice will consume from.

```x-scs-group``` : This will add the group value on a binding which configures your microservice to use [Consumer Groups](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/current/reference/html/spring-cloud-stream.html#consumer-groups)

### Parameters

Parameters can be passed to the generator using command line arguments in the form ```-p param=value -p param2=value2```. Here is a list of the parameters that can be used with this template. In some cases these can be put into the AsyncAPI documents using the specification extensions feature. In those cases, the 'info' prefix means that it belongs in the info section of the document.

Parameter | Extension | Default | Description
----------|-----------|---------|---
actuator    |              | false | If true, it adds the dependencies for spring-boot-starter-web, spring-boot-starter-actuator and micrometer-registry-prometheus.
artifactId  |  info.x-artifact-id | project-name | The Maven artifact id.
artifactType | | application | The type of project to generate, application or library. When generating an application, the pom.xml file will contain the complete set of dependencies required to run an app, and it will contain an Application class with a main function. Otherwise the pom file will include only the dependencies required to compile a library.
binder | | kafka | The name of the binder implementation, one of kafka, rabbit or solace. Default: kafka. If you need other binders to be supported, please let us know!
groupId | info.x-group-id | com.company | The Maven group id.
host | | tcp://localhost:55555 | The host connection property. Currently this only works with the Solace binder. When other binders are used this parameter is ignored.
javaClass | info.x-java-class | Application | The name of the main class. Only used when artifactType is 'application'.
javaPackage | info.x-java-package | | The Java package of the generated classes. If not set then the classes will be in the default package.
msgVpn | | default | The message vpn connection property. Currently this only works with the Solace binder. When other binders are used this parameter is ignored.
password | | default | The client password connection property. Currently this only works with the Solace binder. When other binders are used this parameter is ignored.
reactive | | false | If true, the generated functions will use the Reactive style and use the Flux class.
solaceSpringCloudVersion | info.x-solace-spring-cloud-version | 1.0.0 | The version of the solace-spring-cloud-bom dependency used when generating an application.
springBootVersion | info.x-spring-boot-version | 2.2.6.RELEASE | The version of Spring Boot used when generating an application. 
springCloudVersion | info.x-spring-cloud-version | Hoxton.SR3 | The version of the spring-cloud-dependencies BOM dependency used when generating an application. 
springCloudStreamVersion | info.x-spring-cloud-stream-version | 3.0.3.RELEASE | The version of the spring-cloud-stream dependency specified in the Maven file, when generating a library. When generating an application, the spring-cloud-dependencies BOM is used instead
username | | default | The client username connection property. Currently this only works with the Solace binder. When other binders are used this parameter is ignored.

## Specification Extensions

The following specification extensions are supported. In some cases their value can be provided as a command line parameter. The 'info' prefix means that it belongs in the info section of the document.

Extension | Parameter | Default | Description
----------|-----------|---------|-------------
info.x-artifact-id | artifactId | project-name | The Maven artifact id.
info.x-group-id | groupId | com.company | The Maven group id.
info.x-java-class | javaClass | Application | The name of the main class. Only used when artifactType is 'application'.
info.x-java-package | javaPackage | | The Java package of the generated classes. If not set then the classes will be in the default package.
info.x-solace-spring-cloud-version | solaceSpringCloudVersion | 1.0.0 | The version of the solace-spring-cloud BOM dependency used when generating an application.
info.x-spring-boot-version | info.x-spring-boot-version | 2.2.6.RELEASE | The version of the Spring Boot used when generating an application.
info.x-spring-cloud-version | info.x-spring-cloud-version | Hoxton.SR3 | The version of the spring-cloud-dependencies BOM dependency used when generating an application.
info.x-spring-cloud-stream-version | springCloudStreamVersion | 3.0.3.RELEASE | The version of the spring-cloud-stream dependency specified in the Maven file, when generating a library. When generating an application, the spring-cloud-dependencies BOM is used instead.
operation.x-scs-function-name | | | This specifies the base function name to use on a publish or subscribe operation. If the same name is used on one subscribe operation and one publish operation, a processor function will be generated.
channel.subscription.x-scs-destination | | | This overrides the destination on an incoming binding. It can be used to specify, for example, the name of a queue to subscribe to instead of a topic.
channel.subscription.x-scs-group | | | This is used to specify the group property of an incoming binding.




