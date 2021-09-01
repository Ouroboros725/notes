`Provider<T>` is a JSR-330 interface which is extended by the CDI interface `Instance<T>`.

Injecting `MyBean`, your application will throw an exception during startup when there is no matching bean or more than one matching bean.

Injecting `Instance<MyBean>`, bean resolution is delegated to the application: you can iterate over all candidate beans and `select()` the one you want or call `isUnsatisfied()` and decide what to do when there is no matching bean.

For beans with `@Dependent` scope, calling `Instance.get()` will create a new instance for each invocation, and you should invoke `Instance.destroy(t)` for each such instance when you no longer need it.

`Provider` just has the `get()` method, but no `destroy()` or `select()` and does not support iteration. In a CDI environment, for any use case addressed by `Provider<T>`, you had better use `Instance<T>` instead.

https://stackoverflow.com/questions/33127576/when-should-instancet-and-providert-be-used-to-inject-beans-in-cdi