# Supported Encodings

The  `java.io.InputStreamReader`,  `java.io.OutputStreamWriter`,  `java.lang.String`  classes, and classes in the  `java.nio.charset`  package can convert between Unicode and a number of other character encodings. The supported encodings vary between different implementations of Java SE 8. The class description for  `[java.nio.charset.Charset](https://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html)`  lists the encodings that any implementation of Java SE 8 is required to support.

JDK 8 for all platforms (Solaris, Linux, and Microsoft Windows) and JRE 8 for Solaris and Linux support all encodings shown on this page. JRE 8 for Microsoft Windows may be installed as a complete international version or as a European languages version. By default, the JRE 8 installer installs a European languages version if it recognizes that the host operating system only supports European languages. If the installer recognizes that any other language is needed, or if the user requests support for non-European languages in a customized installation, a complete international version is installed. The European languages version only supports the encodings shown in the following Basic Encoding Set table. The international version (which includes the  lib/charsets.jar  file) supports all encodings shown on this page.

The following tables show the encoding sets supported by Java SE 8. The canonical names used by the new  `java.nio`  APIs are in many cases not the same as those used in the  `java.io`  and  `java.lang`  APIs.

## Basic Encoding Set (contained in lib/rt.jar)

https://docs.oracle.com/javase/8/docs/technotes/guides/intl/encoding.doc.html

## Extended Encoding Set (contained in lib/charsets.jar)

https://docs.oracle.com/javase/8/docs/technotes/guides/intl/encoding.doc.html

