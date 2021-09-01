Kafka Connect can work in two modes - the so called standalone and distributed. The Kafka Connect deployment from Strimzi is using the distributed mode in which the connectors are configured through the REST interface. The reason why we use the distributed mode is that the standalone mode is using disk to store the offsets etc. and that makes is a bit harder to handle on Kubernetes.

We have plans for *operator for connectors* which will do that. But it doesn't exist today. So for now it is a bit crude - you can for example just use `curl`. Here is an example how to do it with *one command* (you would need to replace it with your plugin and your configuration and you might need to change the label selector in the command).

```kubectl exec -ti $(kubectl get pod -l app=my-connect-cluster -o=jsonpath='{.items[0].metadata.name}') -- curl -X POST -H "Content-Type: application/json" --data '{ "name": "sink-test", "config": { "connector.class": "FileStreamSink", "tasks.max": "3", "topics": "my-topic", "file": "/tmp/test.sink.txt" } }' http://localhost:8083/connectors```

https://github.com/strimzi/strimzi-kafka-operator/issues/1438