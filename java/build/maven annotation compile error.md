https://stackoverflow.com/questions/36248959/bad-service-configuration-file-or-exception-thrown-while-constructing-processor

The default maven lifecycle runs javac with **javax.annotation.processing.Processor** file as a part of classpath. This cause compiler to expect a compiled instance of annotation processors listed in the files. But `LogMeCustomAnnotationProcessor` is not compiled at that moment so compiler raises "Bad service configuration file ..." error. See [bug report][1].
<br>
<br>
To solve this issue maven compilation phase can be separated to compile annotation processor at the first place and then compile whole project.

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
                <executions>
                    <execution>
                        <id>default-compile</id>
                        <configuration>
                            <compilerArgument>-proc:none</compilerArgument>
                            <includes>
                                <include>fun/n/learn/annotation/LogMeCustomAnnotationProcessor.java</include>
                                <!--include dependencies required for LogMeCustomAnnotationProcessor -->
                            </includes>
                        </configuration>
                    </execution>
                    <execution>
                        <id>compile-project</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

`default-compile` execution compiles `LogMeCustomAnnotationProcessor` with disabled annotation processing in order to have successful compilation.
<br>
`compile-project` compiles whole project with annotaton processing.

[1]: https://issues.apache.org/jira/browse/MCOMPILER-97