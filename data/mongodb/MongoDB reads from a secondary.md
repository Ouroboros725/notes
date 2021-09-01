You have to set "slave okay" mode to let the mongo shell know that you're allowing reads from a secondary. This is to protect you and your applications from performing eventually consistent reads by accident. You can do this in the shell with:

```rs.slaveOk()```

After that you can query normally from secondaries.

A note about "eventual consistency": under normal circumstances, replica set secondaries have all the same data as primaries within a second or less. Under very high load, data that you've written to the primary may take a while to replicate to the secondaries. This is known as "replica lag", and reading from a lagging secondary is known as an "eventually consistent" read, because, while the newly written data will show up at some point (barring network failures, etc), it may not be immediately available.

**Edit**: You only need to set slaveok when querying from secondaries, and only once per session.

**WARNING**: slaveOk() is deprecated and may be removed in the next major release. Please use secondaryOk() instead. `rs.secondaryOk()`

https://stackoverflow.com/questions/8990158/mongodb-replicates-and-error-err-not-master-and-slaveok-false-code