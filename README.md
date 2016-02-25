# mysqlcluster
setup mysql cluster 7.x

Setup Overview

MySQL Cluster has one or more of several major components. Typically each component will have its own server (a node). Those components are

    sql node(front-end)
    data node (back-end)
    management node (central config,meta-data and etc)

Generally speaking, application requests are sent to the SQL nodes, which then talk to the data nodes and coordinates each other via management node(s) to actually recover the content you want.
For redundancy you need at least two data nodes. 

MGM Node -  holds the configuration for MySQL Cluster in a file called config.ini. It also writes a cluster log, and takes part in arbitration to prevent split-brain or network partitioning. 
You are recommended to have two MGM Nodes for redundancy. Since MGM hardly use any resources, and is not involved in query processing or routing of requests, it can be colocated with the SQL Nodes/Access Nodes.

SQL Node / Access Node - the most common way to read/write data to the Data Nodes is by using the MySQL Server (aka SQL Node). A query comes into the SQL Node. The SQL Node parses, optimizes and executes the query on the Data Nodes.You need at least two SQL Nodes for redundancy. Nodes in the access layer should have fast CPUs, disks are not important, fast network to the data nodes (same specs as for the data nodes), and about 2+ GB of RAM (depending on the number of connections, per thread buffers etc). Do not co-locate the SQL Nodes/Access Nodes with the Data Nodes.

Data nodes usually operate in pairs, so the number of data nodes would be two, four, six, etc. The data nodes should have many cores, fast cores, enough RAM to store your data, and fast disks.Distributing Data Nodes that belong to one cluster over WAN is generally not recommended, unless the WAN connection is very stable and the latency is very low (at most a few milliseconds).  

Note:- All cluster nodes are using centos 6.7 x64bit

Node Configuration
==================

management node = 10.10.10.10
sql-node1= 10.10.10.20
sql-node2= 10.10.10.21
data-node1= 10.10.10.30
data-node2 = 10.10.10.31


              apache/php webserver
             __________|_________
            |                    |
   Mgmt|Data|SQLnode       Data | SQLnode

   Let’s begin by disabling SELinux on all nodes. 

   On management node:-

    
   iptables -A INPUT -p tcp --dport 1186 -j ACCEPT

   Download the MySQL cluster packages:-

   wget https://dev.mysql.com/get/Downloads/MySQL-Cluster-7.3/MySQL-Cluster-gpl-7.3.12-1.el6.x86_64.rpm-bundle.tar  [New GA release might be available]

   [Download the latest GA version]

   tar xvf MySQL*.tar

   Install MySQL cluster server package:-
   yum install libaio numactl perl-Class-MethodMaker rsync wget vim openssh-server -y
   yum remove -y mysql-libs

   rpm -Uvh MySQL-Cluster-server-gpl-7.3.6-2.el6.x86_64.rpm

# create config.ini for initiating cluster nodes
# Recommeded way to Configure and deploy production-class database clusters for MySQL
# http://severalnines.com/cluster-configurator/

# vim config.ini

[tcp default]
SendBufferMemory=128M
ReceiveBufferMemory=128M

[ndb_mgmd]
NodeId=10
hostname=100.126.1.101
datadir=/var/lib/mycluster

[ndbd default]
NoOfReplicas=2
LockPagesInMainMemory=1
DataMemory=10165M
IndexMemory=1271M
ServerPort=2202
ODirect=1
#CompressedLCP=1
#CompressedBackup=1
#table related things

MaxNoOfConcurrentOperations=100000
MaxNoOfConcurrentTransactions=16384

StringMemory=25
MaxNoOfTables=4096
MaxNoOfAttributes=24756
MaxNoOfOrderedIndexes=2048
MaxNoOfUniqueHashIndexes=512
TimeBetweenLocalCheckpoints=30
### Params for BACKUP 
BackupMaxWriteSize=1M
BackupDataBufferSize=24M
BackupLogBufferSize=16M
BackupMemory=40M

FragmentLogFileSize=256M
InitFragmentLogFiles=SPARSE
NoOfFragmentLogFiles=40
RedoBuffer=64M

TransactionBufferMemory=8M

TimeBetweenGlobalCheckpoints=1000
TimeBetweenEpochs=100

TimeBetweenEpochsTimeout=0

### Watchdog 
TimeBetweenWatchdogCheckInitial=60000
### TransactionInactiveTimeout  - should be enabled in Production 
TransactionInactiveTimeout=60000
### New 7.1.10 redo logging parameters 
RedoOverCommitCounter=3
RedoOverCommitLimit=20

### DISK DATA 
SharedGlobalMemory=20M
DiskPageBufferMemory=64M
BatchSizePerLocalScan=512
[ndbd]
NodeId=20
hostname=202.94.66.36
datadir=/var/lib/mycluster-data
#BackupDataDir=/dir/to/backup

[ndbd]
NodeId=30
hostname=202.94.66.49
datadir=/var/lib/mycluster-data
#BackupDataDir=/dir/to/backup

[mysqld]
NodeId=40
hostname=202.94.66.44

[mysqld]
NodeId=50
"config.ini" 90L, 1510C written
[root@mgmtsql ~]# cat config.ini
[tcp default]
SendBufferMemory=128M
ReceiveBufferMemory=128M

[ndb_mgmd]
NodeId=10
hostname=100.126.1.101
datadir=/var/lib/mycluster

