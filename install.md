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

## Prepare Local Repo using Lighttpd
Reference
- https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/setting_up_a_local_repository_with_no_internet_access.html

Install lighttpd
```
yum -y install lighttpd
systemctl enable lighttpd
systemctl start lighttpd
```

Edit lighttpd.conf
```
vi /etc/lighttpd/lighttpd.conf
```

Disable IPV6
```
server.use-ipv6 = "disable"
```

Enable directory listing
```
vi /etc/lighttpd/conf.d/dirlisting.conf
```

Restart lighttpd
```
systemctl restart lighttpd
```

Copy and extract the ambari tarball under /var/www/lighttpd/

## Create local repo

1. Copy the extracted tarball under the document root of lighttpd
2. run createrepo command at the base directory
```
createrepo .
```

## Download remaining RPMs for local install

Create local directory
```
mkdir container-selinux  device-mapper-persistent-data  docker-ce  lvm2  mariadb-server  slapd
```

Using yum to download only
```
yum install --downloadonly --downloaddir=mariadb-server mariadb-server
yum install --downloadonly --downloaddir=slapd openldap openldap-clients openldap-servers
yum install --downloadonly --downloaddir=device-mapper-persistent-data device-mapper-persistent-data
yum install --downloadonly --downloaddir=lvm2 lvm2
yum install --downloadonly --downloaddir=container-selinux http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.95-2.el7_6.noarch.rpm
yum install --downloadonly --downloaddir=docker-ce docker-ce
```

## Install from local repo

Install JDK
```
rpm -i jdk-8u211-linux-x64.rpm
```

Copy JCE
```
cp *.jar /usr/java/latest/jre/lib/security
```

## Install remaining RPM 

Install later using yum localinstall
```
yum --disablerepo=* localinstall *.rpm
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

# Dataplane Platform Installation

Reference
- https://docs.hortonworks.com/HDPDocuments/DP/DP-1.2.2/installation/content/dp_prepare_your_clusters.html

## Setup LDAP Authentication for Ambari
Reference
- https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/ambari-authentication-ldap-ad/content/authe_ldapad_configuring_ambari_for_ldap_or_active_directory_authentication.html

On ambari server host, run the setup command
```
ambari-server setup-ldap
```

Restart ambari
```
ambari-server restart
```

Sync users from ldap
```
ambari-server sync-ldap --all
```

Check if user sync is success
- Logon using the ldap user
- Logon using ambari admin user then go to "Manage Ambari" from the top right corner, then open "Users" on the left side menu

## Setup Knox SSO Overview

Reference
- https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/configuring-knox-sso/content/knox_sso.html
- https://developer.ibm.com/hadoop/2016/08/03/ldap-integration-with-apache-knox/
- https://www.ibm.com/support/knowledgecenter/en/SSPT3X_4.2.5/com.ibm.swg.im.infosphere.biginsights.admin.doc/doc/admin_sso_knox.html
- https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/configuring-knox-sso/content/sec_configuring_ldapad_identity_provider_idp.html

To set up Knox SSO with LDAP/AD, complete the following workflow:
1. Install Knox.
2. Configure Ambari Authentication for LDAP/AD.
3. Configure an LDAP/AD Identity Provider (IdP).
4. Enable Knox SSO using the Ambari CLI.
5. Configure Knox SSO for HDFS, Oozie, MapReduce2, Zeppelin, or YARN.
6. Restart all services that require a restart via Ambari.

## Edit Knox SSO Topology File

To edit knox sso topology file
1. Login to ambari as admin
2. Open Knox service page
3. Go to Configs tab
4. Open "Advanced knoxsso-topology"

The following parameters must be updated to match your environment:
- main.ldapRealm.userDnTemplate
- main.ldapRealm.contextFactory.url
- knoxsso.redirect.whitelist.regex

Save and restart knox

## Ambari SSO Setup
Reference
- https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/configuring-knox-sso/content/sso_set_up_knox_sso.html
- https://community.hortonworks.com/articles/114601/how-to-configure-and-troubleshoot-a-knox-topology.html

Get the public certificate PEM
```
cd /usr/hdf/current/knox-server/bin
./knoxcli.sh export-cert --type PEM
cat /usr/hdf/3.4.1.1-4/knox/data/security/keystores/gateway-identity.pem
```

Go to Ambari host, run the following command
```
ambari-server setup-sso
```

Restart ambari when finished
```
ambari-server restart
```

Try logging in to Ambari, it will redirecto to Knox SSO page.

## Setup Docker
Reference
- https://docs.hortonworks.com/HDPDocuments/DP/DP-1.2.2/installation/content/dp_installation_prerequisites.html

Disable SE Linux
```
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/sysconfig/selinux
```

Install required packages
```
yum install yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.95-2.el7_6.noarch.rpm
yum install docker-ce
systemctl start docker
```

## Setup DP-Core
Add Public Repo URL
```
cd /etc/yum.repos.d
wget xxx
```

Install dpcore
```
yum install dp-core
```

Initialize DP
Reference
- https://docs.hortonworks.com/HDPDocuments/DP/DP-1.2.2/installation/content/dp_initialize.html

Run system check
```
cd /usr/dp/current/core/bin
./dpdeploy.sh system-check
```
GCE somehow use port 80, so modify the APP port in config.env.sh. Change to something else like 8080

Initialize DP
```
./dpdeploy.sh init --all
```

## Initialize LDAP integration
1. Access DP web through browser https://<dp address>, login using admin account
2. Configure LDAP
3. Login again using LDAP user

## Configure Knox SSO for DP
Reference
- https://docs.hortonworks.com/HDPDocuments/DP/DP-1.2.2/installation/content/dp_configure_sso_with_hdp_clusters.html

Logon to DP host
```
cd /usr/dp/current/core/bin/certs/
cat ssl-cert.pem
```

Logon to knox host. Create token.xml
```
vi /etc/knox/conf/topologies/token.xml

