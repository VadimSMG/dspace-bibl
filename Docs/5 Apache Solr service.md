# Змінні
SOLR_CONTAINER_NAME="solr_service"
# Розгортання контейнеру Solr
docker run -d \
  --name $SOLR_CONTAINER_NAME \
  --network $DSPACE_NETWORK \
  -v $SOLR_VOLUME_NAME:/var/solr \
  -p 8983:8983 \
  solr solr-precreate gettingstarted
<!--
5. Створення ядра (core)
Для створення нового ядра в Solr можна використовувати команду solr-precreate. Наприклад, щоб створити ядро з ім'ям mycore, запустіть:

bash
docker run -d -p 8983:8983 --name my_solr solr:8 solr-precreate mycore

6. Налаштування конфігурації (за потреби)
Якщо у вас є власні файли конфігурації (наприклад, solrconfig.xml, schema.xml), ви можете змонтувати їх як обсяг:

bash
docker run -d -p 8983:8983 --name my_solr -v /path/to/your/config:/opt/solr/server/solr/mycores/mycore/conf solr:8

Тут /path/to/your/config — це шлях до вашої локальної директорії з конфігураційними файлами.
-->
