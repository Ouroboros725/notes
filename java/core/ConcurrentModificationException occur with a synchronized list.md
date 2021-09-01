https://stackoverflow.com/questions/23427387/java-why-concurrentmodificationexception-occur-with-a-synchronized-list

Even most synchronized collections do not like modifications and iterator together. From the API description of Collections.synchronizedList:

> It is imperative that the user manually synchronize on the returned
> list when iterating over it:
>
>   `List list = Collections.synchronizedList(new ArrayList());
>       ...   synchronized (list) {
>       Iterator i = list.iterator(); // Must be in synchronized block
>       while (i.hasNext())
>           foo(i.next());   }`

Also: You can use the collections from java.concurrent instead. They usually have a more refined synchronization approach.

