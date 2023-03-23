# Swagger UI

### Task completed

1. Set up Swagger and Swagger UI page

<br>
## 1. Maven Dependencies

```xml
<dependency>
	<groupId>org.apache.camel</groupId>
	<artifactId>camel-swagger-java</artifactId>
	<version>3.14.5</version>
</dependency>
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-boot-starter</artifactId>
	<version>3.0.0</version>
</dependency>
```

<br>
## 2. Properties file

Add the following into the `application.properties` file.

```php
# Used for the swagger ui, where '/camel/api-doc' is the path the doc located
springfox.documentation.swagger.v2.path=/camel/api-doc
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```

<br>
## 3. Adjust the Rest Configuration

At the `restConfiguration` that defined the RESTful api paths, add the following to enable a JSON swagger doc page (located at: **localhost:9090/camel/api-doc**). 

 ******`.contextPath` is set to *“/camel”* as the camel rest default would have such a prefix. Notice that only APIs defined using `rest()` would appeared in the swagger doc.

```java
// Class defined Rest call path
public class RestApi extends RouteBuilder {
	@Override
	public void configure(){
		restConfiguration()
			.component("servlet")
			.port(9090)
			.host(localhost)
			.bindingMode(RestBindingMode.auto)
			// Added the following
			.contextPath("/camel")
			.dataFormatProperty("prettyPrint", "true")
			.apiContextPath("/api-doc")
			.apiProperty("api.title", "Your API DOC Title")
			.apiProperty("api.version", "Your API DOC Version")
			.apiProperty("cors", "true");
		
		// Define rest path here
		// ...
	}
}
```

<br>
## 4. Add Information to the APIs

To make the Swagger more informative, add information such as description to the apis.

```java
// Class defined rest call path
...
rest("/somePath")
	.get()
	// Add information
	.description("This is the description.")
	.consumes("text/plain")
	.produces("text/plain")
	.to("direct:endPoint");
	
	.post()
	// Turn input json request to your class
	.type(YourOutputType.class)
	.to("direct:endPoint2");
```

<br>
## 5. Config Class for Swagger UI

Create a swagger configuration class with the following to enable a Swagger UI page. Enter the page with **[http://localhost:9090/swagger-ui/](http://localhost:9090/swagger-ui/) .**

```java
@Configuration
public class SwaggerConfig {
  @Bean
  public Docket api() {
    return new Docket(DocumentationType.SWAGGER_2)
            .select()
            .apis(RequestHandlerSelectors.any())
            .paths(PathSelectors.any())
            .build();
  }
}
```