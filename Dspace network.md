# Змінні
DSPACE_NET_NAME="dspace-network"
# Створення мережі між контейнерами
docker network create dspace-network
# Приєднати існуючи контейнери до створеної мережі
docker network connect dspace-network $TOMCAT_CONTAINER_NAME
docker network connect dspace-network $POSTGRES_CONTAINER_NAME
docker network connect dspace-network $SOLR_CONTAINER_NAME
