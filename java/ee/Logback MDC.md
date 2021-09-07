http://logback.qos.ch/manual/mdc.html

One of the design goals of logback is to audit and debug complex distributed applications. Most real-world distributed systems need to deal with multiple clients simultaneously. In a typical multithreaded implementation of such a system, different threads will handle different clients. A possible but slightly discouraged approach to differentiate the logging output of one client from another consists of instantiating a new and separate logger for each client. This technique promotes the proliferation of loggers and may increase their management overhead.

A lighter technique consists of uniquely stamping each log request servicing a given client. Neil Harrison described this method in the book  _Patterns for Logging Diagnostic Messages_  in Pattern Languages of Program Design 3, edited by R. Martin, D. Riehle, and F. Buschmann (Addison-Wesley, 1997). Logback leverages a variant of this technique included in the SLF4J API: Mapped Diagnostic Contexts (MDC).

To uniquely stamp each request, the user puts contextual information into the  `MDC`, the abbreviation of Mapped Diagnostic Context. The salient parts of the MDC class are shown below. Please refer to the  [MDC javadocs](http://www.slf4j.org/api/org/slf4j/MDC.html)  for a complete list of methods.

```java
package org.slf4j;  
  
public  class MDC {  
	//Put a context value as identified by key  
	//into the current thread's context map.  
	public  static  void put(String key,  String val);
	
	//Get the context identified by the key parameter.  
	public  static  String  get(String key);
	  
	//Remove the context identified by the key parameter.  
	public  static  void remove(String key);  

	//Clear all entries in the MDC.  
	public  static  void clear();
}
```

The  `MDC`  class contains only static methods. It lets the developer place information in a  _diagnostic context_  that can be subsequently retrieved by certain logback components. The  `MDC`  manages contextual information on a  _per thread basis_. Typically, while starting to service a new client request, the developer will insert pertinent contextual information, such as the client id, client's IP address, request parameters etc. into the  `MDC`. Logback components, if appropriately configured, will automatically include this information in each log entry.

Please note that MDC as implemented by logback-classic assumes that values are placed into the MDC with moderate frequency. Also note that a child thread does not automatically inherit a copy of the mapped diagnostic context of its parent.

The next application named  [`SimpleMDC`](http://logback.qos.ch/xref/chapters/mdc/SimpleMDC.html) demonstrates this basic principle.

_Example 7.1: Basic MDC usage ( [logback-examples/src/main/java/chapters/mdc/SimpleMDC.java)](http://logback.qos.ch/xref/chapters/mdc/SimpleMDC.html)_

```java
package chapters.mdc;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

import ch.qos.logback.classic.PatternLayout;  
import ch.qos.logback.core.ConsoleAppender;

public class SimpleMDC {
    static public void main(String[] args) throws Exception {
        // You can put values in the MDC at any time. Before anything else
        // we put the first name
        MDC.put("first", "Dorothy");

        [ SNIP ]

        Logger logger = LoggerFactory.getLogger(SimpleMDC.class);
        // We now put the last name
        MDC.put("last", "Parker");

        // The most beautiful two words in the English language according
        // to Dorothy Parker:
        logger.info("Check enclosed.");
        logger.debug("The most beautiful two words in English.");

        MDC.put("first", "Richard");
        MDC.put("last", "Nixon");
        logger.info("I am not a crook.");
        logger.info("Attributed to the former US president. 17 Nov 1973.");
    }
	[ SNIP ]
}
```

The main method starts by associating the value  _Dorothy_  with the key  _first_  in the  `MDC`. You can place as many value/key associations in the  `MDC`  as you wish. Multiple insertions with the same key will overwrite older values. The code then proceeds to configure logback.

For the sake of conciseness, we have omitted the code that configures logback with the configuration file  [simpleMDC.xml](http://github.com/qos-ch/logback/blob/master/logback-examples/src/main/java/chapters/mdc/simpleMDC.xml). Here is the relevant section from that file.

```xml
<appender  name="CONSOLE"  class="ch.qos.logback.core.ConsoleAppender"> 
	<layout>  
		<Pattern>**%X{first} %X{last}** - %m%n</Pattern>  
	</layout>  
</appender>
```

Note the usage of the  _%X_  specifier within the  `PatternLayout`  conversion pattern. The  _%X_  conversion specifier is employed twice, once for the key named  _first_  and once for the key named  _last_. After obtaining a logger corresponding to  `SimpleMDC.class`, the code associates the value  _Parker_  with the key named  _last_. It then invokes the logger twice with different messages. The code finishes by setting the  `MDC`  to different values and issuing several logging requests. Running SimpleMDC yields:
```
Dorothy Parker - Check enclosed.
Dorothy Parker - The most beautiful two words in English.
Richard Nixon - I am not a crook.
Richard Nixon - Attributed to the former US president. 17 Nov 1973.
```
The  `SimpleMDC`  application illustrates how logback layouts, if configured appropriately, can automatically output  `MDC`  information. Moreover, the information placed into the  `MDC`  can be used by multiple logger invocations.

### [Advanced Use](http://logback.qos.ch/manual/mdc.html#AdvancedUse)

Mapped Diagnostic Contexts shine brightest within client server architectures. Typically, multiple clients will be served by multiple threads on the server. Although the methods in the  `MDC`  class are static, the diagnostic context is managed on a per thread basis, allowing each server thread to bear a distinct  `MDC`  stamp.  `MDC`  operations such as  `put()`  and  `get()`  affect only the  `MDC`  of the  _current_  thread, and the children of the current thread. The  `MDC`  in other threads remain unaffected. Given that  `MDC`  information is managed on a per thread basis, each thread will have its own copy of the  `MDC`. Thus, there is no need for the developer to worry about thread-safety or synchronization when programming with the  `MDC`  because it handles these issues safely and transparently.

The next example is somewhat more advanced. It shows how the  `MDC`  can be used in a client-server setting. The server-side implements the  `NumberCruncher`  interface shown in Example 7.2 below.  `The NumberCruncher`  interface contains a single method named  `factor()`. Using RMI technology, the client invokes the  `factor()`  method of the server application to retrieve the distinct factors of an integer.

_Example 7.2: The service interface ( [logback-examples/src/main/java/chapters/mdc/NumberCruncher.java)](http://logback.qos.ch/xref/chapters/mdc/NumberCruncher.html)_
```java
package chapters.mdc;

import java.rmi.Remote;
import java.rmi.RemoteException;

/**
 * NumberCruncher factors positive integers.
 */
public interface NumberCruncher extends Remote {
    /**
     * Factor a positive integer <code>number</code> and return its
     * <em>distinct</em> factor's as an integer array.
     * */
    int[] factor(int number) throws RemoteException;
}
```
The  `NumberCruncherServer`  application, listed in Example 7.3 below, implements the  `NumberCruncher`  interface. Its main method exports an RMI Registry on the local host that accepts requests on a well-known port.

_Example 7.3: The server side ( [logback-examples/src/main/java/chapters/mdc/NumberCruncherServer.java)](http://logback.qos.ch/xref/chapters/mdc/NumberCruncherServer.html)_
```java
package chapters.mdc;

import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.server.UnicastRemoteObject;
import java.util.Vector;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.classic.joran.JoranConfigurator;
import ch.qos.logback.core.joran.spi.JoranException;

/**
 * A simple NumberCruncher implementation that logs its progress when
 * factoring numbers. The purpose of the whole exercise is to show the
 * use of mapped diagnostic contexts in order to distinguish the log
 * output from different client requests.
 * */
public class NumberCruncherServer extends UnicastRemoteObject implements NumberCruncher {

    private static final long serialVersionUID = 1L;

    static Logger logger = LoggerFactory.getLogger(NumberCruncherServer.class);

    public NumberCruncherServer() throws RemoteException {
    }

    public int[] factor(int number) throws RemoteException {
        // The client's host is an important source of information.
        try {
            MDC.put("client", NumberCruncherServer.getClientHost());
        } catch (java.rmi.server.ServerNotActiveException e) {
            logger.warn("Caught unexpected ServerNotActiveException.", e);
        }

        // The information contained within the request is another source
        // of distinctive information. It might reveal the users name,
        // date of request, request ID etc. In servlet type environments,
        // useful information is contained in the HttpRequest or in the
        // HttpSession.
        MDC.put("number", String.valueOf(number));

        logger.info("Beginning to factor.");

        if (number <= 0) {
            throw new IllegalArgumentException(number + " is not a positive integer.");
        } else if (number == 1) {
            return new int[] { 1 };
        }

        Vector<Integer> factors = new Vector<Integer>();
        int n = number;

        for (int i = 2; (i <= n) && ((i * i) <= number); i++) {
            // It is bad practice to place log requests within tight loops.
            // It is done here to show interleaved log output from
            // different requests.
            logger.debug("Trying " + i + " as a factor.");

            if ((n % i) == 0) {
                logger.info("Found factor " + i);
                factors.addElement(i);

                do {
                    n /= i;
                } while ((n % i) == 0);
            }

            // Placing artificial delays in tight loops will also lead to
            // sub-optimal resuts. :-)
            delay(100);
        }

        if (n != 1) {
            logger.info("Found factor " + n);
            factors.addElement(n);
        }

        int len = factors.size();

        int[] result = new int[len];

        for (int i = 0; i < len; i++) {
            result[i] = ((Integer) factors.elementAt(i)).intValue();
        }

        // clean up
        MDC.remove("client");
        MDC.remove("number");

        return result;
    }

    static void usage(String msg) {
        System.err.println(msg);
        System.err.println("Usage: java chapters.mdc.NumberCruncherServer configFile\n" + "   where configFile is a logback configuration file.");
        System.exit(1);
    }

    public static void delay(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
        }
    }

    public static void main(String[] args) {
        if (args.length != 1) {
            usage("Wrong number of arguments.");
        }

        String configFile = args[0];

        if (configFile.endsWith(".xml")) {
            try {
                LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
                JoranConfigurator configurator = new JoranConfigurator();
                configurator.setContext(lc);
                lc.reset();
                configurator.doConfigure(args[0]);
            } catch (JoranException je) {
                je.printStackTrace();
            }
        }

        NumberCruncherServer ncs;

        try {
            ncs = new NumberCruncherServer();
            logger.info("Creating registry.");

            Registry registry = LocateRegistry.createRegistry(Registry.REGISTRY_PORT);
            registry.rebind("Factor", ncs);
            logger.info("NumberCruncherServer bound and ready.");
        } catch (Exception e) {
            logger.error("Could not bind NumberCruncherServer.", e);

            return;
        }
    }
}
```
The implementation of the  `factor(int number)`  method is of particular relevance. It starts by putting the client's hostname into the  `MDC`  under the key  _client_. The number to factor, as requested by the client, is put into the  `MDC`  under the key  _number_. After computing the distinct factors of the integer parameter, the result is returned to the client. Before returning the result however, the values for the  _client_  and  _number_  are cleared by calling the  `MDC.remove()`  method. Normally, a  `put()`  operation should be balanced by the corresponding  `remove()`  operation. Otherwise, the  `MDC`  will contain stale values for certain keys. We would recommend that whenever possible,  `remove()`  operations be performed within finally blocks, ensuring their invocation regardless of the execution path of the code.

After these theoretical explanations, we are ready to run the number cruncher example. Start the server with the following command:

java chapters.mdc.NumberCruncherServer src/main/java/chapters/mdc/mdc1.xml

The  _mdc1.xml_  configuration file is listed below:

_Example 7.4: Configuration file (logback-examples/src/main/java/chapters/mdc/mdc1.xml)_
```xml
<configuration>  
	<appender  name="CONSOLE"  class="ch.qos.logback.core.ConsoleAppender">  
		<layout>  
			<Pattern>%-4r [%thread] %-5level **C:%X{client} N:%X{number}** - %msg%n</Pattern>  
		</layout>  
	</appender>  

	<root  level="debug">  
		<appender-ref  ref="CONSOLE"/>  
	</root>  
</configuration>
```
Note the use of the  _%X_  conversion specifier within the  Pattern  option.

The following command starts an instance of  `NumberCruncherClient`  application:

`java chapters.mdc.NumberCruncherClient `_`hostname`_

where  _hostname_  is the host where the  `NumberCruncherServer`  is running

Executing multiple instances of the client and requesting the server to factor the numbers 129 from the first client and shortly thereafter the number 71 from the second client, the server outputs the following:
```
**70984 [RMI TCP Connection(4)-192.168.1.6] INFO  C:orion N:129 - Beginning to factor.**
70984 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 2 as a factor.
71093 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 3 as a factor.
71093 [RMI TCP Connection(4)-192.168.1.6] INFO  C:orion N:129 - Found factor 3
71187 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 4 as a factor.
71297 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 5 as a factor.
71390 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 6 as a factor.
**71453 [RMI TCP Connection(5)-192.168.1.6] INFO  C:orion N:71 - Beginning to factor.**
71453 [RMI TCP Connection(5)-192.168.1.6] DEBUG C:orion N:71 - Trying 2 as a factor.
71484 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 7 as a factor.
71547 [RMI TCP Connection(5)-192.168.1.6] DEBUG C:orion N:71 - Trying 3 as a factor.
71593 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 8 as a factor.
71656 [RMI TCP Connection(5)-192.168.1.6] DEBUG C:orion N:71 - Trying 4 as a factor.
71687 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 9 as a factor.
71750 [RMI TCP Connection(5)-192.168.1.6] DEBUG C:orion N:71 - Trying 5 as a factor.
71797 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 10 as a factor.
71859 [RMI TCP Connection(5)-192.168.1.6] DEBUG C:orion N:71 - Trying 6 as a factor.
71890 [RMI TCP Connection(4)-192.168.1.6] DEBUG C:orion N:129 - Trying 11 as a factor.
71953 [RMI TCP Connection(5)-192.168.1.6] DEBUG C:orion N:71 - Trying 7 as a factor.
72000 [RMI TCP Connection(4)-192.168.1.6] INFO  C:orion N:129 - Found factor 43
72062 [RMI TCP Connection(5)-192.168.1.6] DEBUG C:orion N:71 - Trying 8 as a factor.
72156 [RMI TCP Connection(5)-192.168.1.6] INFO  C:orion N:71 - Found factor 71
```
The clients were run from a machine called  _orion_  as can be seen in the above output. Even if the server processes the requests of clients near-simultaneously in separate threads, the logging output pertaining to each client request can be distinguished by studying the output of the  `MDC`. Note for example the stamp associated with  _number_, i.e. the number to factor.

The attentive reader might have observed that the thread name could also have been used to distinguish each request. The thread name can cause confusion if the server side technology recycles threads. In that case, it may be hard to determine the boundaries of each request, that is, when a given thread finishes servicing a request and when it begins servicing the next. Because the  `MDC`  is under the control of the application developer,  `MDC`  stamps do not suffer from this problem.

### [Automating access to the  `MDC`](http://logback.qos.ch/manual/mdc.html#autoMDC)

As we've seen, the  `MDC`  is very useful when dealing with multiple clients. In the case of a web application that manages user authentication, one simple solution could be to set the user's name in the  `MDC`  and remove it once the user logs out. Unfortunately, it is not always possible to achieve reliable results using this technique. Since  `MDC`  manages data on a  _per thread_  basis, a server that recycles threads might lead to false information contained in the  `MDC`.

To allow the information contained in the  `MDC`  to be correct at all times when a request is processed, a possible approach would be to store the username at the beginning of the process, and remove it at the end of said process. A servlet  [`Filter`](http://java.sun.com/javaee/5/docs/api/javax/servlet/Filter.html)  comes in handy in this case.

Within the servlet filter's  `doFilter`  method, we can retrieve the relevant user data through the request (or a cookie therein), store it the  `MDC`. Subsequent processing by other filters and servlets will automatically benefit from the MDC data that was stored previously. Finally, when our servlet filter regains control, we have an opportunity to clean MDC data.

Here is an implementation of such a filter:

_Example 7.5: User servlet filter ( [logback-examples/src/main/java/chapters/mdc/UserServletFilter.java)](http://logback.qos.ch/xref/chapters/mdc/UserServletFilter.html)_
```java
package chapters.mdc;

import java.io.IOException;
import java.security.Principal;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;

import org.slf4j.MDC;

public class UserServletFilter implements Filter {

    private final String USER_KEY = "username";

    public void destroy() {
    }

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

        boolean successfulRegistration = false;
        HttpServletRequest req = (HttpServletRequest) request;
        Principal principal = req.getUserPrincipal();
        // Please note that we also could have used a cookie to
        // retrieve the user name

        if (principal != null) {
            String username = principal.getName();
            successfulRegistration = registerUsername(username);
        }

        try {
            chain.doFilter(request, response);
        } finally {
            if (successfulRegistration) {
                MDC.remove(USER_KEY);
            }
        }
    }

    public void init(FilterConfig arg0) throws ServletException {
    }

    /**
     * Register the user in the MDC under USER_KEY.
     * 
     * @param username
     * @return true id the user can be successfully registered
     */
    private boolean registerUsername(String username) {
        if (username != null && username.trim().length() > 0) {
            MDC.put(USER_KEY, username);
            return true;
        }
        return false;
    }

}
```
When the filter's  `doFilter()`  method is called, it first looks for a  `java.security.Principal`  object in the request. This object contains the name of the currently authenticated user. If a user information is found, it is registered in the  `MDC`.

Once the filter chain has completed, the filter removes the user information from the  `MDC`.

The approach we just outlined sets MDC data only for the duration of the request and only for the thread processing it. Other threads are unaffected. Furthermore, any given thread will contain correct MDC data at any point in time.

### [MDC And Managed Threads](http://logback.qos.ch/manual/mdc.html#managedThreads)

A copy of the mapped diagnostic context can not always be inherited by worker threads from the initiating thread. This is the case when  `java.util.concurrent.Executors`  is used for thread management. For instance,  `newCachedThreadPool`  method creates a  `ThreadPoolExecutor`  and like other thread pooling code, it has intricate thread creation logic.

In such cases, it is recommended that  `MDC.getCopyOfContextMap()`  is invoked on the original (master) thread before submitting a task to the executor. When the task runs, as its first action, it should invoke  `MDC.setContextMapValues()`  to associate the stored copy of the original MDC values with the new  `Executor`  managed thread.

### [MDCInsertingServletFilter](http://logback.qos.ch/manual/mdc.html#mis)

Within web applications, it often proves helpful to know the hostname, request uri and user-agent associated with a given HTTP request.  [`MDCInsertingServletFilter`](http://logback.qos.ch/xref/ch/qos/logback/classic/helpers/MDCInsertingServletFilter.html)  inserts such data into the MDC under the following keys.
|**MDC key**|**MDC value**|
|---|---|
|`req.remoteHost`|as returned by the [getRemoteHost()](http://java.sun.com/j2ee/sdk_1.3/techdocs/api/javax/servlet/ServletRequest.html#getRemoteHost%28%29) method|
|`req.xForwardedFor`|value of the ["X-Forwarded-For"](http://en.wikipedia.org/wiki/X-Forwarded-For) header|
|`req.method`|as returned by [getMethod()](http://java.sun.com/j2ee/sdk_1.3/techdocs/api/javax/servlet/http/HttpServletRequest.html#getMethod%28%29) method|
|`req.requestURI`|as returned by [getRequestURI()](http://java.sun.com/j2ee/sdk_1.3/techdocs/api/javax/servlet/http/HttpServletRequest.html#getRequestURI%28%29) method|
|`req.requestURL`|as returned by [getRequestURL()](http://java.sun.com/j2ee/sdk_1.3/techdocs/api/javax/servlet/http/HttpServletRequest.html#getRequestURL%28%29) method|
|`req.queryString`|as returned by [getQueryString()](http://java.sun.com/j2ee/sdk_1.3/techdocs/api/javax/servlet/http/HttpServletRequest.html#getQueryString%28%29) method|
|`req.userAgent`|value of the "User-Agent" header|
To install  `MDCInsertingServletFilter`  add the following lines to your web-application's  _web.xml_  file
```xml
<filter>  
	<filter-name>MDCInsertingServletFilter</filter-name>  
	<filter-class> 
		ch.qos.logback.classic.helpers.MDCInsertingServletFilter 
	</filter-class>  
</filter>  
<filter-mapping>  
	<filter-name>MDCInsertingServletFilter</filter-name>  
	<url-pattern>/*</url-pattern>  
</filter-mapping>
```
**If your web-app has multiple filters, make sure that  `MDCInsertingServletFilter`  is declared before other filters.**  For example, assuming the main processing in your web-app is done in filter 'F', the MDC values set by  `MDCInsertingServletFilter`  will not be seen by the code invoked by 'F' if  `MDCInsertingServletFilter`  comes after 'F'.

Once the filter is installed, values corresponding to each MDC key will be output by the %X  [conversion word](http://logback.qos.ch/manual/layouts.html#conversionWord)  according to the key passes as first option. For example, to print the remote host followed by the request URI on one line, the date followed by the message on the next, you would set  `PatternLayout`'s pattern to:
`%X{req.remoteHost} %X{req.requestURI}%n%d - %m%n`