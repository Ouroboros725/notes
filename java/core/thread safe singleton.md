https://stackoverflow.com/questions/16106260/thread-safe-singleton-class

You are implementing the [*lazy initialization*][1] pattern - where the instance is created when first used.

But there is a simple trick that allows you to code a threadsafe implementation that *doesn't* require synchronization! It is known as the [*Initialization-on-demand holder idiom*][2], and it looks like this:

    public class CassandraAstyanaxConnection {

        private CassandraAstyanaxConnection(){ }

        private static class Holder {
           private static final CassandraAstyanaxConnection INSTANCE = new CassandraAstyanaxConnection();
        }

        public static CassandraAstyanaxConnection getInstance() {
            return Holder.INSTANCE;
        }
        // rest of class omitted
    }

This code initializes the instance on the first calling of `getInstance()`, and importantly doesn't need synchronization because of the contract of the class loader:

- the class loader loads classes when they are first accessed (in this case `Holder`'s only access is within the `getInstance()` method)
- when a class is loaded, and before anyone can use it, all static initializers are guaranteed to be executed (that's when `Holder`'s static block fires)
- the class loader has its own synchronization built right in that make the above two points guaranteed to be threadsafe

It's a neat little trick that I use whenever I need lazy initialization. You also get the bonus of a `final` instance, even though it's created lazily. Also note how clean and simple the code is.

**Edit:** You should set all constructors as private or protected. Setting and empty private constructor will do the work

[1]: http://en.wikipedia.org/wiki/Lazy_initialization
[2]: http://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom