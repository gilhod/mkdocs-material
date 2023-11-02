# OpenAPI Code Generation Maven Plugin ![language](https://img.shields.io/badge/language-java%2017-orange.svg)
A maven plugin for generating [vertx-web-openapi](https://vertx.io/docs/vertx-web-openapi/java/) server code and models from an [OpenAPI 3.0](https://swagger.io/specification/) spec file.\
The plugin uses [openapi-generator](https://github.com/OpenAPITools/openapi-generator) to generate code and the implementation is based off [openapi-generator-maven-plugin](https://github.com/OpenAPITools/openapi-generator/tree/master/modules/openapi-generator-maven-plugin).\
I highly encourage the reader to visit those above links before reading this in order to have a better context.


## 📦️ Usage
This section explains how to generate server code, implement, and use it.
### Generate code
Follow those simple steps in order to use the plugin to generate code.
1. Configure and add plugin to your maven project
```xml
<plugin>
    <groupId>com.viber.core.infra</groupId>
    <artifactId>openapi-codegen-plugin</artifactId>
    <version>1.0.4</version>
    <configuration>
        <inputSpec>${path_to_openapi_spec}</inputSpec>
        <output>${path_to_output_directory}</output>
        <packageName>${generated_code_root_package}</packageName>
    </configuration>
    <executions>
        <execution>
            <id>executionId</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
2. Build the project using maven.

### Generated code
Generated code follows this structure:
```
<output-root>
└── src
    └── main
        └── java
            └── <root-package>
                ├── ApisMounter.java
                ├── ... API classes
                ├── dto
                │   └── ... DTO classes
                ├── request
                │   └── ... request classes
                └── response
                    └── ... response classes
```
Note that the code will always be generated under <output_root>/src/main/java/<root_package_path>.
#### Example
See [example output](docs/example/generated) for [petstore.yaml](docs/example/petstore.yaml) spec file.

#### ApisMounter

| Filename | Description | Codegen Template | Example |
| ------ | ------ | ------ | ------ | 
| `ApisMounter.java` | An aggregator object used for registering all generated operations to the [vertx-web-openapi#RouterBuilder](https://vertx.io/docs/vertx-web-openapi/java/#_routerbuilder).<br>This class also handles the creation/extraction of the current request context.<br>See [example usage](src/it/test-solution/src/main/java/solution/OpenAPIServer.java).| [apiRouterBuilderMounter.mustache](src/main/resources/vertx/apiRouterBuilderMounter.mustache) | [ApisMounter.java](docs/example/generated/ApisMounter.java) | 

#### API classes

| Generated Per | Filename | Description | Codegen Template | Example |
| ------ | ------ | ------ | ------ | ------ | 
| [Unique tag](https://swagger.io/docs/specification/grouping-operations-with-tags/) | `<tag_name>Api.java` | Interface for all operations under a certain tag, as a developer you will implement this interface.<br>Each method receives a [RoutingContext](https://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#:~:text=public%20interface%20RoutingContext,(Object)%20of%20the%20router.), [generated request input](#request-classes) and returns a Future of a [generated response](#response-classes).<br>Method names are determined by [operationId](https://swagger.io/docs/specification/paths-and-operations/). | [api.mustache](src/main/resources/vertx/api.mustache) | [PetApi.java](docs/example/generated/PetApi.java) | 
| [Unique tag](https://swagger.io/docs/specification/grouping-operations-with-tags/) | `<tag_name>ApiProcessor.java` | A middleware object for serializing/deserializing.<br>This object requires a concnrete implementation of its corrisonding `<tag_name>Api` interface.<br>Method names are determined by [operationId](https://swagger.io/docs/specification/paths-and-operations/). | [apiProcessor.mustache](src/main/resources/vertx/apiProcessor.mustache) | [PetApiProcessor.java](docs/example/generated/PetApiProcessor.java) | 


#### DTO classes
| Generated Per | Filename | Description | Codegen Template | Example |
| ------ | ------ | ------ | ------ | ------ | 
| [Data model](https://swagger.io/docs/specification/data-models/) | `<model_name>DTO.java` | A POJO representing a model schema from the spec file. | [model.mustache](src/main/resources/vertx/model.mustache) | [PetDTO.java](docs/example/generated/dto/PetDTO.java) | 

#### Request classes
| Generated Per | Filename | Description | Codegen Template | Example |
| ------ | ------ | ------ | ------ | ------ | 
| [Operation](https://swagger.io/docs/specification/paths-and-operations/) | `<operation_id>Request.java` | A POJO representing input parameters for an operation.<br>Possible input sources: body, path params, query params.<br> | [operationRequest.mustache](src/main/resources/vertx/operationRequest.mustache) | [AddPetRequest.java](docs/example/generated/request/AddPetRequest.java), [DeletePetRequest.java](docs/example/generated/request/DeletePetRequest.java) | 

#### Response classes
| Generated Per | Filename | Description | Codegen Template | Example |
| ------ | ------ | ------ | ------ | ------ | 
| [Operation](https://swagger.io/docs/specification/paths-and-operations/) | `<operation_id>Response.java` | A sealed class representing all possible [responses](https://swagger.io/docs/specification/describing-responses/) for an operation.<br>An inline (package-private) subclass named `<operation_id><http_status>Response` is generated for each response as well as a static creating method:<br>`<operation_id>Response#<operation_id><http_status>Response(...)`.<br> | [operationResponse.mustache](src/main/resources/vertx/operationResponse.mustache) | [AddPetResponse.java](docs/example/generated/response/AddPetResponse.java), [DeletePetResponse.java](docs/example/generated/response/DeletePetResponse.java) | 

#### Overriding
The generator will override existing files when generating but will ignore unrelated files that resides in the output directory.\
It is highly reccomended to use [maven-clean-plugin](https://maven.apache.org/plugins/maven-clean-plugin/) in order to clear output directory before each build.

#### Generated Code Dependencies
Generated code uses `vertx-web-openapi` and `lombok`.\
Add those dependencies to your project if missing.

```xml
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-web-openapi</artifactId>
    <version>4.2.4</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
</dependency>
<dependency>
    <groupId>com.viber.core.infra</groupId>
    <artifactId>context</artifactId>
    <version>1.0.1</version>
</dependency>
```


### Using Generated Code
1. Implement API interfaces. [see example](src/it/test-solution/src/main/java/solution/SolutionTestApiImpl.java).
2. Wire your implementation to the vertx-web-openapi server. [see example](src/it/test-solution/src/main/java/solution/OpenAPIServer.java).

## 🖇 Request Context
The generated code is dependent on the library `com.viber.core.infra:context`.\
We use the single interface Context.java from the library to represent a request or a flow that is propagated across services/systems, read more about [Context Library](https://lab.vibelab.net/backend/dev-infra-libraries/-/tree/master/context).

To set the context you will have to add a [root handler](https://lab.vibelab.net/backend/dev-infra-libraries/-/tree/master/context) that will create and store the context on the Vert.x's [RoutingContext](https://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) object with a specific key.\
Currently, the key should is hardcoded to be `request-context` and the value is read and used in the generated [ApisMounter](#apismounter).\
If the key is not present a random UUID will be generated.

Here's an example of such a root handler that reads a header called "x-request-id" and supplies it as a context:
```java
routerBuilder.rootHandler(routingContext -> {
    var traceId = routingContext.request().headers().get("x-request-id")
    Context context = () -> traceId;
    routingContext.put("request-context", context);
    routingContext.next();
});
```

## ☑ Plugin Configuration
| Name | Description | Required | Default |
| ------ | ------ | :------: |:------: |
| inputSpec | Path to input OpenAPI 3.0 spec file | :heavy_check_mark: | - |
| output | Path to output folder, will be appended with "src/main/java"  | :heavy_check_mark: | - |
| packageName | Generated classes root package | :heavy_check_mark: | - |
| verbose | Code generation verbose logging | :x: | false |

## ✋ OpenAPI 3.0 Unsuported Features
- [oneof anyof allof not](https://swagger.io/docs/specification/data-models/oneof-anyof-allof-not/)
- [vendor extentions](https://swagger.io/docs/specification/openapi-extensions/)
- [enum in path/query](https://swagger.io/docs/specification/data-models/enums/)
- Inline schemas (Define everything inside "model")
- non-boolean [additional properties](https://json-schema.org/understanding-json-schema/reference/object.html#additional-properties) schemas
## 🛠 Support
Please contact Itzhak Eretz-Kdosha (itzhak@viber.com)

## :construction_worker: Maintainer Notes
- The project's integration tests also depends on `com.viber.core.infra:context` which is located on our AWS CodeArtifact. Because of that you must have a fresh `CODEARTIFACT_AUTH_TOKEN` environment variable in order to build it.\
  If you with to skip the integration tests use `skip-openapi-codegen-it` maven profile which will disable `maven-invoker-plugin` that we use to run the IT.
- If a generated code dependency (for example, vertx-web-openapi) version is changed, please update the [Generated Code Dependencies](#generated_code_dependencies) section and the [integration test pom file](src/it/test-solution/pom.xml).

