# Змінні
DSPACE_BACKEND_CNAME="dspace_backend_service"
# Використання стандартного образу Ubuntu для встановлення Dspace Backend
docker run --name $DSPACE_BACKEND_CNAME -it ubuntu
# Підготовка залежностей
apt update -y
## Встановлення Java JDK 17
apt install openjdk-17-jdk -y
## Додавання змінних
<!-- Видалення існуючих значень у .bashrc -->
sed -i '/^export JAVA_HOME=/d' ~/.bashrc
sed -i '/^export PATH=\$PATH:\$JAVA_HOME\/bin/d' ~/.bashrc
<!-- Додавання нових значень у кінець файлу .bashrc-->
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> ~/.bashrc
source ~/.bashrc
## Встановлення Apache Maven
JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
apt install maven -y
mvn -v
## Встановлення Apache Ant
apt install ant -y
## Редагування файлу local.cfg
<!--
# Database configuration
db.url = jdbc:postgresql://<postgres_container_ip>:5432/dspace
db.username = dspaceuser
db.password = dspacepassword

# Solr configuration
solr.server = http://<solr_container_ip>:8983/solr
-->


