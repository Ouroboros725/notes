https://stackoverflow.com/questions/9809074/java-difference-between-strong-soft-weak-phantom-reference

Java provides two different types/classes of *Reference Objects*: **strong** and **weak**. Weak Reference Objects can be further divided into *soft* and *phantom*.

- Strong
- Weak
- soft
- phantom

Let's go point by point.

**Strong Reference Object**

    StringBuilder builder = new StringBuilder();

This is the default type/class of Reference Object, if not differently specified: `builder` is a strong Reference Object. This kind of reference makes the referenced object not eligible for GC. That is, whenever an object is referenced by a *chain of strong Reference Objects*, it cannot be garbage collected.

**Weak Reference Object**

    WeakReference<StringBuilder> weakBuilder = new WeakReference<StringBuilder>(builder);

Weak Reference Objects are not the default type/class of Reference Object and to be used they should be explicitly specified like in the above example. This kind of reference makes the reference object eligible for GC.  That is, in case the only reference reachable for the `StringBuilder` object in memory is, actually, the weak reference, then the GC is allowed to garbage collect the `StringBuilder` object. When an object in memory is reachable only by Weak Reference Objects, it becomes automatically eligible for GC.

**Levels of Weakness**

Two different levels of weakness can be enlisted: *soft* and *phantom*.

A *soft* Reference Object is basically a weak Reference Object that remains in memory a bit more: normally, it resists GC cycle until no memory is available and there is risk of `OutOfMemoryError` (in that case, it can be removed).

On the other hand, a *phantom* Reference Object is useful only to know exactly when an object has been effectively removed from memory: normally they are used to fix *weird finalize() revival/resurrection behavior*, since they actually do not return the object itself but only help [in keeping track of their memory presence][1].

Weak Reference Objects are ideal to implement cache modules. In fact, a sort of automatic eviction can be implemented by allowing the GC to clean up memory areas whenever objects/values are no longer reachable by strong references chain. An example is the [WeakHashMap][2] retaining weak keys.

[1]: https://community.oracle.com/blogs/enicholas/2006/05/04/understanding-weak-references
[2]: http://docs.oracle.com/javase/7/docs/api/java/util/WeakHashMap.html