### 构建web应用 ###
----

### 静态资源访问

Spring Boot 默认提供静态资源目录位置需置于Classpath下，目录名需符合下列规则：

* /static

* /public

* /resources

* /META-INF/resources

ef:我们可以在 src/main/resources/目录下创建 static，在该位置放置一个图片文件1.jpg,启动程序后，尝试访问 http://localhost:8080/1.jpg 如能显示图片，配置成功。

### 渲染web界面

通过@RestController来处理请求，返回的内容为JSON对象，可以通过以模板引擎方式渲染html页面。

Spring Boot提供的默认模板引擎：

* Thymeleaf

* FreeMarker

* Velocity

* Groovy

* Mustache

*注：Spring Boot 建议使用这些引擎，避免使用JSP，若一定要使用JSP将无法实现Spring Boot的多种特性，当使用上面的任一模板时，默认的模板配置路径为：src/main/resources/templates*


如：使用Thymeleaf模板使用

添加 maven依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
  </dependency>
```

### Restfull 注解使用说明

* @Controller：修饰class，用来创建处理http请求的对象

* @RestController：Spring4之后加入的注解，原来在 @Controller中返回json需要 @ResponseBody来配合，如果直接用 @RestController替代 @Controller就不需要再配置 @ResponseBody，默认返回json格式。

* @RequestMapping：配置url映射


### 统一异常处理

Spring Boot中实现了默认的error映射，但是在实际应用中，上面你的错误页面对用户来说并不够友好，我们通常需要去实现我们自己的异常提示。

自定义异常提示界面操作:

* 创建全局异常处理类:通过使用 @ControllerAdvice定义统一的异常处理类，而不是在每个Controller中逐个定义。 @ExceptionHandler用来定义函数针对的异常类型，最后将Exception对象和请求URL映射到 error.html中,在 @ControllerAdvice类中，根据抛出的具体 Exception类型匹配 @ExceptionHandler中配置的异常类型来匹配错误映射和处理。

```
@ControllerAdvice
public class GlobalExceptionHandler {

	private static final String DEFAULT_ERROR_VIEW = "error";

	@ExceptionHandler(value = Exception.class)
	public ModelAndView defaultExceptionHandler(HttpServletRequest request, Exception e) {

		ModelAndView mv = new ModelAndView();

		mv.addObject("exception", e);

		mv.addObject("url", request.getRequestURL());

		mv.setViewName(DEFAULT_ERROR_VIEW);

		return mv;
	}

}
```

异常统一处理JSON格式返回：

只需在 @ExceptionHandler之后加入 @ResponseBody，就能让处理函数return的内容转换为JSON格式。

实现返回JSON格式的异常处理：

* 创建一个统一的JSON返回对象

```
public class ExceptionInfo<T> {

  /** 异常code **/
	private int code;

	/** 异常消息 **/
	private String message;

	/** 请求url **/
	private String url;

	/** 请求响应数据 **/
	private T data;
}
```

* 定义一个RuntimeException的处理类

```
@ControllerAdvice
public class GlobalExceptionHandler {

  @ExceptionHandler(value = RuntimeException.class)
	@ResponseBody
	public ExceptionInfo<String> defaultExceptionHandler(HttpServletRequest request, RuntimeException e) {

		ExceptionInfo<String> excptionInf = new ExceptionInfo<>();

		excptionInf.setCode(500);

		excptionInf.setMessage(e.getMessage());

		excptionInf.setUrl(request.getRequestURL().toString());

		return excptionInf;
	}
}
```
