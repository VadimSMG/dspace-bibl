# Run Ubuntu docker image (in interactive mode)
docker run -it ubuntu
# Update packages
apt update
# Install Java 
apt install openjdk-11-jdk openjdk-17-jdk maven ant postgresql postgresql-contrib wget lsof git -y
<!--Select region "Europe" timezone "Athenes"-->
java --version
## Add OpenJDK to PATH
<!-- Remove old strings in .bashrc if it exist-->
sed -i '/^export JAVA_HOME=/d' ~/.bashrc
sed -i '/^export PATH=\$PATH:\$JAVA_HOME\/bin/d' ~/.bashrc
<!-- Add new sting at the end of .bashrc-->
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> ~/.bashrc
source ~/.bashrc
## Check JAVA_HOME
echo $JAVA_HOME
# Install Maven latest
apt install maven
mvn -v
# Install Apache Ant
apt install ant
ant -v
# Install PostgresSQL
apt install postgresql postgresql-contrib
## Check service
service postgresql status
## Start server (if need)
service postgresql start
# Config PostgresSQL 
## Switch user to postgres
su - postgres
## Run PostgreSQL shell
psql
## Create new user without password
CREATE USER dspace WITH PASSWORD NULL;
## Create new database with unicode encoding
CREATE DATABASE dspace WITH ENCODING 'UTF8' OWNER dspace;
## Check database encoding
SHOW SERVER_ENCODING;
## Add full access to database
GRANT ALL PRIVILEGES ON DATABASE dspace TO dspace;
## Leave PostgreSQL shell
\q
## Return to root
logout
# Config TCP/IP for PostgresSQL
## Edit postgresql.conf
sed -i '$ a listen_addresses = '\''localhost'\''' /etc/postgresql/16/main/postgresql.conf
<!--
Adding "listen_addresses = 'localhost'" to postgresql.conf file for ver. 16. It necessary for using TCP/IP.
-->
## Edit pg_hba.conf
sed -i '$ a host    dspace dspace   127.0.0.1       255.255.255.255         md5' /etc/postgresql/16/main/pg_hba.conf
<!--
Adding "host dspace dspace 127.0.0.1 255.255.255.255 md5" to pg_hba.conf file for ver. 16. It necessary for using TCP/IP.
first "dspace" - is DB name
second "dspace" - is user and PW for DB access
-->
## Restart PostgreSQL service
service postgresql restart
# Install Apache Solr
## Apache Solr install (It necessary for indexation an search)
apt install wget <!-- If it not included in image-->
wget https://archive.apache.org/dist/lucene/solr/8.11.3/solr-8.11.3.tgz
tar -xzf solr-*.tgz
bash solr-*/bin/install_solr_service.sh solr-*.tgz
## Check user for Solr
ps -ef | grep solr
## Change open files limit
<!--Create file if it doesn`t exist`-->
touch /etc/security/limits.d/root.conf
<!--Add content to file. Set file limit to 65000-->
sed -i '$ a root soft nofile 65000' /etc/security/limits.d/root.conf
sed -i '$ a root hard nofile 65000' /etc/security/limits.d/root.conf
ulimit -n 65000
## Check Apache Solr service
<!--It may run automaticaly after instllation. If not use next command -->
service solr status
<!--If ont run in auto use: -->
service solr start
# Isntall Tomcat 9 API
## Install OpenJDK 11
apt install openjdk-11-jdk
## Add user for Tomcat
useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat
## Download latest Tomcat version
wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.2/bin/apache-tomcat-11.0.2.tar.gz
## Install Tomcat
mkdir /usr/share/tomcat
tar -xvzf apache-tomcat-*.tar.gz
mv apache-tomcat-* /usr/share/tomcat
## Create launch script for Tomcat
cat <<EOF > /etc/init.d/tomcat
#!bin/bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export JAVA_HOME
export CATALINA_HOME=/usr/share/tomcat
$CATALINA_HOME/bin/catalina.sh start
EOF
## Add permission to script
chmod +x /etc/init.d/tomcat
## Run Tomcat service
/etc/init.d/tomcat start
## Check service status
service tomcat status
## Access rules and launch at startup
chmod +x /etc/init.d/tomcat
update-rc.d tomcat defaults
## Check .bashrc
tail ~/.bashrc
<!--Right content in last strings:
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
-->
<!--If need edit-->
nano ~/.bashrc
source ~/.bashrc
# Dspace install
## Install git
apt install git
## Create Dspace user
adduser dspace
## Add Dspace user to sudo group
usermod -aG sudo dspace
## Switch user to Dspace
su - dspace
## Clone Dspace repo
git clone https://github.com/DSpace/DSpace.git
cd DSpace
## Build DSpace project
mvn clean install
# Configure Dspace
## Copy files to Docker image
docker cp /path/to/local/file.txt <container_id>:<path/in/container>



## Downkoad source
wget https://github.com/DSpace/DSpace/archive/refs/tags/dspace-8.0.tar.gz
tar -zxvf dspace-8.0.tar.gz
<!--
In Dspace archive was Dockerfile fo building ready-to-go Dspace image. Need to check it for working with. 
-->
## Backup config file
cp ./DSpace-dspace-8.0/dspace/config/local.cfg.EXAMPLE ./DSpace-dspace-8.0/dspace/config/local.cfg
## Edit config file
<!--
Need to copy Dspace config from live server.
-->
## Build Dsapce via maven
cd DSpace-dspace-8.0/
mvn package
<!--It necessary to internet connection for Maven. Process take a while. -->
## Run Ant
cd dspace/target/dspace-installer
ant fresh_install