<?xml version="1.0" encoding="UTF-8"?>
<topology>
   <uri>https://$KNOX_HOSTNAME_FQDN:8443/gateway/token</uri>
   <name>token</name>
   <gateway>
      <provider>
         <role>federation</role>
         <name>SSOCookieProvider</name>
         <enabled>true</enabled>
         <param>
            <name>sso.authentication.provider.url</name>
            <value>https://bcx-1.gce.cloudera.com:8443/gateway/knoxsso/api/v1/websso</value>
         </param>
         <param>
            <name>sso.token.verification.pem</name>
            <value>
				PASTE SSL-CERT.PEM HERE
            </value>
         </param>
      </provider>
      <provider>
         <role>identity-assertion</role>
         <name>HadoopGroupProvider</name>
         <enabled>true</enabled>
      </provider>
      
   </gateway>

   <service>
      <role>KNOXTOKEN</role>
      <param>
         <name>knox.token.ttl</name>
         <value>100000</value>
      </param>
      <param>
         <name>knox.token.client.data</name>
         <value>cookie.name=hadoop-jwt</value>
      </param>
      <param>
         <name>main.ldapRealm.authorizationEnabled</name>
         <value>false</value>
      </param>
   </service>
</topology>
```

Create redirect.xml
```
vi /etc/knox/conf/topologies/redirect.xml

<topology>
    <name>tokensso</name>
    <gateway>
        <provider>
            <role>federation</role>
            <name>JWTProvider</name>
            <enabled>true</enabled>
        </provider>
        <provider>
            <role>identity-assertion</role>
            <name>Default</name>
            <enabled>true</enabled>
        </provider>
    </gateway>
    <service>
        <role>KNOXSSO</role>
        <param>
            <name>knoxsso.cookie.secure.only</name>
            <value>true</value>
        </param>
        <param>
            <name>knoxsso.token.ttl</name>
            <value>600000</value>
        </param>
        <param>
            <name>knoxsso.redirect.whitelist.regex</name>
            <value>.*</value>
        </param>
    </service>
</topology> 
```

Create redirecttoken.xml
```
cp /etc/knox/conf/topologies/token.xml /etc/knox/conf/topologies/redirecttoken.xml
```

Create ui.xml
```
vi /etc/knox/conf/topologies/ui.xml

