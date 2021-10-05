https://stackoverflow.com/questions/18510395/does-spring-publish-beans-in-thread-safe-manner

As Evgeniy has stated, the initialization of an application context happens in a single thread.  Therefore the answer to your question isn't so much to do with the internals of Spring but rather the synchronization details between the thread that creates the context and the thread or threads that use the context once it has been created.

The Java memory model is based on the [_happens-before_ relation][1] (Java Language Specification, §17.4.5) which is defined by various rules.  For example, the call to `Thread.start` made in one thread to start a new thread _happens-before_ all actions in the newly started thread itself.  So if your application's main thread first creates an application context, and then starts other threads for processing, then the processing threads are guaranteed to see the fully-initialized context.

Fields marked `volatile` also impose a _happens-before_ relation, in the sense that if thread A writes a value to a `volatile`, any other thread that sees the result of that write is also guaranteed to see anything else that thread A did _before_ it did the volatile write.  Therefore if there isn't any explicit synchronization between the initialization thread and the processing threads then the following pattern would be sufficient to ensure safety

    public class Setup {
      private volatile boolean inited = false;

      private ApplicationContext ctx;

      public boolean isInited() { return inited; }

      public ApplicationContext getContext() { return ctx; }

      public void init() {
        ctx = new ClassPathXmlApplicationContext("context.xml");
        inited = true; // volatile write
      }
    }

    public class Processor {
      private void ensureInit() {
        while(!setup.isInited()) { // volatile read
          Thread.sleep(1000);
        }
      }

      public void doStuff() {
        ensureInit();
        // at this point we know the context is fully initialized
      }
    }


  [1]: http://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.5

---

https://stackoverflow.com/questions/16248898/memory-consistency-happens-before-relationship-in-java

Modern CPUs don't always write data to memory in the order it was updated, for example if you run the pseudo code (assuming variables are always stored to memory here for simplicity);

```java
a = 1
b = a + 1

```

...the CPU may very well write  `b`  to memory before it writes  `a`  to memory. This isn't really a problem as long as you run things in a single thread, since the thread running the code above will never see the old value of either variable once the assignments have been made.

Multi threading is another matter, you'd think the following code would let another thread pick up the value of your heavy computation;

```java
a = heavy_computation()
b = DONE

```

...the other thread doing...

```java
repeat while b != DONE
    nothing

result = a

```

The problem though is that the done flag may be set in memory before the result is stored to memory, so  _the other thread may pick up the value of memory address a before the computation result is written to memory._

The same problem would -  _if  `Thread.start`  and  `Thread.join`  didn't have a "happens before" guarantee_  - give you problems with code like;

```java
a = 1
Thread.start newthread
...

newthread:
    do_computation(a)

```

...since  `a`  may not have a value stored to memory when the thread starts.

Since you almost always want the new thread to be able to use data you initialized before starting it,  `Thread.start`  has a "happens before" guarantee, that is,  _data that has been updated before calling  `Thread.start`  is guaranteed to be available to the new thread_. The same thing goes for  `Thread.join`  where  _data written by the new thread is guaranteed to be visible to the thread that joins it after termination_.

It just makes threading much easier.