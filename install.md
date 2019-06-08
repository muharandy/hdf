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

## Check DNS or edit /etc/hosts
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
create database superset DEFAULT CHARACTER SET utf8;;
grant all privileges on superset.* to 'superset'@'localhost' identified by 'cloudera123';
grant all privileges on superset.* to 'superset'@'%.gce.cloudera.com' identified by 'cloudera123';
create database druid DEFAULT CHARACTER SET utf8;;
grant all privileges on druid.* to 'druid'@'localhost' identified by 'cloudera123';
grant all privileges on druid.* to 'druid'@'%.gce.cloudera.com' identified by 'cloudera123';
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

## Prepare passwordless SSH

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

## Setup Prerequisite

Enable NTPD
```
yum install -y ntp
systemctl enable ntpd
```
Check DNS or edit /etc/hosts
Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/check_dns.html

Configure IPTABLES
```
systemctl disable firewalld
service firewalld stop
```

Disable SELinux
```
setenforce 0
```

## Install HDF mpack
Reference
- https://docs.hortonworks.com/HDPDocuments/HDF3/HDF-3.4.0/installing-hdf/content/installing_the_hdf_management_pack_on_an_hdf_cluster.html

Get the mpack
```
wget http://public-repo-1.hortonworks.com/HDF/centos7/3.x/updates/3.4.0.0/tars/hdf_ambari_mp/hdf-ambari-mpack-3.4.0.0-155.tar.gz
ambari-server install-mpack \
--mpack=hdf-ambari-mpack-3.4.0.0-155.tar.gz \
--purge \
--verbose
```

Restart ambari
```
ambari-server restart
```

## Install HDF Cluster
Reference
- https://docs.hortonworks.com/HDPDocuments/HDF3/HDF-3.4.0/installing-hdf/content/install_an_hdf_cluster_using_ambari.html

# Install LDAP

Reference
- https://www.itzgeek.com/how-tos/linux/centos-how-tos/step-step-openldap-server-configuration-centos-7-rhel-7.html
- https://www.thegeekstuff.com/2015/01/openldap-linux/

## Install SLAPD
```
yum install -y openldap openldap-clients openldap-servers
systemctl start slapd
systemctl enable slapd
```

## Configure Base Directory

Generate password
```
slappasswd
```

Create db.ldif to modify the LDAP config. Fill olcRootPW with the encrypted hash password generated by slappasswd command

```
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=gce,dc=cloudera,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldapadm,dc=gce,dc=cloudera,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}8GhANEreEDN7M3ZiFCiSHUl/QP+I04/1
```

Update the config file using ldapmodify
```
ldapmodify -Y EXTERNAL  -H ldapi:/// -f db.ldif
```

Copy sample database config to /var/lib/ldap
```
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown ldap:ldap /var/lib/ldap/*
```

Add cosine and nis LDAP schemas
```
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```

Create base.ldif to setup the base directory structure
```
dn: dc=gce,dc=cloudera,dc=com
dc: gce
objectClass: top
objectClass: domain

dn: cn=ldapadm,dc=gce,dc=cloudera,dc=com
objectClass: organizationalRole
cn: ldapadm
description: LDAP Manager

dn: ou=people,dc=gce,dc=cloudera,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=group,dc=gce,dc=cloudera,dc=com
objectClass: organizationalUnit
ou: Group
```

Build the directory structure
```
ldapadd -x -W -D "cn=ldapadm,dc=gce,dc=cloudera,dc=com" -f base.ldif
```

## Add User and Group

Add user by creating user.ldif
```
dn: uid=fajar,ou=people,dc=gce,dc=cloudera,dc=com
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: fajar
uid: fajar
uidNumber: 9999
gidNumber: 100
homeDirectory: /home/fajar
loginShell: /bin/bash
gecos: Fajar
userPassword: {crypt}x
shadowLastChange: 0
shadowMax: 0
shadowWarning: 0
```

Add using ldapadd
```
ldapadd -x -W -D "cn=ldapadm,dc=gce,dc=cloudera,dc=com" -f user.ldif
```

Set password to user
```
ldappasswd -s cloudera123 -W -D "cn=ldapadm,dc=gce,dc=cloudera,dc=com" -x "uid=fajar,ou=people,dc=gce,dc=cloudera,dc=com"
```

Create new group using hdfadm_group.ldif
```
dn: cn=hdfadm,ou=group,dc=gce,dc=cloudera,dc=com
objectClass: top
objectClass: posixGroup
gidNumber: 100
```

Add the group using ldapadd
```
ldapadd -x -W -D "cn=ldapadm,dc=gce,dc=cloudera,dc=com" -f hdfadm_group.ldif
```

Create user to the group we just created using addtogroup.ldif
```
dn: cn=hdfadm,ou=group,dc=gce,dc=cloudera,dc=com
changetype: modify
add: memberuid
memberuid: fajar
```

Add using ldapmodify
```
ldapmodify -x -W -D "cn=ldapadm,dc=gce,dc=cloudera,dc=com" -f addtogroup.ldif
```








