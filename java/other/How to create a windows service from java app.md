https://stackoverflow.com/questions/68113/how-to-create-a-windows-service-from-java-app

I've had some luck with [the Java Service Wrapper][1]


[1]: http://wrapper.tanukisoftware.org/doc/english/introduction.html

---

[Apache Commons Daemon][1] is a good alternative. It has [Procrun][2] for windows services, and [Jsvc][3] for unix daemons. It uses less restrictive Apache license, and Apache Tomcat uses it as a part of itself to run on Windows and Linux! To get it work is a bit tricky, but there is an [exhaustive article][5] with working example.

Besides that, you may look at the bin\service.bat in [Apache Tomcat][4] to get an idea how to setup the service.  In Tomcat they rename the Procrun binaries (prunsrv.exe -> tomcat6.exe, prunmgr.exe -> tomcat6w.exe).

Something I struggled with using Procrun, your start and stop methods must accept the parameters (String[] argv).  For example "start(String[] argv)" and "stop(String[] argv)" would work, but "start()" and "stop()" would cause errors.  If you can't modify those calls, consider making a bootstrapper class that can massage those calls to fit your needs.



[1]: http://commons.apache.org/daemon/index.html
[2]: http://commons.apache.org/daemon/procrun.html
[3]: http://commons.apache.org/daemon/jsvc.html
[4]: http://tomcat.apache.org/
[5]: http://web.archive.org/web/20090228071059/http://blog.platinumsolutions.com/node/234