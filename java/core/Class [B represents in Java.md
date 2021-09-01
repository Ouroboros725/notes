https://stackoverflow.com/questions/1466508/what-does-the-class-class-b-represents-in-java

Those are arrays of primitives (`[B == byte[]`, `[C == char`, `[I == int`). `[Lx;` is an array of class type `x`.

For a full list:

    [Z = boolean
    [B = byte
    [S = short
    [I = int
    [J = long
    [F = float
    [D = double
    [C = char
    [L = any non-primitives(Object)

Also see the Javadoc for [`Class.getName`][1].


[1]: https://docs.oracle.com/javase/9/docs/api/java/lang/Class.html#getName--