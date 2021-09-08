https://stackoverflow.com/questions/2625546/is-using-the-class-instance-as-a-map-key-a-best-practice

Yes, you do need to be cautious!  For example, if your code is running in a web container and you are in the habit of doing hot deployment of webapps, a retained reference to a single class object can cause a significant permgen memory leak.

[This article][1] explains the problem in detail.  But in a nutshell, the problem is that each class contains a reference to its classloader, and each classloader contains references to every class that it has loaded.  So if one class is reachable, all of them are.

The other thing to note is that if one of the classes that you are using as a key is reloaded then:

1.  The old and new versions of the class will not be equal.
2.  Looking up the new class will initially give a "miss".
3.  After you have added the new class to the map, you will now have two different map entries for the different versions of the class.
4.  This applies even if there is no *code* difference between the two versions of the class.  They will be different simply because they were loaded by different classloaders.

-----------------

> From Java 8 - Permgen was removed. Do you think it is ok to use Class instance as HashMap key in any situations?

Be aware that you will still have a memory leak.  Any dynamicly loaded class used in your HashMap (key or value) and (at least) other dynamically loaded classes will be kept reachable.  This means the GC won't be able to unload / delete them.

What was previously a permgen leak is now a ordinary heap and  metaspace storage leak.  (Metaspace is where the class descriptors and code objects for the classes are kept.)

[1]: http://frankkieviet.blogspot.com/2006/10/classloader-leaks-dreaded-permgen-space.html