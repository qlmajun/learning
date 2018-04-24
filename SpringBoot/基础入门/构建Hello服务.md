### 构建Hello服务###
----

### SpringBoot项目结构

Spring Boot的基础结构共有三个文件夹：

* src/main/java 下的程序入口:HelloApplication

* src/main/resources下的配置文件:application.properties

* src/test/下的测试入口:HelloApplicationTests

### Spring boot 的基础模块

* spring-boot-starter ：核心模块，包括自动配置支持、日志和YAML

* spring-boot-starter-test ：测试模块，包括JUnit、Hamcrest、Mockito

maven pom 文件引入

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

引入web模块，需要添加spring-boot-starter-web模块:

```
<dependency>
	 <groupId>org.springframework.boot</groupId>
	 <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 编写Hello服务

* 创建一个HelloController类：

```
@RestController
public class HelloController {

	@RequestMapping("/hello")
	public String index() {
		return "hello";
	}
}
```

* 主程序类SpringBootHelloApplication:

```
@SpringBootApplication
public class SpringBootHelloApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootHelloApplication.class, args);
	}
}
```

* 启动主程序，在浏览器中输入http://localhost:8080/hello ,在页面中输出hello。
