# Run Ubuntu docker image (in interactive mode)
docker run --name dspace-service -it -p 8000:8000 -p 8080:8080 -p 8983:8983 -p 5432:5432 ubuntu
# Update packages
apt update -y
apt upgrade -y

# PREPARE SYSTEM
# Install Java 
apt install openjdk-21-jdk openjdk-17-jdk wget vim git -y
<!--Select region "Europe" timezone "Kyiv"-->
java --version
## Add OpenJDK to PATH
<!-- Remove old strings in .bashrc if it exist-->
sed -i '/^export JAVA_HOME=/d' ~/.bashrc
sed -i '/^export PATH=\$PATH:\$JAVA_HOME\/bin/d' ~/.bashrc
<!-- Add new sting at the end of .bashrc-->
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> ~/.bashrc
source ~/.bashrc
## Check JAVA_HOME
echo $JAVA_HOME

# Install Apache Maven 3.9.9
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
tar -xvf apache-maven-3.9.9-bin.tar.gz
mv apache-maven-3.9.9 /opt/maven
rm apache-maven-3.9.9-bin.tar.gz
## Configure environment for Maven
echo 'export M2_HOME=/opt/maven' >> /etc/profile.d/maven.sh
echo 'export PATH=$M2_HOME/bin:$PATH' >> /etc/profile.d/maven.sh
source /etc/profile.d/maven.sh
## Check Maven
mvn -version

# Install Ant 1.10.15
wget https://dlcdn.apache.org//ant/binaries/apache-ant-1.10.15-bin.tar.gz
tar -xvf apache-ant-1.10.15-bin.tar.gz
mv apache-ant-1.10.15 /opt/ant
rm apache-ant-1.10.15-bin.tar.gz
## Configure environment for Ant
echo 'export ANT_HOME=/opt/ant' >> /etc/profile.d/ant.sh
echo 'export PATH=$ANT_HOME/bin:$PATH' >> /etc/profile.d/ant.sh
source /etc/profile.d/ant.sh
## Check Ant
ant -version

# Install PostgreSQL
apt install postgresql-common -y
/usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
apt install postgresql-17 postgresql-contrib -y
## Configure TCP/IP in PostgreSQL for DSpace
sed -i "/listen_addresses = 'localhost'/s/./ /" /etc/postgresql/17/main/postgresql.conf
sed -i '$ a host    dspace dspace   127.0.0.1       255.255.255.255         md5' /etc/postgresql/17/main/pg_hba.conf
## Check PostgreSQL service
service postgresql status
service postgresql start

# Install Solr 9
wget https://downloads.apache.org/solr/solr/9.7.0/solr-9.7.0.tgz
tar xzf solr-9.7.0.tgz
apt install lsof -y
bash solr-*/bin/install_solr_service.sh solr-*.tgz
rm solr-9.7.0.tgz
## Configure file limits for Solr
echo '* soft nofile 65536' >> /etc/security/limits.conf
echo '* hard nofile 65536' >> /etc/security/limits.conf
echo 'fs.file-max = 100000' >> /etc/sysctl.conf
sysctl -p
## Check file limits
ulimit -n
cat /proc/sys/fs/file-max
## Restart Solr
service solr restart
## Check Solr
service solr status
## Check Solr user
cat /etc/passwd | grep 'solr'
## Allow acces to Solr fom external IPs (only for testing)
<!-- Find parmeter SOLR_JETTY_HOST and set it to allow all (0.0.0.0)-->
sed -i 's/^SOLR_JETTY_HOST=.*/SOLR_JETTY_HOST=0.0.0.0/' /etc/default/solr.in.sh
service solr restart

