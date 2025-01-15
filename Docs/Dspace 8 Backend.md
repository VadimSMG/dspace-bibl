# Змінні
DSPACE_BACKEND_CNAME="dspace_backend_service"
# Використання стандартного образу Ubuntu для встановлення Dspace Backend
docker run --name $DSPACE_BACKEND_CNAME -it ubuntu
# Підготовка залежностей
apt update -y

## ВСТАНОВЛЕННЯ Java JDK 17
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
apt install maven -y
## Встановлення Apache Ant
apt install ant -y

# ВСТАНОВЛЕННЯ БД PostgreSQL + pgcrypto
apt install postgresql postgresql-contrib -y
service postgresql start
# Створення та налаштування БД
## Перейти до користувача postgres
su - postgres
### Запустити оболонку
psql
### Створити нового користувача dspace
CREATE USER dspace WITH PASSWORD NULL;
### Створити нову БД dspace з кодуванням UTF8
CREATE DATABASE dspace WITH ENCODING 'UTF8' OWNER dspace;
### Перевірка кодування БД
SHOW SERVER_ENCODING;
### Надати повний доступ до БД користувачу dspace
GRANT ALL PRIVILEGES ON DATABASE dspace TO dspace;
### Додати розширення pgcrypto
CREATE EXTENSION IF NOT EXISTS pgcrypto;
### Вийти з оболонки БД
\q
## Налаштування TCP/IP для Postgres
### Редагування конфігурації postgresql.conf
sed -i '$ a listen_addresses = '\''localhost'\''' /etc/postgresql/16/main/postgresql.conf
<!--
Додавання "listen_addresses = 'localhost'" у кінець файлу postgresql.conf для ver. 16. Це необхідне налаштування для використання TCP/IP.
-->
### Редагування конфігурації pg_hba.conf
sed -i '$ a host    dspace dspace   127.0.0.1       255.255.255.255         md5' /etc/postgresql/16/main/pg_hba.conf
<!--
Додавання "host dspace dspace 127.0.0.1 255.255.255.255 md5" у кінець файлу pg_hba.conf file for ver. 16.
first "dspace" - is DB name
second "dspace" - is user and PW for DB access
-->
## Вихід з облікового запису postgres
logout

# Встановлення та налаштування Apache Solr 9
apt install wget lsof -y
## Завантажити архів
wget https://dlcdn.apache.org/solr/solr/9.7.0/solr-9.7.0.tgz
## Розпакувати архів
tar xzf solr-9.7.0.tgz
## Виконати скрипт встановлення
bash solr-9.7.0/bin/install_solr_service.sh solr-9.7.0.tgz
## Налаштування Solr конфігурації
<!--
Внесення цих змін необхідно для коректної роботи DSpace.
-->
sed -i.bak '/<!-- Include contributed libraries that we use in DSpace. -->/,/regex="solr-cell-\d.*\.jar"/{ 
s|<!-- Include contributed libraries that we use in DSpace. -->|<!-- the contributed libraries have a different location in solr 9.0 -->|; 
s|<lib dir='\''${solr.install.dir}/contrib/analysis-extras/lib/'\'' regex='\''icu4j-.*\.jar'\''/>|<lib dir='\''${solr.install.dir}/modules/analysis-extras/lib/'\'' regex='\''icu4j-.*\.jar'\''/>|; 
s|<lib dir='\''${solr.install.dir}/contrib/analysis-extras/lucene-libs/'\'' regex='\''lucene-analyzers-icu-.*\.jar'\''/>|<lib dir='\''${solr.install.dir}/modules/analysis-extras/lib/'\'' regex='\''lucene-analysis-icu-.*\.jar'\''/>|; 
s|<lib dir="\${solr.install.dir}/contrib/extraction/lib" />|<lib dir="\${solr.install.dir}/modules/extraction/lib" />|
}' /opt/solr/server/solr/configsets/_default/conf/solrconfig.xml
<!--
 * sudo: Виконує команду з правами адміністратора.
 * sed: Інструмент для обробки тексту.
 * -i.bak: Змінює файл на місці і створює резервну копію з розширенням .bak.
    /pattern1/,/pattern2/{ ... }: Вказує діапазон рядків для заміни, починаючи з pattern1 до pattern2.
 * s|old_text|new_text|: Заміна old_text на new_text.
-->
## Вимкнення автентифікації Solr
sed -i.bak 's|^SOLR_OPTS=.*|SOLR_OPTS="$SOLR_OPTS -Dsolr.security.json="|' /etc/default/solr.in.sh
## Перезапустити сервіс Solr
service solr restart

# ВСТАНОВЛЕННЯ Tomcat
## Створення користувача
groupadd tomcat
useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
## Завантаження Tomcat
apt install curl -y
cd /tmp
curl -O https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.2/bin/apache-tomcat-11.0.2.tar.gz
## Встановлення Tomcat
mkdir /opt/tomcat
tar xzvf apache-tomcat-11.0.2.tar.gz -C /opt/tomcat --strip-components=1
## Зміна прав доступу до скриптів
chown -RH tomcat: /opt/tomcat/
chmod +x /opt/tomcat/bin/*.sh
## Створення служби Tomcat
nano /etc/systemd/system/tomcat.service
<!-- Вміст файлу -->
[Unit]
Description=Apache Tomcat Server
After=syslog.target network.target

[Service]
Type=forking
User=tomcat
Group=tomcat
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment=CATALINA_PID=/opt/tomcat/tomcat.pid
ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
## Завантаження конфігурації служби
systemctl daemon-reload
