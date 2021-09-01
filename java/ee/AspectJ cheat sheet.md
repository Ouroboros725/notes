https://blog.espenberntsen.net/2010/03/20/aspectj-cheat-sheet/

##   
@AspectJ cheat sheet

[with 17 comments](https://blog.espenberntsen.net/2010/03/20/aspectj-cheat-sheet/#comments)

This cheat sheet uses AspectJ’s @AspectJ style. It’s also possible to use the original  [AspectJ syntax](http://eclipse.org/aspectj/doc/released/progguide/language.html "http://eclipse.org/aspectj/doc/released/progguide/language.html")  like this  [example](http://www.ibm.com/developerworks/java/library/j-aopwork8/#code8)  demonstrates, but I prefer to use standard Java classes with the AspectJ logic inside annotations.

## Pointcuts

The definition of a pointcut from the AspectJ  [homepage](http://www.eclipse.org/aspectj/doc/next/progguide/semantics-pointcuts.html):

> A pointcut is a program element that picks out join points and exposes data from the execution context of those join points. Pointcuts are used primarily by advice. They can be composed with boolean operators to build up other pointcuts.

A pointcut example:

[![](https://espenberntsen.files.wordpress.com/2010/03/aspectj-pointcut-explanation.png?w=700&h=231 "aspectj-pointcut-explanation")](https://espenberntsen.files.wordpress.com/2010/03/aspectj-pointcut-explanation.png)

## Pointcut designators

A method pointcut:
```java
@Pointcut("[method designator](* aspects.trace.demo.*.*(..))")
public void traceMethodsInDemoPackage() {}
```

-   call – The pointcut will find all methods that calls a method in the demo package.
-   execution – The pointcut will find all methods in the demo package.
-   withincode – All the statements inside the methods in the demo package.

A type pointcut:
```java
@Pointcut("[type designator](*..*Test)")
public void inTestClass() {}
```

-   within – all statements inside the a class that ends with Test.

A field pointcut:
```java
@Pointcut("[field designator](private org.springframework.jdbc.core.JdbcTemplate " +
		"integration.db.*.jdbcTemplate)")
public void jdbcTemplateGetField() {}
```

-   get – all reads to jdbcTemplate fields of type JdbcTemplate in the  integration.db  package. Includes all methods on this field if it’s an object.
-   set – when you set the jdbcTemplate field of type JdbcTemplate in the  integration.db  package to a new value.

## Signature pointcuts

This chapter explains more advanced signature pointcuts than illustrated on the image above.

Support for sub packages is provided with “..”:
```java
@Pointcut("within(*..*Test)")
public void inTestClass() {}
```

To find the joinpoints inside a type that ends with Test inside the ..demo package:
```java
@Pointcut("within(com.redpill.linpro.demo..*Test)")
public void inDemoProjectTestClass() {}
```

All statements inside all classes except test classes in the ..demo package with the “!” before the designator:
```java
@Pointcut("!within(com.redpill.linpro.demo..*Test)")
public void notInDemoProjectTestClass() {}
```

All methods in the Service class or a subtype of it:
```java
@Pointcut("execution(void *..service.Service+.*(..))")
public void servicePointcut() {}
```

All getCoffeeType methods in a class that begins with CoffeeTypeRepository:
```java
@Pointcut("execution(CoffeeType integration.db.CoffeeTypeRepository*.getCoffeeType(CoffeeTypeName))" +
		" && args(coffeeTypeName)")
public void getCoffeeTypePointcut(CoffeeTypeName coffeeTypeName) {}
```

Note that the pointcut also contains args(coffeeTypeName) and that the Java method has a method with a CoffeeTypeName as input parameter. An advice that advices this pointcut must also have this input parameter.

This field pointcut finds all the places in the integration package where a field named jdbcTemplate gets a new value of type JdbcOperations or a subtype of it like JdbTemplate.
```java
@Pointcut("set(private org.springframework.jdbc.core.JdbcOperations+ " +
		"integration..*.jdbcTemplate)")
public void jdbcTemplateSetField() {}
```

## Annotation pointcuts

A pointcut can declare an annotation before the signature pattern.

An example of an annotation used as a marker interface:
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PerformanceLogable {}
```

All it does is to provide meta information to methods.

A pointcut that finds all methods marked with the @PerformanceLogable on the classpath:
```java
@Pointcut("execution(@aspects.log.performance.PerformanceLogable * *(..))")
public void performanceLogableMethod() {}
```

The @Transactional annotation supports both method and type target. Which means it can be used with both method and type designators. The pointcut below is the same as the performanceLogableMethod pointcut, except it finds methods with Spring’s @Transactional annotation.
```java
@Pointcut("execution(@org.springframework.transaction.annotation.Transactional * *(..))")
public void transactionalMethod() {}
```

A pointcut that finds all constructors marked with @Inject and have an integer as input parameter:
```java
@Pointcut("execution(@javax.inject.Inject *.new(Integer)) && args(integer)")
public void constructorAnnotatedWithInjectAndIndemoPackage(Integer integer) {}
```

The @Service annotation has target set to type and can therefore only annotate types. The pointcut below will find all statements in all types marked with @Service.
```java
@Pointcut("within(@org.springframework.stereotype.Service *)")
public void serviceBean() {}
```

The joinpoint will be all statements inside a type marked with the @Component annotation or a specialization of it:
```java
@Pointcut("within(@(@org.springframework.stereotype.Component *) *)")
public void beanAnnotatedWithComponentOrASpecializationOfIt() {}
```

Finds all statements that’s not inside a method marked with @Test:
```java
@Pointcut("!withincode(@org.junit.Test * *(..))")
public void notInTestMethod() {}
```

Finds all methods with one or more parameters marked with the @MyParamAnnotation:
```java
@Pointcut("execution(public * *(.., @aspects.MyParamAnnotation (*), ..))")
public void methodWithAnnotationOnAtLeastOneParameter() {}
```

For a full example with explanations, see this  [post](http://stackoverflow.com/questions/2766844/pointcut-matching-methods-with-annotated-parameters/2766981#2766981).

A pointcut with a runtime condition and a required public static method:
```java
@Pointcut("execution(* *.actionPerformed(java.awt.event.ActionEvent)) " +
		"&& args(actionEvent) && if()")
public static boolean button1Pointcut(ActionEvent actionEvent) {
	return (actionEvent.getSource() == j1);
}
```

More information can be found in this  [post](http://http//stackoverflow.com/questions/2939879/how-to-capture-button-click-if-more-than-one-button-in-aspectj/2940141#2940141)

## Combining pointcuts

Instead of having a large pointcut, it’s a much better approach to combine several pointcuts into one.

`@Pointcut``(``"traceMethodsInDemoPackage() && notInTestClass() && notSetMethodsInTraceDemoPackage()"``)`

`public` `void` `filteredTraceMethodsInDemoPackage() {}`

## Advices

The definition from the AspectJ  [homepage](http://eclipse.org/aspectj/doc/released/progguide/language-anatomy.html#advice):

> A piece of advice brings together a pointcut and a body of code to define aspect implementation that runs at join points picked out by the pointcut.

An advice can be executed before, after, after returning, after throwing or around the joinpoint.

The before advice below is executed before the target method specified in filteredTraceMethodsInDemoPackage pointcut:
```java
@Before("filteredTraceMethodsInDemoPackage()")
public void beforeTraceMethods(JoinPoint joinPoint) {
	// trace logic ..
}
```

This after advice is executed after the target method.
```java
@After("filteredTraceMethodsInDemoPackage()")
public void afterTraceMethods(JoinPoint joinPoint) {
	// trace logic ..
}
```

The afterThrowing advice will be executed if the method that matches the pointcut throws an exception. You can also declare the Exception and handle it like this:
```java
@AfterThrowing(value="serviceMethodAfterExpcetionFromIntegrationLayerPointcut()", throwing="e")
public void serviceMethodAfterExceptionFromIntegrationLayer(JoinPoint joinPoint,
				RuntimeException e) {
	StringBuilder arguments = generateArgumentsString(joinPoint.getArgs());
	logger.error("Error in service " + joinPoint.getSignature() + " with the arguments: " +
			arguments, e);
}
```

The AfterReturning advice will only be executed if the adviced method returns successfully.
```java
@AfterReturning("transactionalMethod()")
public void afterTransactionalMethod(JoinPoint joinPoint) {
	transactionService.commit();
}
```

The around advice is quite powerful, but also consumes more resources and should only be used if you can’t make it work with other advices. The example below simulates some logic before and after the adviced method. The ProceedingJoinPoint extends the JoinPoint class and is required to call the adviced method with the proceed() method.
```java
@Around("performanceLogablePointcut()")
public void aroundPerformanceLogableMethod(ProceedingJoinPoint point) {
	// Pre processing work..
	point.proceed();
	// Post processing work..
}
```

Note: You don’t need an around advice to monitor the execution time for a method. It’s just the simplest option and therefore used in this example.

The advice below combines two pointcuts and the last one has an integer object as input parameter. This requires the Java method to also have an integer parameter. It enables the advice logic to access the input parameter directly and in a type safe manner. A less elegant alternative is to access the parameter with the joinPoint’s getArgs() method.
```java
@AfterReturning("beanAnnotatedWithComponentOrASpecializationOfIt() && " +
	"constructorAnnotatedWithInjectAndIndemoPackage(integer)")
public void afterReturningFromConstructorInSpringBeanWithIntegerParameter(
		JoinPoint joinPoint, Integer integer) {
	// Advice logic..
}
```

## Summary

This cheat sheet consist of many simple pointcuts. Often you have to combine them to add value to your application. This  [example article](https://blog.espenberntsen.net/2010/03/18/aspectj-examples-with-pointcuts-based-on-annotations/)  combines pointcuts with a class marked with @Service and a constructor and method marked with @Inject. It also filters away the test method with the @Test annotation.

The code used in this article was developed in my work time at Redpill Linpro.