<topology>
    <gateway>
        <provider>
            <role>authentication</role>
            <name>Anonymous</name>
            <enabled>true</enabled>
        </provider>
        <provider>
            <role>identity-assertion</role>
            <name>Default</name>
            <enabled>false</enabled>
        </provider>
    </gateway>
    <service>
        <role>AMBARI</role>
        <url>http://bcx-1.gce.cloudera.com:8080</url>
    </service>
    <service>
        <role>AMBARIUI</role>
        <url>http://bcx-1.gce.cloudera.com:8080</url>
    </service>
</topology>
```

# Install SMM

## Install SMM App on DP host
Reference
- https://docs.hortonworks.com/HDPDocuments/SMM/SMM-1.2.1/installation/content/install_the_service.html

1. Configure local repo
2. Install SMM App
3. Install SMM REST admin server

Install SMM App on the DP host
```
yum install smm-app
```

Initialize SMM App
```
cd /usr/smm-app/current/streams-messaging-manager/bin
./smmdeploy.sh init
./smmdeploy.sh ps
```

## Install SMM Rest Admin Server
Reference
- https://docs.hortonworks.com/HDPDocuments/SMM/SMM-1.2.1/installation/content/smm-install-rest-server.html

1. Configure SMM Database
2. Install SMM Management Pack
3. Update SMM Base URL
4. Add SMM REST Server as a service
5. Configure Knox for SMM Integration

Reference
- https://docs.hortonworks.com/HDPDocuments/SMM/SMM-1.2.1/installation/content/smm-configure-database.html

Setup SMM DB
```
mysql -u root

create database streamsmsgmgr;
CREATE USER 'streamsmsgmgr'@'localhost' IDENTIFIED BY 'cloudera123';
GRANT ALL PRIVILEGES ON streamsmsgmgr.* TO 'streamsmsgmgr'@'localhost' WITH GRANT OPTION;
CREATE USER 'streamsmsgmgr'@'%' IDENTIFIED BY 'cloudera123';
GRANT ALL PRIVILEGES ON streamsmsgmgr.* TO 'streamsmsgmgr'@'%' WITH GRANT OPTION;
```

Backup ambari resource folder
```
cp -r /var/lib/ambari-server/resources /var/lib/ambari-server/resources.backup
```

Install mpack and restart ambari afterwards
```
ambari-server install-mpack --mpack=smm-ambari-mpack-1.2.1.0-10.tar.gz --verbose
ambari-server restart
```

1. Logon to Ambari
2. Add SMM service

## Configure Knox for SMM integration

1. Update authentication.provider.url at Advanced SMM SSO Config
2. Generate public.key.pem

```
/usr/jdk64/jdk1.8.0_112/bin/keytool \
-export \
-alias gateway-identity \
-rfc \
-file /root/knox-sso-cert.pem -keystore /usr/hdf/current/knox-server/data/security/keystores/gateway.jks
```

Paste the key to public.key.pem at Advanced SMM SSO Config

## Assign Users the kafka DevOps Role
Reference
- https://docs.hortonworks.com/HDPDocuments/SMM/SMM-1.2.1/installation/content/smm-users-roles.html

## Enable SMM service in target cluster
Reference
- https://docs.hortonworks.com/HDPDocuments/SMM/SMM-1.2.1/installation/content/smm-enable-service-in-dp.html

To fix an internal server error when opening up SMM, run the following on the DP host
```
./dpdeploy.sh utils add-host 172.31.113.25 bcx-3.gce.cloudera.com
```

### SRM Notes
1. /etc/streams-replication-manager/srm.properties is not created by default
2. mm2-offsets.xxx.internal is created using replication factor = 3 by default even though default.replication.factor is set to 1 for that cluster. Might cause issue in a single node cluster
- org.apache.kafka.connect.errors.ConnectException: Error while attempting to create/find topic(s) 'mm2-offsets.vpc.internal'
Caused by: java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.InvalidReplicationFactorException: Replication factor: 3 larger than available brokers: 1.
Caused by: org.apache.kafka.common.errors.InvalidReplicationFactorException: Replication factor: 3 larger than available brokers: 1.
3. For SRM control to work, Requires kafka producer library to be installed on the SRM node (not mentioned in the documentation)
org.apache.kafka.common.KafkaException: Failed to construct kafka producer
	at org.apache.kafka.clients.producer.KafkaProducer.<init>(KafkaProducer.java:430)
	at org.apache.kafka.clients.producer.KafkaProducer.<init>(KafkaProducer.java:287)


















