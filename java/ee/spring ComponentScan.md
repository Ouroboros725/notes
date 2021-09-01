https://springframework.guru/spring-component-scan/

## @ComponentScan – Identifying Base Packages

The `@ComponentScan` annotation is used with the `@Configuration` annotation to tell Spring the packages to scan for annotated components. `@ComponentScan` is also used to specify base packages and base package classes using `thebasePackageClasses` or `basePackages` attributes of `@ComponentScan`.

The `basePackageClasses` attribute is a type-safe alternative to `basePackages`. When you specify basePackageClasses, Spring will scan the package (and subpackages) of the classes you specify.

A Java class annotated with `@ComponentScan` with the `basePackageClassesattribute` is this.

##### BlogPostsApplicationWithComponentScan.java

```java
//package guru.springframework.blog;

import guru.springframework.blog.componentscan.example.demopackageB.DemoBeanB1;
import org.springframework.boot.SpringApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = {"guru.springframework.blog.componentscan.example.demopackageA",
		"guru.springframework.blog.componentscan.example.demopackageD",
		"guru.springframework.blog.componentscan.example.demopackageE"},
		basePackageClasses = DemoBeanB1.class)
public  class BlogPostsApplicationWithComponentScan {
	public  static  void  main(String[] args)  {
		ApplicationContext context = SpringApplication.run(BlogPostsApplicationWithComponentScan.class,args);
		System.out.println("Contains A "+context.containsBeanDefinition("demoBeanA"));
		System.out.println("Contains B2 " + context.containsBeanDefinition("demoBeanB2"));
		System.out.println("Contains C " + context.containsBeanDefinition("demoBeanC"));
		System.out.println("Contains D " + context.containsBeanDefinition("demoBeanD"));
	}
}
```

The output on running the main class is this.  
![Output of BlogPostsApplicationWithComponentScan.java Class](http://springframework.guru/wp-content/uploads/2017/11/Output-of-BlogPostsApplicationWithComponentScan.java_-1024x150.png)

The `@ComponentScan` annotation uses the `basePackages` attribute to specify three packages (and subpackages) that will be scanned by Spring. The annotation also uses the `basePackageClasses` attribute to declare the `DemoBeanB1` class, whose package Spring Boot should scan.

As `demoBeanC` is in a different package, Spring did not find it during component scanning.