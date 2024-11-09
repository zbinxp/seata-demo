# seata lab

1. setup nacos by running `docker-compose -f docker-compose.yml up -d nacos`
2. open nacos admin page `http://localhost:8848/nacos`, add configuration for seata server.

- dataId: seataServer.properties
- group: SEATA_GROUP
- configuration content:

```properties
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.cj.jdbc.Driver
store.db.url=jdbc:mysql://mysql:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=demo
store.db.password=demopass
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
# log configuration
server.recovery.committingRetryPeriod=1000
server.recovery.asynCommittingRetryPeriod=1000
server.recovery.rollbackingRetryPeriod=1000
server.recovery.timeoutRetryPeriod=1000
server.maxCommitRetryTimeout=-1
server.maxRollbackRetryTimeout=-1
server.rollbackRetryTimeoutUnlockEnable=false
server.undo.logSaveDays=7
server.undo.logDeletePeriod=86400000
transport.serialization=seata
transport.compressor=none
# disable metrics
metrics.enabled=false
metrics.registryType=compact
metrics.exporterList=prometheus
metrics.exporterPrometheusPort=9898
```

Now we'll setup mysql.

3. setup mysql and seater-server by running `docker-compose -f docker-compose.yml up -d seata-server`. You can check it out on nacos that whether seata-server gets registered.

4. setup database for the lab using `script/init-lab.sql`, which Iwe create a database named `seata_lab` with four tables: `account_tbl`,`stock_tbl`,`order_tbl`,`undo_log`(used in AT mode).

Before we actually run the lab, we need client configuration in nacos for out springboot projects. 

- dataId: client.properties
- group: SEATA_GROUP
- configuration content

```properties
# align with cluster config in conf/application.yml
service.vgroupMapping.my_test_tx_group=SH 
```

4. open `springboot-feign-seata-xa` project, run `mvn spring-boot:run` to start the microservice projects in order: account -> storage -> order -> business.

5. access `http://localhost:8084/purchase?rollback=true` to test a distributed transaction that is rollbacked.

6. the code test [XA](https://en.wikipedia.org/wiki/X/Open_XA) mode in seata, you can change it into AT mode by commenting out `@EnableAutoDataSourceProxy(dataSourceProxyMode = "XA")` for each project in entrypoint file, as it's the default mode in seata.

# TODO

- TCC lab
- SAGA lab