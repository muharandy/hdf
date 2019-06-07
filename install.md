# Ambari Install

## Setup Passwordless SSH
Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/set_up_password-less_ssh.html

Generate public key and private key
```
ssh-keygen
```

## Enable NTPD
Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/enable_ntp_on_the_cluster_and_on_the_browser_host.html
```
yum install -y ntp
systemctl enable ntpd
```

# Check DNS or edit /etc/hosts
Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/check_dns.html

## Configure IPTABLES
Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/configuring_iptables.html
```
systemctl disable firewalld
service firewalld stop
```

## Disable SELinux
Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/disable_selinux_and_packagekit_and_check_the_umask_value.html
```
setenforce 0
```

## Install MySQL
Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/administering-ambari/content/amb_example_install_mysql-mariadb_for_multiple_components.html

Install MariaDB
```
yum install mariadb-server -y
systemctl start mariadb
systemctl enable mariadb
```

Secure Installation
```
/usr/bin/mysql_secure_installation
```

Create Database and Schema for Ambari
```
mysql -u root

CREATE USER 'ambari'@'%' IDENTIFIED BY 'cloudera123';
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';
CREATE USER 'ambari'@'localhost' IDENTIFIED BY 'cloudera123';
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'localhost';
CREATE USER 'ambari'@'bcx-ambari.gce.cloudera.com' IDENTIFIED BY 'cloudera123';
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'bcx-ambari.gce.cloudera.com';
FLUSH PRIVILEGES;
```

Create Database for other services
```
mysql -u root

create database hive;
grant all privileges on hive.* to 'hive'@'localhost' identified by 'cloudera123';
grant all privileges on hive.* to 'hive'@'%.gce.cloudera.com' identified by 'cloudera123';
create database ranger;
grant all privileges on ranger.* to 'ranger'@'localhost' identified by 'cloudera123';
grant all privileges on ranger.* to 'ranger'@'%.gce.cloudera.com' identified by 'cloudera123';
create database rangerkms;
grant all privileges on rangerkms.* to rangerkms@'localhost' identified by 'cloudera123';
grant all privileges on rangerkms.* to rangerkms@'%.gce.cloudera.com' identified by 'cloudera123';
create database oozie;
grant all privileges on oozie.* to 'oozie'@'localhost' identified by 'cloudera123';
grant all privileges on oozie.* to 'oozie'@'%.gce.cloudera.com' identified by 'cloudera123';
```

Create Database for SAM and Schema Registry
```
mysql -u root -p

create database registry;
create database streamline;

CREATE USER 'registry'@'%' IDENTIFIED BY 'cloudera123';
CREATE USER 'streamline'@'%' IDENTIFIED BY 'cloudera123';

GRANT ALL PRIVILEGES ON registry.* TO 'registry'@'%' WITH GRANT OPTION ;
GRANT ALL PRIVILEGES ON streamline.* TO 'streamline'@'%' WITH GRANT OPTION ;
```

## Install Ambari from Public Repo

Download repo
```
wget -nv http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.3.0/ambari.repo -O /etc/yum.repos.d/ambari.repo
```

Install Ambari Server
```
yum install ambari-server
```

Download JDBC Connector
```
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.45.tar.gz
tar xzvf mysql-connector-java-5.1.45.tar.gz

mkdir /usr/share/java
cp mysql-connector-java-5.1.45/mysql-connector-java-5.1.45-bin.jar /usr/share/java/mysql-connector-java.jar
```

Copy the connector
```
ambari-server setup --jdbc-db=mysql --jdbc-driver=/path/to/mysql/mysql-connector-java.jar
```

Confirm that .jar is in the Java share directory.
```
ls /usr/share/java/mysql-connector-java.jar
```

Load the Ambari Server database schema
```
cd /var/lib/ambari-server/resources/
mysql -u ambari -p
CREATE DATABASE ambari;
USE ambari;
SOURCE Ambari-DDL-MySQL-CREATE.sql;
```

Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/administering-ambari/content/amb_using_ambari_with_mysql_or_mariadb.html

Run the setup
```
ambari-server setup
```
Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/set_up_the_ambari_server.html

Start the ambari server
```
ambari-server start
```

# Setup hosts for Kafka and NiFi

Copy public key from ambari server to the target hosts
```
scp .ssh/id_rsa.pub root@bcx-kafka01.gce.cloudera.com:/root/.ssh
```

Login to the target host and add the SSH public key to the authorized_keys
```
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
```


