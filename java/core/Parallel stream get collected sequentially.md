https://stackoverflow.com/questions/29709140/why-parallel-stream-get-collected-sequentially-in-java-8

There are two different kinds of "ordering" going on here, which makes the discussion confusing.

One kind is *encounter order*, which is defined in the [streams documentation][1]. A good way to think about this is the *spatial* or *left-to-right* order of elements in the source collection. If the source is a `List`, consider the earlier elements being to the left of later elements.

There is also *processing* or *temporal* order, which isn't defined in the documentation, but which is the time order in which elements are processed by different threads. If the elements of a list are being processed in parallel by different threads, a thread might process the rightmost element in the list before the leftmost element. But the next time it might not.

Even when computations are done in parallel, most `Collectors` and some terminal operations are carefully arranged so that they preserve *encounter order* from the source through to the destination, independently of the *temporal order* in which different threads might process each element.

Note that the `forEach` terminal operation does **not** preserve encounter order. Instead, it's run by whatever thread happens to produce the next result. If you want something like `forEach` that preserves encounter order, use `forEachOrdered` instead.

See also the [Lambda FAQ][2] for further discussion about ordering issues.

[1]: http://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#Ordering
[2]: http://www.lambdafaq.org/in-what-order-do-the-elements-of-a-stream-become-available/