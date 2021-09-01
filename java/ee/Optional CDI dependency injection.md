There are many questions lurking in this seemingly simple question. I'll try to answer them bearing in mind the spirit of the question.

First, as a general rule, if you `@Inject` a `Fred`, that `Fred` cannot be `null` unless `Fred` is in `@Dependent` scope, and even then a producer method or custom bean will have to explicitly be written to return `null`. There are edge cases but in all modern CDI implementations this is a good rule of thumb to bear in mind.

Second, `Optional` isn't special. From the standpoint of CDI, an `Optional` is just another Java object, so see my first statement above. If you have something that produces an `Optional` (like a producer method) then it cannot make a `null` `Optional` (unless, again, the production is defined to be in the `@Dependent` scope—and if you were writing such a method to make `Optional` instances and returning `null` you are definitely going to confuse your users). If you are in control of producing `Optional` instances, then you can make them any way you like.

Third, in case you want to test to see if there is a managed bean or a producer of some kind for a `Fred`, you can, as one of the comments on your question indicates, inject a `Provider<Fred>` or an `Instance<Fred>`. These are "made" by the container automatically: you don't have to write anything special to produce them yourself. A `Provider<Fred>` is an accessor of `Fred` instances and does not attempt to acquire an instance until its `get()` method is called. An `Instance` is a `Provider` and an `Iterable` of all known `Fred`s and can additionally tell you whether (a) it is "unsatisfied"—there are no producers of `Fred` at all—and (b) it is "resolvable"—i.e. there is exactly one producer of `Fred`.

Fourth, the common idiom in cases where you want to see if something is there is to inject an `Instance` parameterized with the type you want, and then check its `isResolvable()` method. If that returns `true`, then you can call its `get()` method and trust that its return value will be non-`null` (assuming the thing it makes is not in `@Dependent` scope).

https://stackoverflow.com/questions/59692479/can-cdi-dependency-injection-be-optional