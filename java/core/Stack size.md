https://stackoverflow.com/questions/10136281/update-a-java-threads-stack-size-at-runtime

The stack size dynamcally updates itself as it is used so you never need to so this.

What you can set is the maximum size it can be with `-Xss` This is the virtual memory size used and you can make it as large as you like on 64-bit JVMs. The actual memory used is based on the amount of memory you use. ;)

EDIT: The important distinction is that the maximum size is reserved as virtual memory (so is the heap btw). i.e. the address space is reserved, which is also why it cannot be extended.  In 32-bit systems you have limited address space and this can still be a problem. But in 64-bit systems, you usually have up to 256 TB of virtual memory (a processor limitation) so virtual memory is cheap.  The actual memory is allocated in pages (typically 4 KB) and they are only allocated when used.  This is why the memory of a Java application appears to grow over time even though the maximum heap size is allocated on startup. The same thing happens with thread stacks. Only the pages actually touched are allocated.

---

There's not a way to do this in the standard JDK, and even the `stackSize` argument isn't set in stone:

>**The effect of the stackSize parameter, if any, is highly platform dependent.** ... **On some platforms, the value of the stackSize parameter may have no effect whatsoever.** ... The virtual machine is free to treat the stackSize parameter as a suggestion.

(Emphasis in [original][1].)


[1]: http://docs.oracle.com/javase/7/docs/api/java/lang/Thread.html#Thread%28java.lang.ThreadGroup,%20java.lang.Runnable,%20java.lang.String,%20long%29

---

https://stackoverflow.com/questions/20030120/what-is-the-default-stack-size-can-it-grow-how-does-it-work-with-garbage-colle

## How much a stack can grow?

You can use a VM option named `ss` to adjust the maximum stack size. A VM option is usually passed using -X{option}. So you can use `java -Xss1M` to set the maximum of stack size to 1M.

Each thread has at least one stack. Some Java Virtual Machines (JVM) put Java stack (Java method calls) and native stack (Native method calls in VM) into one stack, and perform stack unwinding using a "Managed to Native Frame", known as [M2nFrame][1]. Some JVMs keep two stacks separately. The `Xss` set the size of the Java Stack in most cases.

For many JVMs, they put different default values for stack size on different platforms.

---
## Can we limit this growth?

When a method call occurs, a new stack frame will be created on the stack of that thread. The stack will contain local variables, parameters, return address, etc. In Java, you can never put an object on stack, only object reference can be stored on stack. Since array is also an object in Java, arrays are also not stored on stack. So, if you reduce the amount of your local primitive variables, parameters by grouping them into objects, you can reduce the space on stack. Actually, the fact that we cannot explicitly put objects on Java stack affects the performance some time (cache miss).

---
## Does stack has some default minimum value or default maximum value?

As I said before, different VMs are different, and may change over versions. See [here][2].

---
## How does garbage collection work on stack?

Garbage collections in Java is a hot topic. Garbage collection aims to collect unreachable objects in the **heap**. So that needs a definition of 'reachable.' Everything on the stack constitutes part of the root set references in GC. Everything that is reachable from every stack of every thread should be considered as live. There are some other root set references, like Thread objects and some class objects.

This is only a very vague use of stack on GC. Currently most JVMs are using a generational GC. [This article][3] gives brief introduction about Java GC. And recently I read [a very good article][4] talking about the GC on .NET platform. The GC on Oracle JVM is quite similar so I think that might also help you.


[1]: https://harmony.apache.org/subcomponents/drlvm/developers_guide.html#M2nFrame
[2]: http://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/jrdocs/refman/optionX.html#wp1024112
[3]: http://www.cubrid.org/blog/dev-platform/understanding-java-garbage-collection/
[4]: https://docs.microsoft.com/en-us/archive/blogs/abhinaba/back-to-basics-generational-garbage-collection