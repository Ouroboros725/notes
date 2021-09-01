https://stackoverflow.com/questions/15093894/aspectj-pointcut-for-annotated-private-methods

[8. Aspect Oriented Programming with Spring](http://static.springsource.org/spring/docs/3.1.x/spring-framework-reference/html/aop.html#aop-introduction-spring-defn)

> Due to the proxy-based nature of Spring's AOP framework, protected methods are by definition not intercepted, neither for JDK proxies (where this isn't applicable) nor for CGLIB proxies (where this is technically possible but not recommendable for AOP purposes). As a consequence, any given pointcut will be matched against public methods only!
>
> If your interception needs include protected/private methods or even constructors, consider the use of Spring-driven native AspectJ weaving instead of Spring's proxy-based AOP framework. This constitutes a different mode of AOP usage with different characteristics, so be sure to make yourself familiar with weaving first before making a decision.