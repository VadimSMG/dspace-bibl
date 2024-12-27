# Змінні
DB_VOLUME_NAME="postgres_db"
SOLR_VOLUME_NAME="solr_data"
TOMCAT_VOLUME_NAME="tomcat_logs"
# Створення DockerVolume для розміщення файлів бази даних PostgreSQL
docker volume create $DB_VOLUME_NAME
docker volume create $SOLR_VOLUME_NAME
docker volume create $TOMCAT_VOLUME_NAME
## Перевірити всі наявні Docker Volume
docker volume ls

