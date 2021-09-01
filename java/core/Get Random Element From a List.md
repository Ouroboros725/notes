https://www.baeldung.com/java-random-list-element

## **1. Introduction**[](https://www.baeldung.com/java-random-list-element#Introduction)

Picking a random  _List_  element is a very basic operation but not so obvious to implement. In this article, we'll show the most efficient way of doing this in different contexts.

## **2. Picking a Random Item/Items**[](https://www.baeldung.com/java-random-list-element#Random)

In order to get a random item from a  _List_  instance, you need to generate a random index number and then fetch an item by this generated index number using  _List.get()_  method.

The key point here is to remember that you mustn't use an index that exceeds your  _List's_  size.

### **2.1. Single Random Item**[](https://www.baeldung.com/java-random-list-element#1-single-random-item)

In order to select a random index, you can use  _Random.nextInt(int bound)_ method:

```java
public void givenList_shouldReturnARandomElement() {
    List<Integer> givenList = Arrays.asList(1, 2, 3);
    Random rand = new Random();
    int randomElement = givenList.get(rand.nextInt(givenList.size()));
}
```

Instead of  _Random_ class, you can always use static method  _Math.random()_ and multiply it with list size (_Math.random()_  generates  _Double_  random value between 0 (inclusive) and 1 (exclusive), so remember to cast it to  _int_  after multiplication).

### **2.2. Select Random Index in Multithread Environment**[](https://www.baeldung.com/java-random-list-element#2-select-random-index-in-multithread-environment)

When writing multithread applications using the single  _Random_  class instance, might result in picking same value for every process accessing this instance. We can always create a new instance per thread by using a dedicated  _ThreadLocalRandom_ class:

```java
int randomElementIndex
  = ThreadLocalRandom.current().nextInt(listSize) % givenList.size();
```

### **2.3. Select Random Items With Repetitions**[](https://www.baeldung.com/java-random-list-element#3-select-random-items-with-repetitions)

Sometimes you might want to pick few elements from a list. It is quite straightforward:

```java
public void givenList_whenNumberElementsChosen_shouldReturnRandomElementsRepeat() {
    Random rand = new Random();
    List<String> givenList = Arrays.asList("one", "two", "three", "four");

    int numberOfElements = 2;

    for (int i = 0; i < numberOfElements; i++) {
        int randomIndex = rand.nextInt(givenList.size());
        String randomElement = givenList.get(randomIndex);
    }
}
```

### **2.4. Select Random Items Without Repetitions**[](https://www.baeldung.com/java-random-list-element#4-select-random-items-without-repetitions)

Here, you need to make sure that element is removed from the list after selection:

```java
public void givenList_whenNumberElementsChosen_shouldReturnRandomElementsNoRepeat() {
    Random rand = new Random();
    List<String> givenList = Lists.newArrayList("one", "two", "three", "four");

    int numberOfElements = 2;

    for (int i = 0; i < numberOfElements; i++) {
        int randomIndex = rand.nextInt(givenList.size());
        String randomElement = givenList.get(randomIndex);
        givenList.remove(randomIndex);
    }
}
```

### **2.5. Select Random Series**[](https://www.baeldung.com/java-random-list-element#5-select-random-series)

In case you would like to obtain random series of elements,  _Collections_  utils class might be handy:

```java
public void givenList_whenSeriesLengthChosen_shouldReturnRandomSeries() {
    List<Integer> givenList = Lists.newArrayList(1, 2, 3, 4, 5, 6);
    Collections.shuffle(givenList);

    int randomSeriesLength = 3;

    List<Integer> randomSeries = givenList.subList(0, randomSeriesLength);
}
```

## **3. Conclusion**[](https://www.baeldung.com/java-random-list-element#Conclusion)

In this article, we explored the most efficient way of fetching random elements from a  _List_  instance for different scenarios.

Code examples can be found on  [GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list).