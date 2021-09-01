https://forum.liquibase.org/t/configurable-databasechangelog-table-name/570

Is it possible to configure a different name for the DATABASECHANGELOG and DATABASECHANGELOGLOCK tables ?  
I have a scenario where a couple of separate components need to manage their own sets of tables, and they need to do on their own. And yes, they have to access the same schema.

I use Oracle and manage Liquibase through Maven plugin.

Thanks for help.

---

Yes, you can. It looks for liquibase.databaseChangeLogTableName and liquibase.databaseChangeLogLockTableName system properties and will use those rather than the default if passed.

You can also specify your own Database implementation (how varies between 1.9 and 2.0) and override the getDatabaseChangeLogTableName() and getDatabaseChangeLogLockTableName() methods.

---

Got it. Sorry.

---

Is there a way to do this from the command line?



– jst

---

It was not easy to find out the only way I could make it work, so maybe it’ll be useful for someone else:

JAVA_OPTS=-Dliquibase.databaseChangeLogTableName=MYTABLE ./liquibase --url=jdbc… --username=u --password=p --changeLogFile=c.xml updateSQL

INSERT INTO test.MYTABLE (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, DESCRIPTION, COMMENTS, EXECTYPE, LIQUIBASE) VALUES (‘2’, ‘user’, ‘c.xml’, NOW(), 5, ‘7:1d127652402f4036e82f57e2a37cb40c’, ‘createTable’, ‘’, ‘EXECUTED’, ‘3.2.2’);

---

> JAVA_OPTS=-Dliquibase.databaseChangeLogTableName=MYTABLE ./liquibase --url=jdbc… --username=u --password=p --changeLogFile=c.xml updateSQL

Working now . Thanks for help

---

I have a DATABASE which is used by different UI applications.

EX: TABLE1, TABLE2, TABLE3 - application1

```
   TABLE4, TABLE5, TABLE6 - application2</p>

```

```
   TABLE7, TABLE8, TABLE9 - application3</p>

```

But all tables are in same database here. I need separate Changelog table for each set of tables. If this is possible, how can I achieve this?

---

Well, create one changelog per application, and run liquibase for each application too, with - as said previously - the parameter  _-Dliquibase.databaseChangeLogTableName=anameforapp1_  and  _-D_  _getDatabaseChangeLogLockTableName=anameforapp2_.

---

Thanks Gavriel - your example was super helpful to get me going.

I hit  [this weird issue  1](http://forum.liquibase.org/#Topic/49382000001643015) when attempting to configure liquibase control tables against an Oracle database (in first run control tables were created as expected, but on subsequent attempts to migrate the schema the control tables were attempted to be recreated resulting in errors).

Posting the solution I stumbled upon, it may be useful for someone (IMO its an oracle-specific issue):

**Against Oracle Database:**

The configured liquibase control table names SHOULD be specified in ALL CAPS for it to be consistently recognized across multiple liquibase runs (first time setup & subsequent attempts to migrate).

**Against Postgres Database:**

The configured liquibase control table names are not required to be in ALL CAPS, i.e., it works appropriately across multiple liquibase runs even when the table names are specified is in small case.

As Liquibase uses JDBC connectivity to perform its operations using the database-specific driver - I assume its safe to attribute this behavior to ojdbc7.jar (oracle driver I specified in the classpath when performing liquibase operations against Oracle database).

---

> JAVA_OPTS=-Dliquibase.databaseChangeLogTableName=MYTABLE ./liquibase --url=jdbc… --username=u --password=p --changeLogFile=c.xml updateSQL

Working now . Thanks for help

---

Can we please request a new feature to be able to configure not only via system settings. 2 disadvantages exist here:

1.  It forces ITO to change a standard deployment workflow. Which is not good when there are a lot of clients. The solution must exist when we build our application.

2.  When different applications are deployed and they share the same database, I would say it is impossible to control it via system setting. I cannot change it, deploy one application, change the settings again, deploy the next application.


I would offer to include this feature into the master changelog. It is the only file. It will be clear and quite flexible, I think. Probably settings should have higher priority, or, I would even throw an exception if it is configured both, via setting and via the master changelog. User must fix it first. We must be as explicit here as possible.

---

> JAVA_OPTS=-Dliquibase.databaseChangeLogTableName=MYTABLE ./liquibase --url=jdbc… --username=u --password=p --changeLogFile=c.xml updateSQL

Working now . Thanks for help

---

is it required to execute updatesql every deployements

Please suggest me

---

The updateSQL is a Liquibase command that allows you to inspect the SQL that would be executed by the update command, without actually executing it in your database target.

See here for more details:  
[https://docs.liquibase.com/commands/community/updatesql.html  8](https://docs.liquibase.com/commands/community/updatesql.html)

---

HI ,

i am not able to do deployment due to following errors

[liquibase.lockservice] Waiting for changelog lock…

please help me

---

That indicates that the lock is set in the databasechangeloglock table, which means either:

1.  You have another Liquibase deployment running in the same schema/database.
2.  A prior Liquibase deployment failed, leaving the lock set in the databasechangeloglock table.

If your sure there isn’t another Liquibase deployment running, then I’d recommend running the “releaseLocks” command to remove the lock.

[https://docs.liquibase.com/commands/community/releaselocks.html  1](https://docs.liquibase.com/commands/community/releaselocks.html)

---

Thanks for your advice

Regards

---

Does this still works on the latest version? I am using 4.2.0 .

I tried using this syntax:

`java -Dliquibase.databaseChangeLogTableName=NEWCHANGELGOTABLE -jar liquibase.jar update`

And I get an error message:

Error: Could not find or load main class .databaseChangeLogTableName

I could be missing a very obvious step and would appreciate a fresh set of eyes.

Thanks

---

never mind.

the solution is actually easy. it can be added in the liquibase.properties see below or as a parameter in the command line

```
databaseChangeLogTableName=NEWCHANGELGOTABLE
```