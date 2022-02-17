### Semisynchronous Replication

- MySQL replication by default is asynchronous. The source writes events to its binary log and replicas request them when they are ready. The source does not know whether or when a replica has retrieved and processed the transactions, and there is no guarantee that any event ever reaches any replica. With asynchronous replication, if the source crashes, transactions that it has committed might not have been transmitted to any replica. Failover from source to replica in this case might result in failover to a server that is missing transactions relative to the sourc

- With fully synchronous replication, when a source commits a transaction, all replicas have also committed the transaction before the source returns to the session that performed the transaction. Fully synchronous replication means failover from the source to any replica is possible at any time. The drawback of fully synchronous replication is that there might be a lot of delay to complete a transaction.

- Semisynchronous replication falls between asynchronous and fully synchronous replication. The source waits until at least one replica has received and logged the events (the required number of replicas is configurable), and then commits the transaction. The source does not wait for all replicas to acknowledge receipt, and it requires only an acknowledgement from the replicas, not that the events have been fully executed and committed on the replica side. Semisynchronous replication therefore guarantees that if the source crashes, all the transactions that it has committed have been transmitted to at least one replica.

- Compared to asynchronous replication, semisynchronous replication provides improved data integrity, because when a commit returns successfully, it is known that the data exists in at least two places. Until a semisynchronous source receives acknowledgment from the required number of replicas, the transaction is on hold and not committed.

- Compared to fully synchronous replication, semisynchronous replication is faster, because it can be configured to balance your requirements for data integrity (the number of replicas acknowledging receipt of the transaction) with the speed of commits, which are slower due to the need to wait for replicas.

- The performance impact of semisynchronous replication compared to asynchronous replication is the tradeoff for increased data integrity. The amount of slowdown is at least the TCP/IP roundtrip time to send the commit to the replica and wait for the acknowledgment of receipt by the replica. This means that semisynchronous replication works best for close servers communicating over fast networks, and worst for distant servers communicating over slow networks. Semisynchronous replication also places a rate limit on busy sessions by constraining the speed at which binary log events can be sent from source to replica. When one user is too busy, this slows it down, which can be useful in some deployment situations.

- Semisynchronous replication between a source and its replicas operates as follows:

A replica indicates whether it is semisynchronous-capable when it connects to the source.

*If semisynchronous replication is enabled on the source side and there is at least one semisynchronous replica, a thread that performs a transaction commit on the source blocks and waits until at least one semisynchronous replica acknowledges that it has received all events for the transaction, or until a timeout occurs.

*The replica acknowledges receipt of a transaction's events only after the events have been written to its relay log and flushed to disk.

*If a timeout occurs without any replica having acknowledged the transaction, the source reverts to asynchronous replication. When at least one semisynchronous replica catches up, the source returns to semisynchronous replication.

*Semisynchronous replication must be enabled on both the source and replica sides. If semisynchronous replication is disabled on the source, or enabled on the source but on no replicas, the source uses asynchronous replication.

### Configure mysql semi synchronous replication

- In order to reduce the data loss caused by the main database hardware damage and downtime as much as possible, it is recommended to configure the MHA as mysql semi synchronous replication.

```

mysql> show variables like '%plugin_dir%';
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| plugin_dir    | /opt/mysql/lib/plugin/ |
+---------------+------------------------+
1 row in set (0.00 sec)
```

1. Install the related plug-ins (Master, candidate, slave) on the master and slave nodes respectively. To install the plug-ins on mysql, the database needs to support dynamic loading. Check whether it supports the following detection:

```
mysql> show variables like '%have_dynamic%';
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| have_dynamic_loading | YES   |
+----------------------+-------+
1 row in set (0.00 sec)
```

* For all mysql database servers, install the semi synchronization plug-in (semisync ﹣ master.so, semisync ﹣ slave. So):

```
mysql> install plugin rpl_semi_sync_master soname 'semisync_master.so';
Query OK, 0 rows affected (0.30 sec)

mysql> install plugin rpl_semi_sync_slave soname 'semisync_slave.so';
Query OK, 0 rows affected (0.00 sec)
```

