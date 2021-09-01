I do not think the TO would change it to 1 on its own. Kafka even doesn't allow to decrease the number of partitions. So I would expect that it was already created with 1. A theory about how this could have happened:

* The default settings in Kafka is that when consumer / producer connects to a topic which doesn't exist it will be auto-created by Kafka with the default settings (by default 1 partition, 1 replica).
* The creation of topics by the Cluster Operator is asynchronous. So when you do kubectl apply or kubectl create, it only created the resource in the ETCD database. Afterwards it notifies the Topic Operator which creates the topic.
* But since the previous step is asynchronous, it can have some delay. If you didn't disabled your Kafka brokers from auto creating the topics, it might happen that yous consumer / producer connects before the topic was created by the Topic Operator and triggers its creation in Kafka with the default settings. The Topic Operator would then basically just update the CR to the value form Kafka.

Is it possible that something like this happened in your case? This is pretty much a race condition which the operator cannot easily handle.

To avoid this, the best is to disable the topic autocreation in Kafka:

```
# ...
spec:
  kafka:
    # ...
    config:
      auto.create.topics.enable: "false"
    # ...
```


https://github.com/strimzi/strimzi-kafka-operator/issues/1572#issuecomment-486838749