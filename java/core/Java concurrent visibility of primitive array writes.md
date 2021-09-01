https://stackoverflow.com/questions/14714847/java-concurrent-visibility-of-primitive-array-writes

In terms of visibility, all you need is volatile write-read, on any volatile field. This would work


    final    float[][] matrix  = ...;
    volatile float[][] matrixV = matrix;

Thread 1

    matrix[x][y] = n;
    matrixV = matrix; // volatile write

Thread 2

    float[][] m = matrixV;  // volatile read
    myVar = m[x][y];

    or simply
    myVar = matrixV[x][y];

But this is only good for updating one variable. If writer threads are writing multiple variables and the read thread is reading them, the reader may see an inconsistent picture. Usually it's dealt with by a read-write lock. Copy-on-write might be suitable for some use patterns.

Doug Lea has a new "StampedLock" http://gee.cs.oswego.edu/dl/jsr166/dist/jsr166edocs/jsr166e/StampedLock.html for Java8, which is a version of read-write lock that's much cheaper for read locks. But it is much harder to use too. Basically the reader gets the current version, then go ahead and read a bunch of variables, then check the version again; if the version hasn't changed, there was no concurrent writes during the read session.