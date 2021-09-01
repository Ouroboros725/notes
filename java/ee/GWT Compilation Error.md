https://stackoverflow.com/questions/9549303/no-source-code-is-available-for-type-gwt-compilation-error

Java code running in GWT is translated to Javascript, so some classes that work on a JVM won't work with GWT. The HttpClient and related classes are written to work on a JVM with full support for opening sockets, something that isn't allowed in a web browser, so these classes cannot be used.

To open a connection to the server you are using (subject to the browser Same Origin Policy), consider the RequestBuilder class, which allows you to provide a url and an HTTP method, and optionally headers, parameters, data, etc. This class is an abstraction over the XmlHttpRequest object in JavaScript, commonly used for AJAX requests in plain JS.

---

If you use Maven then you can do this.

[maven-gwt-plugin][1] with parameter [compileSourcesArtifacts][2] will do all sources management work and will let you to compile GWT module.

In the module that you want to include, you'll have to [enable the generation of source package][3]. And take a look at [external GWT module example on Github][4].

GWT can't compile any Java class to JavaScript client code. It supports only several base classes. See [GWT JRE Emulation Reference][5].

Example pom.xml:

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

        <dependencies>
            <dependency>
                <groupId>com.my.group</groupId>
                <artifactId>my-artifact</artifactId>
                <version>1.0</version>
            </dependency>
        </dependencies>

        <!-- ... -->
    
        <build>
            <plugins>
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>gwt-maven-plugin</artifactId>
                    <version>2.5.0</version>
                    <!-- ... -->
                    <configuration>
                        <compileSourcesArtifacts>
                            <compileSourcesArtifact>com.my.group:my-artifact</compileSourcesArtifact>
                        </compileSourcesArtifacts>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </project>


[1]: http://mojo.codehaus.org/gwt-maven-plugin/compile-mojo.html
[2]: http://mojo.codehaus.org/gwt-maven-plugin/compile-mojo.html#compileSourcesArtifacts
[3]: http://maven.apache.org/plugins/maven-source-plugin/usage.html
[4]: https://github.com/slavikdobro/bilionix-core
[5]: https://developers.google.com/web-toolkit/doc/latest/RefJreEmulation