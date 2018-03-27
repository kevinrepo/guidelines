# This is the installation of MariaDB GaleraCluster setup in CentOS (non-Docker version)

----
#### (You can refer to detail description on links at bottom) 


## Install MariaDB

1. Download and install by execute

        $ sudo yum install mariadb-server

2. Test start the service. Read status

        $ sudo systemctl start mariadb

        $ sudo systemctl status mariadb

    * If Active: active (running) show. Then it is installed fine.


3. Securing MariaDB - To remove anonymous users, disallow remote root login, remove the test database, and reload the privilege tables.

        $ sudo mysql_secure_installation

    * Follow the recommandation or press Y is ok


## Setup Galera Cluster

* Assuming Already setup 2 more MariaDB on other servers.

* All 3 servers have ssh access to each others.

1. All we need is to add a new config for each server.
    a. local the directory: */etc/mysql/conf.d*

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




refence [MariaDB on CentOS7](https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-centos-7)



## what is Markdown?
see [Wikipedia](http://en.wikipedia.org/wiki/Markdown)

> Markdown is a lightweight markup language, originally created by John Gruber and Aaron Swartz allowing people "to write using an easy-to-read, easy-to-write plain text format, then convert it to structurally valid XHTML (or HTML)".

----
## usage
1. Write markdown text in this textarea.
2. Click 'HTML Preview' button.

----
## markdown quick reference
# headers

*emphasis*

**strong**

* list

>block quote

    code (4 spaces indent)
[links](http://wikipedia.org)

----
## changelog
* 17-Feb-2013 re-design

----
## thanks
* [markdown-js](https://github.com/evilstreak/markdown-js)