问题描述：MySQL 配置 HA后，启动 Hive，报错
```java
Exception in thread "main" java.lang.RuntimeException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
	at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:522)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:677)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:621)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.util.RunJar.run(RunJar.java:221)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:136)
Caused by: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
	at org.apache.hadoop.hive.metastore.MetaStoreUtils.newInstance(MetaStoreUtils.java:1523)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.<init>(RetryingMetaStoreClient.java:86)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:132)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:104)
	at org.apache.hadoop.hive.ql.metadata.Hive.createMetaStoreClient(Hive.java:3005)
	at org.apache.hadoop.hive.ql.metadata.Hive.getMSC(Hive.java:3024)
	at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:503)
	... 8 more
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.apache.hadoop.hive.metastore.MetaStoreUtils.newInstance(MetaStoreUtils.java:1521)
	... 14 more
Caused by: javax.jdo.JDOException: Couldnt obtain a new sequence (unique id) : Cannot execute statement: impossible to write to binary log since BINLOG_FORMAT = STATEMENT and at least one table uses a storage engine limited to row-based logging. InnoDB is limited to row-logging when transaction isolation level is READ COMMITTED or READ UNCOMMITTED.
NestedThrowables:
java.sql.SQLException: Cannot execute statement: impossible to write to binary log since BINLOG_FORMAT = STATEMENT and at least one table uses a storage engine limited to row-based logging. InnoDB is limited to row-logging when transaction isolation level is READ COMMITTED or READ UNCOMMITTED.
	at org.datanucleus.api.jdo.NucleusJDOHelper.getJDOExceptionForNucleusException(NucleusJDOHelper.java:596)
	at org.datanucleus.api.jdo.JDOPersistenceManager.jdoMakePersistent(JDOPersistenceManager.java:732)
	at org.datanucleus.api.jdo.JDOPersistenceManager.makePersistent(JDOPersistenceManager.java:752)
	at org.apache.hadoop.hive.metastore.ObjectStore.setMetaStoreSchemaVersion(ObjectStore.java:6773)
	at org.apache.hadoop.hive.metastore.ObjectStore.checkSchema(ObjectStore.java:6670)
	at org.apache.hadoop.hive.metastore.ObjectStore.verifySchema(ObjectStore.java:6645)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.hive.metastore.RawStoreProxy.invoke(RawStoreProxy.java:114)
	at com.sun.proxy.$Proxy6.verifySchema(Unknown Source)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.getMS(HiveMetaStore.java:572)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.createDefaultDB(HiveMetaStore.java:624)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.init(HiveMetaStore.java:461)
	at org.apache.hadoop.hive.metastore.RetryingHMSHandler.<init>(RetryingHMSHandler.java:66)
	at org.apache.hadoop.hive.metastore.RetryingHMSHandler.getProxy(RetryingHMSHandler.java:72)
	at org.apache.hadoop.hive.metastore.HiveMetaStore.newRetryingHMSHandler(HiveMetaStore.java:5762)
	at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.<init>(HiveMetaStoreClient.java:199)
	at org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient.<init>(SessionHiveMetaStoreClient.java:74)
	... 19 more
Caused by: java.sql.SQLException: Cannot execute statement: impossible to write to binary log since BINLOG_FORMAT = STATEMENT and at least one table uses a storage engine limited to row-based logging. InnoDB is limited to row-logging when transaction isolation level is READ COMMITTED or READ UNCOMMITTED.
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:1078)
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:4237)
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:4169)
	at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2617)
	at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2778)
	at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2825)
	at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:2156)
	at com.mysql.jdbc.PreparedStatement.executeQuery(PreparedStatement.java:2323)
	at com.jolbox.bonecp.PreparedStatementHandle.executeQuery(PreparedStatementHandle.java:174)
	at org.datanucleus.store.rdbms.ParamLoggingPreparedStatement.executeQuery(ParamLoggingPreparedStatement.java:381)
	at org.datanucleus.store.rdbms.SQLController.executeStatementQuery(SQLController.java:504)
	at org.datanucleus.store.rdbms.valuegenerator.SequenceTable.getNextVal(SequenceTable.java:198)
	at org.datanucleus.store.rdbms.valuegenerator.TableGenerator.reserveBlock(TableGenerator.java:190)
	at org.datanucleus.store.valuegenerator.AbstractGenerator.reserveBlock(AbstractGenerator.java:305)
	at org.datanucleus.store.rdbms.valuegenerator.AbstractRDBMSGenerator.obtainGenerationBlock(AbstractRDBMSGenerator.java:170)
	at org.datanucleus.store.valuegenerator.AbstractGenerator.obtainGenerationBlock(AbstractGenerator.java:197)
	at org.datanucleus.store.valuegenerator.AbstractGenerator.next(AbstractGenerator.java:105)
	at org.datanucleus.store.rdbms.RDBMSStoreManager.getStrategyValueForGenerator(RDBMSStoreManager.java:2005)
	at org.datanucleus.store.AbstractStoreManager.getStrategyValue(AbstractStoreManager.java:1386)
	at org.datanucleus.ExecutionContextImpl.newObjectId(ExecutionContextImpl.java:3827)
	at org.datanucleus.state.JDOStateManager.setIdentity(JDOStateManager.java:2571)
	at org.datanucleus.state.JDOStateManager.initialiseForPersistentNew(JDOStateManager.java:513)
	at org.datanucleus.state.ObjectProviderFactoryImpl.newForPersistentNew(ObjectProviderFactoryImpl.java:232)
	at org.datanucleus.ExecutionContextImpl.newObjectProviderForPersistentNew(ExecutionContextImpl.java:1414)
	at org.datanucleus.ExecutionContextImpl.persistObjectInternal(ExecutionContextImpl.java:2218)
	at org.datanucleus.ExecutionContextImpl.persistObjectWork(ExecutionContextImpl.java:2065)
	at org.datanucleus.ExecutionContextImpl.persistObject(ExecutionContextImpl.java:1913)
	at org.datanucleus.ExecutionContextThreadedImpl.persistObject(ExecutionContextThreadedImpl.java:217)
	at org.datanucleus.api.jdo.JDOPersistenceManager.jdoMakePersistent(JDOPersistenceManager.java:727)
	... 37 more

```


问题解决：
	在 mysql的`/usr/my.conf `中增加
```sql
binlog_format=mixed
```