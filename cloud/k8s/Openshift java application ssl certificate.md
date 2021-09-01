## Add a service certificate

To secure communication to your service, generate a signed serving certificate and key pair into a secret in the same namespace as the service.

* The generated certificate is only valid for the internal service DNS name <service.name>.<service.namespace>.svc, and are only valid for internal communications.

### Prerequisites:
* You must have a service defined.

### Procedure

1. Annotate the service with `service.beta.openshift.io/serving-cert-secret-name`.

```
$ oc annotate service <service-name> \
     service.beta.openshift.io/serving-cert-secret-name=<secret-name>
```

* Replace `<service-name>` with the name of the service to secure.
* `<secret-name>` will be the name of the generated secret containing the certificate and key pair. For convenience, it is recommended that this be the same as `<service-name>`.

For instance, use the following command to annotate the service `foo`:

```
$ oc annotate service foo service.beta.openshift.io/serving-cert-secret-name=foo
```

2. Examine the service to confirm the annotations are present.

```
$ oc describe service <service-name>
...
Annotations:              service.beta.openshift.io/serving-cert-secret-name: <service-name>
                          service.beta.openshift.io/serving-cert-signed-by: openshift-service-serving-signer@1556850837
...
```

3. After the cluster generates a secret for your service, your PodSpec can mount it, and the Pod will run after it becomes available.

## Add a service certificate to a ConfigMap

A Pod can access the service CA certificate by mounting a ConfigMap that is annotated with `service.beta.openshift.io/inject-cabundle=true`. Once annotated, the cluster automatically injects the service CA certificate into the `service-ca.crt` key on the ConfigMap. Access to this CA certificate allows TLS clients to verify connections to services using service serving certificates.

* After adding this annotation to a ConfigMap all existing data in it is deleted. It is recommended to use a separate ConfigMap to contain the `service-ca.crt`, instead of using the same ConfigMap that stores your Pod’s configuration.

### Procedure
1. Annotate the ConfigMap with `service.beta.openshift.io/inject-cabundle=true`.

```
$ oc annotate configmap <configmap-name> \
     service.beta.openshift.io/inject-cabundle=true
```

* Replace `<configmap-name>` with the name of the ConfigMap to annotate.

Explicitly referencing the `service-ca.crt` key in a volumeMount will prevent a Pod from starting until the ConfigMap has been injected with the CA bundle.

For instance, to annotate the ConfigMap `foo` the following command would be used:

```
$ oc annotate configmap foo service.beta.openshift.io/inject-cabundle=true
```

2. View the ConfigMap to ensure the certificate has been generated. This appears as a `service-ca.crt` in the YAML output.

```
$ oc get configmap <configmap-name> -o yaml
apiVersion: v1
data:
  service-ca.crt: |
    -----BEGIN CERTIFICATE-----
...
```
https://docs.openshift.com/container-platform/4.1/authentication/certificates/service-serving-certificate.html

-----

Add a Certificate to a Truststore Using Keytool

Run the keytool -import -alias ALIAS -file public.cert -storetype TYPE -keystore server.truststore command:
`keytool -import -noprompt -trustcacerts -alias teiid -file public.cert -storetype JKS -keystore server.truststore -storepass password`

https://gist.github.com/matthiassb/60516e9f857e27330676986e648a0065

https://access.redhat.com/documentation/en-us/red_hat_jboss_data_virtualization/6.2/html/security_guide/add_a_certificate_to_a_truststore_using_keytool

-----

## Starting the application server with the keystores

When you start the Tomcat application server, you must specify the location and passphrase of the keystore and truststore files.

You used the following JVM java -D system property command arguments to specify the keystore and truststore files:
* -Djavax.net.ssl.keyStore specifies the keystore file.
* -Djavax.net.ssl.keyStorePassword specifies the passphrase of the keystore.
* -Djavax.net.ssl.trustStore specifies the truststore file to use to validate client certificates.
* -Djavax.net.ssl.trustStorePassword specifies the passphrase to access the truststore file.

One way to set these system properties is to use the java command from the command line.

https://docs.oracle.com/cd/E29585_01/PlatformServices.61x/security/src/csec_ssl_jsp_start_server.html

https://developers.redhat.com/blog/2017/11/22/dynamically-creating-java-keystores-openshift/