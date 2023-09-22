# Swagger

## Open API

OpenAPI 规范（OAS），是定义一个标准的、与具体编程语言无关的RESTful API的规范。OpenAPI 规范使得人类和计算机都能在“不接触任何程序源代码和文档、不监控网络通信”的情况下理解一个服务的作用。如果您在定义您的 API 时做的很好，那么使用 API 的人就能非常轻松地理解您提供的 API 并与之交互了。
如果您遵循 OpenAPI 规范来定义您的 API，那么您就可以用文档生成工具来展示您的 API，用代码生成工具来自动生成各种编程语言的服务器端和客户端的代码，用自动测试工具进行测试等等。

### OpenAPI配置全局信息

全局信息描述：

- API Title
- API Description
- Version
- Contact Information
- License

配置全局信息：

- 注解方式
所有这些信息（以及更多）都可以通过在JAX-RS Application类上使用适当的OpenAPI注解来包含在你的Java代码中。因为Quarkus中不需要JAX-RS Application类，所以你可能需要创建一个。它可以是一个扩展了 `javax.ws.rs.core.Application` 的空类。然后这个空类可以用各种OpenAPI注解。

```java
@OpenAPIDefinition(
    tags = {
            @Tag(name="widget", description="Widget operations."),
            @Tag(name="gasket", description="Operations related to gaskets")
    },
    info = @Info(
        title="Example API",
        version = "1.0.1",
        contact = @Contact(
            name = "Example API Support",
            url = "http://exampleurl.com/contact",
            email = "techsupport@example.com"),
        license = @License(
            name = "Apache 2.0",
            url = "https://www.apache.org/licenses/LICENSE-2.0.html"))
)
public class ExampleApiApplication extends Application {
}
```

- 配置文件方式

```properties
quarkus.smallrye-openapi.info-title=Example API
%dev.quarkus.smallrye-openapi.info-title=Example API (development)
%test.quarkus.smallrye-openapi.info-title=Example API (test)
quarkus.smallrye-openapi.info-version=1.0.1
quarkus.smallrye-openapi.info-description=Just an example service
quarkus.smallrye-openapi.info-terms-of-service=Your terms here
quarkus.smallrye-openapi.info-contact-email=techsupport@example.com
quarkus.smallrye-openapi.info-contact-name=Example API Support
quarkus.smallrye-openapi.info-contact-url=http://exampleurl.com/contact
quarkus.smallrye-openapi.info-license-name=Apache 2.0
quarkus.smallrye-openapi.info-license-url=https://www.apache.org/licenses/LICENSE-2.0.html
```

### OpenAPi常用注解

- @Tag
- @Operation
- @Api
- @ApiOperation
- @ApiParam
- @ApiResponses/@ApiResponse
- @ApiModel
- @ApiModelProperty

## 使用swagger

```properties
quarkus.smallrye-openapi.path=/swagger
quarkus.swagger-ui.path=/swagger-ui
```
