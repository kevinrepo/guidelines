# This part is the installation of MariaDB GaleraCluster setup in CentOS (non-Docker version)

----
#### (You can refer to original description on links at bottom) 


## Install MariaDB

### 1. Download and install by execute

        $ sudo yum install mariadb-server

### 2. Test start the service. Read status

        $ sudo systemctl start mariadb

        $ sudo systemctl status mariadb

    * If Active: active (running) show. Then it is installed fine.


### 3. Securing MariaDB - To remove anonymous users, disallow remote root login, remove the test database, and reload the privilege tables.

        $ sudo mysql_secure_installation

    * Follow the recommandation or press Y is ok

&nbsp;

## Setup Galera Cluster

* Assuming Already setup 2 more MariaDB on other servers.

* All 3 servers have ssh access to each others.

### 1. All we need is to add a new config for each server.
    a. locate the directory: */etc/mysql/conf.d*

    b. add a new file with either nano or vim.

        sudo nano /etc/mysql/conf.d/galera.cnf
        sudo vi /etc/mysql/conf.d/galera.cnf

    c. add the following content:

>/etc/mysql/conf.d/galera.cnf

```vim {.line-numbers}
    [mysqld]
    binlog_format=ROW
    default-storage-engine=innodb
    innodb_autoinc_lock_mode=2
    bind-address=0.0.0.0

    # Galera Provider Configuration
    wsrep_on=ON
    wsrep_provider=/usr/lib/galera/libgalera_smm.so

    # Galera Cluster Configuration
    wsrep_cluster_name="test_cluster"
    wsrep_cluster_address="gcomm://first_ip,second_ip,third_ip"

    # Galera Synchronization Configuration
    wsrep_sst_method=rsync

    # Galera Node Configuration
    wsrep_node_address="this_node_ip"
    wsrep_node_name="this_node_name"
```

* Please notice the following lines need to write info for different server:

        wsrep_cluster_address="gcomm://first_ip,second_ip,third_ip"

    *(all three ip addresses)*

        # Galera Node Configuration
        wsrep_node_address="this_node_ip"
        wsrep_node_name="this_node_name"

    *(ip & node name for each server)*

### 2. Stop all servers for rerun  
        $ sudo systemctl stop mysql

### 3. Start the first server/node:
        $ sudo galera_new_cluster

### 4. Check successful starting first node:
        $ mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"

        * output
        +--------------------+-------+
        | Variable_name      | Value |
        +--------------------+-------+
        | wsrep_cluster_size | 1     |
        +--------------------+-------+

### 5. Bring up 2nd & 3rd Node:
        $ sudo systemctl start mysql

*(we only need to start the usual way)*

### If all starting success. We will see the following output with command:
         $ mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"

        * output
        +--------------------+-------+
        | Variable_name      | Value |
        +--------------------+-------+
        | wsrep_cluster_size | 3     |
        +--------------------+-------+


&nbsp;
&nbsp;
&nbsp;

# This part is the installation of MariaDB GaleraCluster setup (Docker version)

----
#### (You can refer to original description on links at bottom) 


## Noticable difference  

### 1. /etc/mysql/conf.d/galera.cnf

* addition of below lines:
        
        innodb_locks_unsafe_for_binlog=1
        query_cache_size=0
        query_cache_type=0

* change from ip address to the following variables
        
        wsrep_cluster_address="gcomm://mariadb-node-0,mariadb-node-1,mariadb-node-2"

* remove of the following lines

        # Galera Node Configuration
        wsrep_node_address="this_node_ip"
        wsrep_node_name="this_node_name"

* optionally add below code

        innodb_flush_log_at_trx_commit=0

### 2. create path to store the MariaDB data

* run & $ mkdir -p /mnt/data

### 3. start first cluster with commmand

        $ docker run \
        --name mariadb-0 \
        -d \
        -v /opt/local/etc/mysql.conf.d:/etc/mysql/conf.d \
        -v /mnt/data:/var/lib/mysql \
        -e MYSQL_INITDB_SKIP_TZINFO=yes \
        -e MYSQL_ROOT_PASSWORD=my-secret-pw \
        -p 3306:3306 \
        -p 4567:4567/udp \
        -p 4567-4568:4567-4568 \
        -p 4444:4444 \
        mariadb:10.1 \
        --wsrep-new-cluster \
        --wsrep_node_address=$(ip -4 addr ls eth0 | awk '/inet / {print $2}' | cut -d"/" -f1)

### 4. after pull image, check the log to make sure MariaDB is fully initialized and ready to accept repicas

        $ docker logs -f mariadb-0

### 5. add nodes

        docker run \
        --name mariadb-1 \
        -d \
        -v /opt/local/etc/mysql.conf.d:/etc/mysql/conf.d \
        -v /mnt/data:/var/lib/mysql \
        -p 3306:3306 \
        -p 4567:4567/udp \
        -p 4567-4568:4567-4568 \
        -p 4444:4444 \
        mariadb:10.1 \
        --wsrep_node_address=$(ip -4 addr ls eth0 | awk '/inet / {print $2}' | cut -d"/" -f1)


### 6. Using Cluster in docker

        # The default password in the sample above is "my-secret-pw"
        $ export MYSQL_IP=$(ip -4 addr ls eth0 | awk '/inet / {print $2}' | cut -d"/" -f1)
        $ docker run --rm -it mariadb:10.1 mysql -h $MYSQL_IP -u root -p




### Links/References 
* [Install MariaDB on CentOS7](https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-centos-7)

* [Install Galera cluster, MariaDB, CoreOS and Docker (Part 1)](https://withblue.ink/2016/03/09/galera-cluster-mariadb-coreos-and-docker-part-1.html)