```
mysql> show plugins;
+---------------------------------+----------+--------------------+--------------------+---------+
| Name                            | Status   | Type               | Library            | License |
+---------------------------------+----------+--------------------+--------------------+---------+
| keyring_file                    | ACTIVE   | KEYRING            | keyring_file.so    | GPL     |
| binlog                          | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| mysql_native_password           | ACTIVE   | AUTHENTICATION     | NULL               | GPL     |
| sha256_password                 | ACTIVE   | AUTHENTICATION     | NULL               | GPL     |
| caching_sha2_password           | ACTIVE   | AUTHENTICATION     | NULL               | GPL     |
| sha2_cache_cleaner              | ACTIVE   | AUDIT              | NULL               | GPL     |
| daemon_keyring_proxy_plugin     | ACTIVE   | DAEMON             | NULL               | GPL     |
| CSV                             | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| MEMORY                          | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| InnoDB                          | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| INNODB_TRX                      | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMP                      | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMP_RESET                | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMPMEM                   | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMPMEM_RESET             | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMP_PER_INDEX            | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMP_PER_INDEX_RESET      | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_BUFFER_PAGE              | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_BUFFER_PAGE_LRU          | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_BUFFER_POOL_STATS        | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_TEMP_TABLE_INFO          | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_METRICS                  | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_FT_DEFAULT_STOPWORD      | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_FT_DELETED               | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_FT_BEING_DELETED         | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_FT_CONFIG                | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_FT_INDEX_CACHE           | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_FT_INDEX_TABLE           | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_TABLES                   | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_TABLESTATS               | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_INDEXES                  | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_TABLESPACES              | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_COLUMNS                  | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_VIRTUAL                  | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CACHED_INDEXES           | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_SESSION_TEMP_TABLESPACES | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| MyISAM                          | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| MRG_MYISAM                      | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| PERFORMANCE_SCHEMA              | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| TempTable                       | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| ARCHIVE                         | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| BLACKHOLE                       | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| FEDERATED                       | DISABLED | STORAGE ENGINE     | NULL               | GPL     |
| ngram                           | ACTIVE   | FTPARSER           | NULL               | GPL     |
| mysqlx_cache_cleaner            | ACTIVE   | AUDIT              | NULL               | GPL     |
| mysqlx                          | ACTIVE   | DAEMON             | NULL               | GPL     |
| rpl_semi_sync_master            | ACTIVE   | REPLICATION        | semisync_master.so | GPL     |
| rpl_semi_sync_slave             | ACTIVE   | REPLICATION        | semisync_slave.so  | GPL     |
+---------------------------------+----------+--------------------+--------------------+---------+
48 rows in set (0.00 sec)
```

2. Modify my.cnf file and configure master-slave synchronization

Note: if the primary mysql server already exists, only later can the secondary mysql server be set up. Before configuring data synchronization, copy the database to be synchronized from the primary mysql server to the secondary mysql server (for example, backup the database on the primary mysql server first, and then restore it from the mysql server)

master mysql host:

```
[root@master ~]# vim /etc/my.cnf
//Add the following:
server-id=1
log-bin=mysql-bin
binlog_format=mixed
log-bin-index=mysql-bin.index
rpl_semi_sync_master_enabled=1
rpl_semi_sync_master_timeout=10000
rpl_semi_sync_slave_enabled=1
relay_log_purge=0
relay-log=relay-bin
relay-log-index=slave-relay-bin.index
```
* RPL ﹣ semi ﹣ sync ﹣ master ﹣ timeout = 10000: millisecond unit. After the main server waits for the confirmation message for 10 seconds, it will not wait any more and will change to asynchronous mode.

Candidate master host:

```
[root@candicatemaster ~]# vim /etc/my.cnf 
server-id=2
log-bin=mysql-bin
binlog_format=mixed
log-bin-index=mysql-bin.index
relay_log_purge=0
relay-log=relay-bin
relay-log-index=slave-relay-bin.index
rpl_semi_sync_master_enabled=1
rpl_semi_sync_master_timeout=10000
rpl_semi_sync_slave_enabled=1
```

* Note: delay ﹣ log ﹣ purge = 0, it is forbidden for sql thread to automatically delete a relay log after executing it. In MHA scenario, for some delayed recovery from the database, it depends on the relay logs of other slave databases, so disable the automatic deletion function.

slave host:
```
[root@slave ~]# vim /etc/my.cnf 
server-id=3
log-bin=mysql-bin
relay-log=relay-bin
relay-log-index=slave-relay-bin.index
read_only=1
rpl_semi_sync_slave_enabled=1
```
--> Restart mysql service (Master, candidate master, slave): systemctl restart mysqld