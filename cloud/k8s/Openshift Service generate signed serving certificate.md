To secure communication to your service, have the cluster generate a signed serving certificate/key pair into a secret in your namespace. To do this, set the **service.alpha.openshift.io/serving-cert-secret-name** annotation on your service with the value set to the name you want to use for your secret. Then, your **PodSpec** can mount that secret. When it is available, your pod will run. The certificate will be good for the internal service DNS name, **<service.name>.<service.namespace>.svc**.

The certificate and key are in PEM format, stored in **tls.crt** and **tls.key** respectively. The certificate/key pair is automatically replaced when expiration is within one hour. View the expiration date in the **service.alpha.openshift.io/expiry** annotation on the secret, which is in RFC3339 format.

Other pods can trust cluster-created certificates (which are only signed for internal DNS names), by using the CA bundle in the ***/var/run/secrets/kubernetes.io/serviceaccount/service-ca.crt*** file that is automatically mounted in their pod.

The signature algorithm for this feature is **x509.SHA256WithRSA**. To manually rotate, delete the generated secret. A new certificate is created.

https://docs.openshift.com/container-platform/3.11/dev_guide/secrets.html#service-serving-certificate-secrets