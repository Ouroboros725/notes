https://www.quora.com/Does-JVM-use-multiple-cores

Most JVMs, including OpenJDK and Oracle's Java runtime (JDK / JRE), use OS threads, which means that if there are multiple cores, they will almost certainly all be used by multi-threaded code running on the JVM. Furthermore, the JVM itself uses multiple threads for things like Garbage Collection.

_For the sake of full disclosure, I work at Oracle. The opinions and views expressed in this post are my own, and do not necessarily reflect the opinions or views of my employer._

---

Yup. The standard HotSpot JVM runs parallel Java threads in parallel, runs multiple threads for Garbage Collection and also the JIT compiler, and runs a number of other java-level but not user threads for a bunch of housekeeping tasks. The JVM is well tested and proven on hundreds of cores with thousands of runnable threads.

The JVM can be “pinched” down to a specific number of cores by any OS which can limit CPUs (e.g. linux cgroups), as long as the OS also multi-tasks the runnable threads on the available cores.

The Azul/Zing JVM runs well on ~1000 cores and ~100K runnable threads (so 10x more cores and 100x more runnable threads).