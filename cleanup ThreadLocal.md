https://stackoverflow.com/questions/13852632/is-it-really-my-job-to-clean-up-threadlocal-resources-when-classes-have-been-exp

# Sigh, this is old news

Well, a bit late to the party on this one.  In October 2007, Josh Bloch (co-author of `java.lang.ThreadLocal` along with Doug Lea) [wrote](http://jsr166-concurrency.10961.n7.nabble.com/Threadlocals-and-memory-leaks-in-J2EE-tp3960p3984.html):

> "The use of thread pools demands extreme care.  Sloppy use of thread
> pools in combination with sloppy use of thread locals can cause
> unintended object retention, as has been noted in many places."

People were complaining about the bad interaction of ThreadLocal with thread pools even then.  But Josh did sanction:

"Per-thread instances for performance.  Aaron's SimpleDateFormat example (above) is one example of this pattern."


# Some Lessons

1. If you put any kind of objects into any object pool, you must provide a way to remove them 'later'.
2. If you 'pool' using a `ThreadLocal`, you have limited options for doing that.  Either:
   a) you *know* that the `Thread`(s) where you put values will terminate when your application is finished; OR
   b) you can later arrange for  *same thread* that invoked ThreadLocal#set() to invoke ThreadLocal#remove() whenever your application terminates
3. As such, your use of ThreadLocal as an object pool is going to exact a heavy price on the design of your application and your class.  The benefits don't come for free.
4. As such, use of ThreadLocal is probably a premature optimization, even though Joshua Bloch urged you to consider it in 'Effective Java'.

In short, deciding to use a ThreadLocal as a form of fast, uncontended access to "per thread instance pools" is not a decision to be taken lightly.

NOTE: There are other uses of ThreadLocal other than 'object pools', and these lessons do not apply to those scenarios where the ThreadLocal is only intended to be set on a temporary basis anyway, or where there is genuine per-thread state to keep track of.


# Consequences for Library implementors

Threre are some consequences for library implementors (even where such libraries are simple utility classes in your project).

Either:

1.  You use ThreadLocal, fully aware that you might 'pollute' long-running threads with extra baggage.  If you are implementing `java.util.concurrent.ThreadLocalRandom`, it might be appropriate.  (Tomcat might still whinge at users of your library, if you aren't implementing in `java.*`).  It's interesting to note the discipline with which `java.*` makes sparing use of the ThreadLocal technique.


OR


2. You use ThreadLocal, and give clients of your class/package:
   a) the opportunity to choose to forego that optimization ("don't use ThreadLocal ... I can't arrange for cleanup"); AND
   b) a way to clean up ThreadLocal resources ("it's OK to use ThreadLocal ... I can arrange for all Threads which used you to invoke `LibClass.releaseThreadLocalsForThread()` when I am finished with them.

Makes your library 'hard to use properly', though.


OR

3. You give your clients the opportunity to supply their own object-pool impelementation (which might use ThreadLocal, or synchronization of some sort).  ("OK, I can give you a `new ExpensiveObjectFactory<T>() { public T get() {...} }` if you think it is really neccesasry".

Not so bad.  If the object are really that important and that expensive to create, explicit pooling is probably worthwhile.


OR

4. You decide it's not worth that much to your app anyway, and find a different way to approach the problem.  Those expensive-to-create, mutable, non-threadsafe objects are causing you pain ... is using them really the best option anyway?



# Alternatives

1. Regular object pooling, with all its contended synchronization.
2. Not pooling objects - just instantiate them in a local scope and discard later.
3. Not pooling threads (unless you can schedule cleanup code when you like) - don't use your stuff in a JaveEE container
4. Thread pools which are smart enough to clean-up ThreadLocals without whinging at you.
5. Thread pools which allocate Threads on a 'per application' basis, and then let them die when the application is stopped.
6. A protocol between thread-pooling containers and applications which allowed registration of a 'application shutdown handler', which the container could schedule to run on Threads which had been used to service the application ... at some point in the future when that Thread was next available.   Eg. `servletContext.addThreadCleanupHandler(new Handler() {@Override cleanup() {...}})`

It'd be nice to see some standardisation around the last 3 items, in future JavaEE specs.


# Bootnote

Actually, instantiation of a `GregorianCalendar` is pretty lightweight.  It's the unavoidable call to `setTime()` which incurs most of the work.  It also doesn't hold any significant state between different points of a thread's exeuction.  Putting a `Calendar` into a `ThreadLocal` is unlikely to give you back more than it costs you ... unless profiling definitely shows a hot spot in `new GregorianCalendar()`.

`new SimpleDateFormat(String)` is expensive by comparison, because it has to parse the format string.  Once parsed, the 'state' of the object is significant to later uses by the same thread.  It's a better fit.  But it might still be 'less expensive' to instantiate a new one, than give your classes extra responsibilities.