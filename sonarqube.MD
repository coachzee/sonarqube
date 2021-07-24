# SonarQube Installation 

SonarQube provides the capability to not only show health of an application but also to highlight issues newly introduced. With a Quality Gate in place, you can fix the leak and therefore improve code quality systematically.


# Prerequisites
1. Java
2. Database ( MySQL is Deprecated so we are going to focus on PostgresSQL)

Increase the vm.max_map_count kernal ,file discriptor and ulimit for current session at runtime.
```sh
# sysctl -w vm.max_map_count=524288
# sysctl -w fs.file-max=131072
# ulimit -n 131072
# ulimit -u 8192
```
```sh
vim  /etc/sysctl.conf  # Add Below To The End of The File
```
```sh
fs.file-max = 2097152
vm.max_map_count = 262144
vm.swappiness = 1
```
```sh
vim /etc/security/limits.conf  # Add Below To The End of The File
```
```sh
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited
elasticsearch - nofile 65536
elasticsearch - nproc 4096
root soft memlock unlimited
root hard memlock unlimited
root - nofile 65536
root - nproc 4096
sonar - nofile 131072
sonar - nproc 8192
sonarqube - nofile 65536
sonarqube - nproc 4096
```

# Installation
Step 1: Install OpenJDK
```sh
$ sudo yum install java-11-openjdk-devel
$ sudo java -version
```

# Step 2: Install PostgreSQL
```sh
$sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm -y 
```
```sh
$ sudo yum install postgresql11-server postgresql11 -y 
$ sudo /usr/pgsql-11/bin/postgresql-11-setup initdb
$ sudo vim /var/lib/pgsql/11/data/pg_hba.conf
```
Find below lines at the bottom of the file and change peer to trust and ident to md5 on local, ipv4 and ipv6 connections
```sh
peer  change to trust
ident change to md5

##Change this

local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident

##To this:

local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```
Start PostgreSQL Database server
```sh
$ sudo systemctl start postgresql-11
```
Enable it to start automatically at System Startup
```sh
$ sudo systemctl enable postgresql-11
```
Change the password for the default PostgreSQL user.
```sh
$ sudo passwd postgres
```
Switch to the postgres user.
```sh
su - postgres
```
Create a new user by typing:
```sh
createuser sonar
```
Switch to the PostgreSQL shell.
```sh
psql
```
Set a password for the newly created user for SonarQube database.
```sh
ALTER USER sonar WITH ENCRYPTED password 'sonar';
```
Create a new database for PostgreSQL database by running:
```sh
CREATE DATABASE sonarqube OWNER sonar;
```
Exit from the psql shell:
```sh
\q
```
Switch back to the sudo user by running the exit command.
```sh
exit
```

### Now That DB Setup Is Completed, Let's Move On To SONARQUBE Installation & Configuration 

# Download stable SonarQube version from below website. 
- Website: https://www.sonarqube.org/downloads/
- Note: This Article written for SonarQube version

Download & unzip SonarQube Latest Version
```sh
# cd /opt
# wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.1.0.31237.zip
# unzip sonarqube-8.1.0.31237.zip
# mv /opt/sonarqube-8.1.0.31237 /opt/sonarqube
```
Create new user give ownership to /opt/sonarqube directory 
```sh
 # useradd sonar
 # passwd sonar
 # chown -R sonar:sonar /opt/sonarqube
```
SonarQube Server Instance
Edit sonar properties file to uncomment and provide required information for below properties. 
```sh
vim /opt/sonarqube/conf/sonar.properties
```

```sh
- File Name: /opt/sonarqube/conf/sonar.properties
  - sonar.jdbc.username=`sonar`
  - sonar.jdbc.password=`sonar`
  - sonar.jdbc.url=jdbc:postgresql://localhost/sonar
```
Switch to sonar user to start SonarQube service : Edit the sonar script file and set RUN_AS_USER
```sh
su - sonar 
sudo vim  /opt/sonarqube/bin/linux-x86-64/sonar.sh

$RUN_AS_USER="sonar"
```
Execute sonar.sh as service account "sonar user"
```sh
# su - sonar
# cd /opt/sonarqube/bin/linux-x86-64/
# ./sonar.sh start
```

### Configure Systemd service
Create a systemd service file for SonarQube to run as System Startup.

Add the below lines,
```sh
vim /etc/systemd/system/sonarqube.service
```
```sh
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=131072
LimitNPROC=8192

[Install]
WantedBy=multi-user.target
```
Save and close the above file. 

### Reload the daemon
```sh
# systemctl daemon-reload
```

Stop existing sonarqube run that was trigger by sonar and login as "root" to rerun sonar as a server . Start the Sonarqube daemon by running:

```sh
# sudo systemctl start sonarqube.service
```
Enable the SonarQube service to automatically  at boot time System Startup.
```sh
# sudo systemctl enable sonar
```
Check if the sonarqube service is running,
```sh
# sudo systemctl status sonar
```
SonarQube application uses port 9000. access SonarQube from browser
```sh
  http://<YourIP OR Hostname>:9000
```
```sh
username: admin
password: admin
```

Troubleshooting 
1. Check to see Firewall is open for port 9000
