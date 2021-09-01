Add the security policy `anyuid` to the service account responsible for creating your deployment, by default this user is default. The dash z indicates that we want to manipulate a service account.

`oc adm policy add-scc-to-user anyuid -z default`

https://dodgydudes.se/allow-containers-to-run-as-root-on-openshift-3-10/

---

you can get root permission as granting anyuid to jenkins service account. “oc adm policy add-scc-to-user anyuid -z jenkins”, refer [Enable Container Images that Require Root](https://docs.openshift.com/container-platform/3.11/admin_guide/manage_scc.html#enable-dockerhub-images-that-require-root) for more details.

check the uid using `id` cmd after `oc rsh <jenkins pod>`

https://stackoverflow.com/questions/53408296/openshift-container-as-root-user#comment93751316_53412273

---

Enable Container Images that Require Root
Some container images (examples: `postgres` and `redis`) require root access and have certain expectations about how volumes are owned. For these images, add the service account to the `anyuid` SCC.

```$ oc adm policy add-scc-to-user anyuid system:serviceaccount:myproject:mysvcacct```

https://docs.openshift.com/container-platform/3.11/admin_guide/manage_scc.html#enable-dockerhub-images-that-require-root