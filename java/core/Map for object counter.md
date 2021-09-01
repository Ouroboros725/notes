https://stackoverflow.com/questions/81346/most-efficient-way-to-increment-a-map-value-in-java

Now there is a shorter way with Java 8 using [`Map::merge`][1].

    myMap.merge(key, 1, Integer::sum)

What it does:

- if *key* do not exists, put *1* as value
- otherwise *sum 1* to the value linked to *key*

More information [here][2].


[1]: https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#merge(K,V,java.util.function.BiFunction)
[2]: https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#merge-K-V-java.util.function.BiFunction-

---

## Some test results

I've gotten a lot of good answers to this question--thanks folks--so I decided to run some tests and figure out which method is actually fastest. The five methods I tested are these:

* the "ContainsKey" method that I presented in [the question](https://stackoverflow.com/questions/81346/most-efficient-way-to-increment-a-map-value-in-java)
* the "TestForNull" method suggested by [Aleksandar Dimitrov](#81765)
* the "AtomicLong" method suggested by [Hank Gay](#81522)
* the "Trove" method suggested by [jrudolph](#81399)
* the "MutableInt" method suggested by [phax.myopenid.com](#81550)

## Method

Here's what I did...

1. created five classes that were identical except for the differences shown below. Each class had to perform an operation typical of the scenario I presented: opening a 10MB file and reading it in, then performing a frequency count of all the word tokens in the file. Since this took an average of only 3 seconds, I had it perform the frequency count (not the I/O) 10 times.
2. timed the loop of 10 iterations but _not the I/O operation_ and recorded the total time taken (in clock seconds) essentially using [Ian Darwin's method in the Java Cookbook](http://books.google.com/books?id=t85jM-ZwTX0C&printsec=frontcover&dq=java+cookbook&sig=ACfU3U1lAe1vnbVUwdIcWeTpaxZi1xVUXQ#PPA734,M1).
3. performed all five tests in series, and then did this another three times.
4. averaged the four results for each method.

## Results

I'll present the results first and the code below for those who are interested.

The **ContainsKey** method was, as expected, the slowest, so I'll give the speed of each method in comparison to the speed of that method.

* **ContainsKey:** 30.654 seconds (baseline)
* **AtomicLong:** 29.780 seconds (1.03 times as fast)
* **TestForNull:** 28.804 seconds (1.06 times as fast)
* **Trove:** 26.313 seconds (1.16 times as fast)
* **MutableInt:** 25.747 seconds (1.19 times as fast)

## Conclusions

It would appear that only the MutableInt method and the Trove method are significantly faster, in that only they give a performance boost of more than 10%. However, if threading is an issue, AtomicLong might be more attractive than the others (I'm not really sure). I also ran TestForNull with <code>final</code> variables, but the difference was negligible.

Note that I haven't profiled memory usage in the different scenarios. I'd be happy to hear from anybody who has good insights into how the MutableInt and Trove methods would be likely to affect memory usage.

Personally, I find the MutableInt method the most attractive, since it doesn't require loading any third-party classes. So unless I discover problems with it, that's the way I'm most likely to go.

## The code

Here is the crucial code from each method.

### ContainsKey

    import java.util.HashMap;
    import java.util.Map;
    ...
    Map<String, Integer> freq = new HashMap<String, Integer>();
    ...
    int count = freq.containsKey(word) ? freq.get(word) : 0;
    freq.put(word, count + 1);

### TestForNull

    import java.util.HashMap;
    import java.util.Map;
    ...
    Map<String, Integer> freq = new HashMap<String, Integer>();
    ...
    Integer count = freq.get(word);
    if (count == null) {
        freq.put(word, 1);
    }
    else {
        freq.put(word, count + 1);
    }

### AtomicLong

    import java.util.concurrent.ConcurrentHashMap;
    import java.util.concurrent.ConcurrentMap;
    import java.util.concurrent.atomic.AtomicLong;
    ...
    final ConcurrentMap<String, AtomicLong> map = 
        new ConcurrentHashMap<String, AtomicLong>();
    ...
    map.putIfAbsent(word, new AtomicLong(0));
    map.get(word).incrementAndGet();

### Trove

    import gnu.trove.TObjectIntHashMap;
    ...
    TObjectIntHashMap<String> freq = new TObjectIntHashMap<String>();
    ...
    freq.adjustOrPutValue(word, 1, 1);

### MutableInt

    import java.util.HashMap;
    import java.util.Map;
    ...
    class MutableInt {
      int value = 1; // note that we start at 1 since we're counting
      public void increment () { ++value;      }
      public int  get ()       { return value; }
    }
    ...
    Map<String, MutableInt> freq = new HashMap<String, MutableInt>();
    ...
    MutableInt count = freq.get(word);
    if (count == null) {
        freq.put(word, new MutableInt());
    }
    else {
        count.increment();
    }

