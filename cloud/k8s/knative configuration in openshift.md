this is the required security set up in any new project, for it to work with istio and by extension, with knative:
```
  oc adm policy add-scc-to-user privileged -z default
  oc adm policy add-scc-to-user anyuid -z default
  ```

---

minishift permission issue solution: [https://github.com/minishift/minishift/issues/2058#issuecomment-385286832](https://github.com/minishift/minishift/issues/2058#issuecomment-385286832)

`oc login -u system:admin -n default` issuing this log-in as admin prior command sorts out all the issues and as follows I could issue administrative command `oc adm policy add-scc-to-user anyuid -z default -n projectname` successfully, without the error message `...`
`securitycontextconstraints.security.openshift.io at the cluster scope: User "developer"`
