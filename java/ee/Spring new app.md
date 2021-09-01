https://www.tutorialspoint.com/spring/spring_java_based_configuration

## @Configuration & @Bean Annotations

Annotating a class with the  **@Configuration**  indicates that the class can be used by the Spring IoC container as a source of bean definitions. The  **@Bean**  annotation tells Spring that a method annotated with @Bean will return an object that should be registered as a bean in the Spring application context. The simplest possible @Configuration class would be as follows −

```java
package com.tutorialspoint;  
import org.springframework.context.annotation.*;  
@Configuration  
public  class  HelloWorldConfig  {  
	@Bean  
	public  HelloWorld helloWorld(){  
		return  new  HelloWorld();  
	}  
}
```
The above code will be equivalent to the following XML configuration −

```xml
<beans>  
	<bean  id  =  "helloWorld"  class  =  "com.tutorialspoint.HelloWorld"  />  
</beans>
```

Here, the method name is annotated with @Bean works as bean ID and it creates and returns the actual bean. Your configuration class can have a declaration for more than one @Bean. Once your configuration classes are defined, you can load and provide them to Spring container using  _AnnotationConfigApplicationContext_  as follows −

```java
public  static  void main(String[] args)  {  
	ApplicationContext ctx =  new AnnotationConfigApplicationContext(HelloWorldConfig.class);  

	HelloWorld helloWorld = ctx.getBean(HelloWorld.class); 
	helloWorld.setMessage("Hello World!"); 
	helloWorld.getMessage();  
}
```