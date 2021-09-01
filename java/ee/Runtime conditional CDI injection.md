For runtime decision (without changing your beans.xml), you basically have two options:

Option 1: use a producer method

```
@Produces
public MyInterface getImplementation() {
   if(runtimeTestPointsTo1)  return new Impl1();
   else                      return new Impl2();
}
```

Drawback: you leave the world of bean creation by using new, therefore your Impl1 and Impl2 cannot @Inject dependencies. (Or you inject both variants in the producer bean and return one of them - but this means both types will be initialized.)

Option 2: use a CDI-extension (portable extention)

Basically listen to processAnotated() and veto everything you don't want. Excellent blog-entry here: http://nightspawn.com/rants/cdi-alternatives-at-runtime/

https://stackoverflow.com/questions/43361113/equivalent-for-conditional-in-cdi