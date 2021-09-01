> message: 'Revision creation failed with message: Failed to list GroupVersionResource
>      {Group:build.knative.dev Version:v1alpha1 Resource:builds}: builds.build.knative.dev
>      is forbidden: User "system:serviceaccount:knative-serving:controller" cannot
>      list builds.build.knative.dev in the namespace "kt": no RBAC policy matched.'


https://github.com/triggermesh/pipeline-tasks/blob/master/README.md

Out of the box Serving failed to create the build, error: `Revision creation failed with message: "taskruns.pipeline.knative.dev is forbidden: User \"system:serviceaccount:knative-serving:controller\" cannot create taskruns.pipeline.knative.dev in the namespace \"default\"".`. That can be solved by granting cluster-admin rights `kubectl create clusterrolebinding serving-controller-pipeline-build --clusterrole=cluster-admin --serviceaccount=knative-serving:controller --namespace=knative-serving` but more restricted access is probably recommended.