[ndbd default]
NoOfReplicas=2
LockPagesInMainMemory=1
DataMemory=10165M
IndexMemory=1271M
ServerPort=2202
ODirect=1
#CompressedLCP=1
#CompressedBackup=1
#table related things

MaxNoOfConcurrentOperations=100000
MaxNoOfConcurrentTransactions=16384

StringMemory=25
MaxNoOfTables=4096
MaxNoOfAttributes=24756
MaxNoOfOrderedIndexes=2048
MaxNoOfUniqueHashIndexes=512
TimeBetweenLocalCheckpoints=30
### Params for BACKUP 
BackupMaxWriteSize=1M
BackupDataBufferSize=24M
BackupLogBufferSize=16M
BackupMemory=40M

FragmentLogFileSize=256M
InitFragmentLogFiles=SPARSE
NoOfFragmentLogFiles=40
RedoBuffer=64M

TransactionBufferMemory=8M

TimeBetweenGlobalCheckpoints=1000
TimeBetweenEpochs=100

TimeBetweenEpochsTimeout=0

### Watchdog 
TimeBetweenWatchdogCheckInitial=60000
### TransactionInactiveTimeout  - should be enabled in Production 
TransactionInactiveTimeout=60000
### New 7.1.10 redo logging parameters 
RedoOverCommitCounter=3
RedoOverCommitLimit=20

### DISK DATA 
SharedGlobalMemory=20M
DiskPageBufferMemory=64M
BatchSizePerLocalScan=512
[ndbd]
NodeId=20
hostname=202.94.66.36
datadir=/var/lib/mycluster-data
#BackupDataDir=/dir/to/backup

[ndbd]
NodeId=30
hostname=202.94.66.49
datadir=/var/lib/mycluster-data
#BackupDataDir=/dir/to/backup

[mysqld]
NodeId=40
hostname=202.94.66.44

[mysqld]
NodeId=50
hostname=202.94.66.45

[mysqld]            # this is kept empty for MySQL Cluster Native Backup Tool

[mysqld]            # this is kept empty for MySQL Cluster Native Backup Tool

#empty api slots for multiple ndb cluster connection pool 
#for more info - https://dev.mysql.com/doc/mysql-cluster-excerpt/5.5/en/mysql-cluster-program-options-mysqld.html#option_mysqld_ndb-cluster-connection-pool

[api]

[api]

[api]

[api]


Configure Data Nodes[run on both 2 data nodes]
==============================================
Download and extract the Mysql cluster rpm's and install below package
  
  yum install libaio numactl -y
  yum install perl-Class-MethodMaker    #needed for running perl scripts provided by mysqlcluster
  yum remove -y mysql-libs

  rpm -Uvh MySQL-Cluster-server-gpl-7.3.6-2.el6.x86_64.rpm


vim /etc/my.cnf
[mysqld]
ndbcluster
ndb-connectstring=10.10.10.10

[mysql_cluster]
ndb-connectstring=10.10.10.10


Configure SQL Nodes[run on both of the 2 nodes]
===============================================

Download and extract the Mysql cluster rpm's and install below package
  
  yum install libaio numactl -y
  yum install perl-Class-MethodMaker    #needed for running perl scripts provided by mysqlcluster
  yum remove -y mysql-libs

  rpm -Uvh  MySQL-Cluster-server-gpl-7.3.6-2.el6.x86_64.rpm
  rpm -Uvh  MySQL-Cluster-shared-compat-gpl-7.3.6-2.el6.x86_64.rpm 
  rpm -Uvh  MySQL-Cluster-shared-gpl-7.3.6-2.el6.x86_64.rpm 
  rpm -Uvh  MySQL-Cluster-client-gpl-7.3.6-2.el6.x86_64.rpm

  vim /etc/my.cnf
  [mysqld]
ndbcluster
default-storage-engine=NDBCLUSTER
net_read_timeout=60000
connect_timeout=60000
max_allowed_packet=32M
max_connections=1000
query_cache_type=1
query_cache_size=37748736
query_cache_limit=2097152
ndb-cluster-connection-pool=2
slow-query-log=1
slow-query-log-file=/var/log/mysql/slowquery.log
long-query_time=2
log-queries-not-using-indexes
[mysql_cluster]
ndb-connectstring=10.10.10.10

run mysql with below command to change mysql root password

# mysqld_safe &

your default root password is already set during installation and is stored in /root/.mysql_secret , copy it and use in next step
run below command to change root password and have recommended settings suggested during the process

# mysql_secure_installation

After setting the root password kill the mysql process

Starting the cluster first time
===============================

#on management node [10.10.10.10]:-

ndb_mgmd --config-file /root/config.ini  --config-dir /usr/mysql-cluster --initial

#on Data nodes [10.10.10.20-21]

ndbd --initial

#on SQL nodes [10.10.10.30-31]

service mysql start


# On any one of the SQL node
# Create database on any one of the node or import existing DB

The SQL engine will need to be set to ndbcluster. If you are importing an existing database, export it using mysqldump, then perform the following commands.


vim YourDB.sql

:%s/engine=InnoDB/engine=ndbcluster/g
	
:%s/engine=MyISAM/engine=ndbcluster/g
 	

mysql -uroot -p [YourDB] < YourDB.sql

Enable remote mysql user to use YourDB database [Run on both SQL node]

mysql> GRANT ALL ON YourDB.* TO remoteuser@'202.54.10.20' IDENTIFIED BY 'PA$$WORD123';
mysql> flush privileges;








