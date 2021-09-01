```
0) make sure you have proper credentials. 
1) docker login (so that the .docker/config.json gets created)
2) oc create secret generic "redhat.io" --from-file=.dockerconfigjson=config.json --type=kubernetes.io/dockerconfigjson
3) oc import-image --all=true redhat-sso72-openshift
```

https://access.redhat.com/RegistryAuthentication