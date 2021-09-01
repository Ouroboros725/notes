https://stackoverflow.com/questions/29928569/how-to-advice-entity-classes-not-spring-beans

> only spring beans could be advised

Well, it's true in case you are using Spring AOP and not(!!!) AspectJ.

If replacing Spring AOP with AspectJ is an option, you can weave what ever you like by using  `@Configurable`

[Here](http://docs.spring.io/spring/docs/4.1.6.RELEASE/spring-framework-reference/html/aop.html#aop-ajlib-other)  You can find the documentation that says that you can put Spring annotations like  `@Transactional`  on your non beans instances.