https://stackoverflow.com/questions/51052406/good-practice-when-using-kafka-with-jpa

---

By looking at your question, I'm assuming that you are trying to achieve CDC (Change Data Capture) of your OLTP System, i.e. logging every change that is going to the transactional database. There are two ways to approach this.

Application code does dual writes to transactional DB as well as Kafka. It is inconsistent and hampers the performance. Inconsistent, because when you make the dual write to two independent systems, the data gets screwed when either of the writes fails and pushing data to Kafka in transaction flow adds latency, which you don't want to compromise on.
Extract changes from DB commit (either database/application-level triggers or transaction log) and send it to Kafka. It is very consistent and doesn't affect your transaction at all. Consistent because the DB commit logs are the reflections of the DB transactions after successful commits. There are a lot of solutions available which leverage this approach like [databus](https://github.com/linkedin/databus), [maxwell](https://github.com/zendesk/maxwell), [debezium](https://github.com/debezium/debezium) etc.
If CDC is your use case, try using any of the already available solutions.

---

As others have said, you could use change data capture to safely propagate the changes applied to your database to Apache Kafka. You cannot update the database and Kafka in a single transaction as the latter doesn't support any kind of 2-phase-commit protocol.

You might either CDC the tables themselves, or, if you wish to have some more control about the structure sent towards Kafka, apply the "outbox" pattern. In that case, your application would write to its actual business tables as well as an "outbox" table which contains the messages to send to Kafka. You can find a detailed description of this approach in this [blog post](https://debezium.io/blog/2019/02/19/reliable-microservices-data-exchange-with-the-outbox-pattern/).

Disclaimer: I'm the author of this post and the lead of Debezium, one of the CDC solutions mentioned in some of the other answers.