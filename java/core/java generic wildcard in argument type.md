https://stackoverflow.com/questions/12733709/java-generics-cannot-be-applied-to-given-type

I have some code similar to this:

    public class Main {

        private static abstract class Bar {}

        private static class SubBar extends Bar {}

        private static abstract class Baz<T extends Bar> {
 
            private T t;

            public void setT(T t) {
                this.t = t;
            }

        }

        private static class SubBaz extends Baz<SubBar> {}


        private void foo(Baz<? extends Bar> baz, Bar bar) {
            baz.setT(bar);
        }
     }

That results in error:

    error: method setT in class Baz<T> cannot be applied to given types;
    required: CAP#1
    found: Bar
    reason: actual argument Bar cannot be converted to CAP#1 by method invocation conversion
    where T is a type-variable:
    T extends Bar declared in class Baz
    where CAP#1 is a fresh type-variable:
    CAP#1 extends Bar from capture of ? extends Bar

I don't understand why. The method setT should accept something that extends Bar and I am passing something of class Bar.

---

>  The method setT should accept something that extends Bar and I am passing something of class Bar.

That's exactly the problem: `<? extends Bar>` means "Some unknown type that is `Bar` or a subclass of it". Since you don't know which type it is, it's actually impossible to call `setT()` in that context, except with a `null` parameter.

This will work as expected:

    private void foo(Baz<Bar> baz, Bar bar) {
        baz.setT(bar);
    }

There are, I am sure, hundreds of variations of this questions on Stackoverflow. It seems almost every programmer at first misunderstands what the `?` wildcard is for and uses it wrongly.

Its utility is in the situation where your `Bar` class has a `public T getT()` method. A variable of type `Baz<? extends Bar>` could hold objects of both `Baz<Bar>` and `Baz<SubBar>`, and you could call `getT()` on it to get something that is a `Bar` (or some subclass) and can be used like that.