<!--
# Install Tomcat
## Add user for Tomcat
useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat
## Check Tomcat user
cat /etc/passwd | grep 'tomcat'
## Add permisson for read and write to DSpace
usermod -aG dspace tomcat
chmod g+rw /dspace
## Download binares
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.34/bin/apache-tomcat-10.1.34.tar.gz
tar xzf apache-tomcat-10.1.34.tar.gz
mv /apache-tomcat-10.1.34/* /opt/tomcat
## Config variables
echo 'export CATALINA_HOME=/opt/tomcat' >> /etc/profile.d/tomcat.sh
echo 'export PATH=$PATH:$CATALINA_HOME/bin' >> /etc/profile.d/tomcat.sh
source /etc/profile.d/tomcat.sh
## Set Tomcat as service
apt install clang -y
cd $CATALINA_HOME/bin
tar xvfz commons-daemon-native.tar.gz
cd commons-daemon-1.1.x-native-src/unix
./configure
make
cp jsvc ../..
cd ../..
## Configuring daemon
CATALINA_BASE=$CATALINA_HOME
cd $CATALINA_HOME
./bin/jsvc \
    -classpath $CATALINA_HOME/bin/bootstrap.jar:$CATALINA_HOME/bin/tomcat-juli.jar \
    -outfile $CATALINA_BASE/logs/catalina.out \
    -errfile $CATALINA_BASE/logs/catalina.err \
    --add-opens=java.base/java.lang=ALL-UNNAMED \
    --add-opens=java.base/java.io=ALL-UNNAMED \
    --add-opens=java.base/java.util=ALL-UNNAMED \
    --add-opens=java.base/java.util.concurrent=ALL-UNNAMED \
    --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED \
    -Dcatalina.home=$CATALINA_HOME \
    -Dcatalina.base=$CATALINA_BASE \
    -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager \
    -Djava.util.logging.config.file=$CATALINA_BASE/conf/logging.properties \
    org.apache.catalina.startup.Bootstrap
## Run Tomcat daemon
$CATALINA_HOME/bin/daemon.sh start
## Modification server.xml
vim /opt/tomcat/conf/server.xml
<!-- Edit <Connector> section to next look -->
<!-- Define a non-SSL HTTP/1.1 Connector on port 8080 -->
<Connector port="8080"
              minSpareThreads="25"
              enableLookups="false"
              redirectPort="8443"
              connectionTimeout="20000"
              disableUploadTimeout="true"
              URIEncoding="UTF-8"/>
-->

# INSTALL DSPACE BACKEND
## Create DSpace system user
useradd -m dspace
## Download DSpace source archive
git clone https://github.com/DSpace/DSpace.git
mv /DSpace /dspace-source

## Create DSpace user for PostgreSQL
### Switch to postgers user
su postgres
### Open PostgreSQL shell
psql
### Create user for DSpace (test PW dspace)
CREATE USER dspace WITH PASSWORD 'dspace' CREATEDB;
### Check users
\du

## Create DSpace database
CREATE DATABASE dspace WITH ENCODING 'UTF8' OWNER dspace;
### Check avaiable datatbases
\l

## Enable pgcrypro in DSpace database
### Connect to dspace database
\c dspace
### Create pgcrypto extension for this database
CREATE EXTENSION IF NOT EXISTS pgcrypto;
### Check the pgcrypto was installed
\dx
## Live the psql shell
\q
## Exit from postgres user to root
exit

## Initial configuration (local.cfg)
### Copy local.cfg.EXAMPLE as local.cfg
mv /dspace-source/dspace/config/local.cfg.EXAMPLE /dspace-source/dspace/config/local.cfg
### Edit new local.cfg
vim /dspace-source/dspace/config/local.cfg
<!-- Change this file for personal porposes. For test porposes use local.cfg.EXAMLE -->

## Making DSpace directory
mkdir dspace
chown -R dspace:dspace /dspace
chown -R dspace:dspace /dspace-source
chmod -R u+w /dspace-source
## Check permisions
ls -l /dspace-source/
ls -l /dspace
## Build Installation 
su - dspace
mvn -version
cd /dspace-source
mvn clean install

## Install DSpace
cd /dspace-source/dspace/target/dspace-installer
ant fresh_install
## Initialize database
cd /
dspace/bin/dspace database migrate
## Check database
dspace/bin/dspace database info
<!-- A fully initialized database should list the state of all migrations as either "Success" or "Out of Order" -->
exit

## Copy Solr cores
cp -R /dspace/solr/* /opt/solr-9.7.0/server/solr/configsets
cp -R /dspace/solr/* /var/solr/data
chmod -R 755 /var/solr/data/
## Add solr user ownership
chown -R solr:solr /opt/solr-9.7.0/server/solr/configsets
chown -R solr:solr /opt/solr-9.7.0/
chown -R solr:solr /opt/solr/
chown -R solr:solr /var/solr/
## Add fix for Solr 9
vim /var/solr/data/search/conf/solrconfig.xml
vim /var/solr/data/qaevent/conf/solrconfig.xml
vim /var/solr/data/suggestion/conf/solrconfig.xml
<!-- CHANGE
    <!-- Include contributed libraries that we use in DSpace. -->
<!--<lib dir='${solr.install.dir}/contrib/analysis-extras/lib/'
         regex='icu4j-.*\.jar'/>
    <lib dir='${solr.install.dir}/contrib/analysis-extras/lucene-libs/'
         regex='lucene-analyzers-icu-.*\.jar'/>
    TO
     <!-- the contributed libraries have a different location in solr 9.0 -->
    <lib dir='${solr.install.dir}/modules/analysis-extras/lib/'
         regex='icu4j-.*\.jar'/>
    <lib dir='${solr.install.dir}/modules/analysis-extras/lib/'
         regex='lucene-analysis-icu-.*\.jar'/>
-->
## Restart Solr
service solr restart

# Deploy web application
java -jar /dspace/webapps/server-boot.jar --dspace.dir=/dspace --logging.config=file:///dspace/config/log4j2.xml
# Check DSpace backend
localhost:8080/server

# Create Administrator Account in DSpace
/dspace/bin/dspace create-administrator

# Create Cron tasks
apt install cron
su - dspace
crontab -e
<!-- Necessary cron tasks here: https://wiki.lyrasis.org/display/DSDOC8x/Scheduled+Tasks+via+Cron -->

# Enable HTTPS support (it`s necessary for Prod not for testing)
## Install Nginx
apt install nginx -y
vim etc/nginx/sites-available/my.dspace.edu
<!-- Insert next default config for 'server block' 
# Setup HTTP to redirect to HTTPS
server {
  listen 80;
  # Add your domain here. We've added "my.dspace.edu" as an example
  server_name my.dspace.edu;
  rewrite ^ https://my.dspace.edu permanent;
}
 
# Setup HTTPS access
server {
  listen 443 ssl;
  # Add your domain here. We've added "my.dspace.edu" as an example
  server_name my.dspace.edu;
 
  # Add your SSL certificate/key path here
  # NOTE: For LetsEncrypt, the certificate should be the full certificate chain file
  ssl_certificate my.dspace.edu.crt (or PEM);
  ssl_certificate_key my.dspace.edu.key;
 
  # Proxy all HTTPS requests to "/server" from NGinx to Tomcat on port 8080
  location /server {
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header X-Forwarded-Host $host;
    proxy_pass http://localhost:8080/server;
  }
}
-->

# INSTALLING FRONTEND
## Install Node.js (LTS)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc
nvm install 22.13.1
## Check install
node -v
## Install NPM
apt install npm -y
## Install Yarn (v1.x)
npm install --global yarn
## Install Process Manager (necessary for Prod)
npm install --global pm2
# Download latest DSpace Aingular frontend
git clone https://github.com/DSpace/dspace-angular.git
## Install local dependencies
cd /dspace-angular
yarn install
## Build/Compile Prod
yarn build:prod
