## Instance

If you want to inject Daos generically at runtime, i would recommend this approach.

```
@Qualifier
@Retention(RUNTIME)
@Target({TYPE,METHOD,FIELD,PARAMETER})
public @interface DAO{

  String value();
}

//Dont worry, CDI allows this quite often than not ;)
public class DAOImpl extends AnnotationLiteral<DAO> implements DAO {

   private final String name;
   public DAOImpl(final String name) {
     this.name = name;
   }

   @Override
   public String value() {
     return name;
   }
}
```

Where it is required.

```
@ApplicationScoped; //Or Whatever
public class MyDAOConsumer {

   @Inject
   @Any
   private Instance<DAOService> daoServices;

   //Just as an example where you can get the dynamic configurations for selecting daos. 
   //Even from property files, or system property.
   @Inject
   private MyDynamicConfigurationService myDanamicConfigurationService;

   public void doSomethingAtRuntime() {
     final String daoToUse = myDanamicConfigurationService.getCurrentConfiguredDaoName();
     final DAO dao = new DAOImpl(daoToUse);

     //Careful here if the DaoService does not exist, you will get UnsatisfiedException
     final DAOService daoService = daoServices.select(dao).get();
     daoService.service();
   }
}
```

And bumped, you can configure which dao to use at runtime without a sweat. And without changing any tiny bit of code.

## Producer
I would forget about the custom annotations and use a producer method. In that method you could lookup the class you want to instantiate from an xml file or config file or whatever way you want to do it.

Take a look at [Producer Methods](https://docs.oracle.com/javaee/7/tutorial/cdi-adv003.htm#GKGKV) in the Java EE tutorial.

https://stackoverflow.com/questions/29745842/dynamic-cdi-injection-at-runtime