### 使用Swagger2构建RESTful API文档 ###
---

### 添加Swagger2 maven依赖

```
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.2.2</version>
</dependency>

<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.2.2</version>
</dependency>
```

### 创建Swagger2配置类

在application类同级创建Swagger2的配置类

```
@Configuration
@EnableSwagger2
public class Swagger2 {

	@Bean
	public Docket createRestApi() {
		return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo()).select()
				.apis(RequestHandlerSelectors.basePackage("org.spring.boot.restfull.controller"))
				.paths(PathSelectors.any()).build();
	}

	private ApiInfo apiInfo() {

		return new ApiInfoBuilder().title("Spring Boot中使用Swagger2构建RESTful APIs")
				.description("更多Spring Boot相关文章请关注：http://blog.didispace.com/").version("1.0").build();
	}

}
```

### 添加文档内容

通过 @ApiOperation注解来给API增加说明、通过 @ApiImplicitParams、 @ApiImplicitParam注解来给参数增加说明。

### 完成配置

启动Spring Boot程序，访问 http://localhost:8080/swagger-ui.html
