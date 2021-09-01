https://gist.github.com/reemsky/f4fa84d24d2b6615a16d

first create this class:

```java
package example;
import org.springframework.context.ApplicationContext;

import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class ApplicationContextProvider implements ApplicationContextAware {
    private static ApplicationContext context;
 
    public static ApplicationContext getApplicationContext() {
        return context;
    }
 
    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        context = ctx;
    }
}
```

In any object that is not Spring managed:
```java
DomainRepository repo = ApplicationContextProvider.getApplicationContext().getBean(DomainRepository.class);
```