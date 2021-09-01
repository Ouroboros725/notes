https://github.com/qos-ch/logback-contrib/wiki/JSON

The Logback JSON extension allows log messages to be formatted as [JSON](http://www.json.org).  It must be paired with another extension that enables your desired JSON Processor (see below).

This extension is organized as 3 modules:

- `logback-ext-json-core`: Provides base classes needed for handling any type of logging event.
- `logback-ext-json-access`: Provides classes for formatting access Logback [IAccessEvent]
  (http://logback.qos.ch/apidocs/ch/qos/logback/access/spi/IAccessEvent.html) events.
- `logback-ext-json-classic`: Provides classes for formatting standard Logback [ILoggingEvent](http://logback.qos.ch/apidocs/ch/qos/logback/classic/spi/ILoggingEvent.html) events.

## JSON Processor

The above JSON modules provide only general Logback JSON support.  They cannot function alone - they must be paired with a JSON Processor of your choice, based on your application's runtime requirements.

The `logback-ext-jackson` module is currently the only out-of-the-box JSON Processor module, which uses the [Jackson](http://jackson.codehaus.org) JSON Processor - a very common processor in Java applications.  Additional JSON processor module implementations can be added by the community as necessary.

## Usage

### 1. Add the following .jars to your application's classpath:
<table>
  <tr>
    <th>Jar</th>
    <th>Maven or Ant+Ivy</th>
  </tr>
  <tr>
    <td><code>logback-ext-json-core-<em>version</em>.jar</code></td>
    <td>Not necessary.  Pulled automatically by other dependencies below.</td>
  </tr>
  <tr>
    <td><code>logback-ext-json-classic-<em>version</em>.jar</code></td>
    <td><pre>
&lt;dependency&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;groupId&gt;ch.qos.logback.extensions&lt;/groupId&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;artifactId&gt;logback-ext-json-classic&lt;/artifactId&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;version&gt;<em>version</em>&lt;/version&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;scope&gt;runtime&lt;/scope&gt;
&lt;/dependency&gt;
</pre></td>
  </tr>
  <tr>
    <td><code>logback-ext-jackson-<em>version</em>.jar</code></td>
    <td><pre>
&lt;dependency&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;groupId&gt;ch.qos.logback.extensions&lt;/groupId&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;artifactId&gt;logback-ext-jackson&lt;/artifactId&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;version&gt;<em>version</em>&lt;/version&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;scope>runtime&lt;/scope&gt;
&lt;/dependency&gt;
</pre></td>
  </tr>
  <tr>
    <td>A Jackson implementation .jar of your choice<b><sup>1</sup></b></td>
    <td><pre>
&lt;dependency&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;groupId&gt;org.codehaus.jackson&lt;/groupId&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;artifactId&gt;jackson-mapper-asl&lt;/artifactId&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;version&gt;<em>Jackson Version</em>&lt;/version&gt;
    &nbsp;&nbsp;&nbsp;&nbsp;&lt;scope>runtime&lt;/scope&gt;
&lt;/dependency&gt;
</pre></td>
  </tr>
</table>
<b><sup>1</sup></b> Logback Extensions compiles against Jackson 1.9.5 or later, but should work with most earlier Jackson versions.


### 2. Configure `logback.xml`
Example TBD