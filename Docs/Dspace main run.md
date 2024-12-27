# Змінні
DSPACE_CONTANER_NAME="dspace-main"
# Завантажити офіційний образ Dspace
docker pull dspace/dspace
# Запустити контейнер з офіційного образу
docker run --name $DSPACE_CONTANER_NAME \
  --network $DSPACE_NET_NAME \
  -e "DB_HOST=$POSTGRES_CONTAINER_NAME" \
  -e "DB_NAME=$POSTGRES_DB" \
  -e "DB_USER=$POSTGRES_USER" \
  -e "DB_PASSWORD=$POSTGRES_PASSWORD" \
  -e "SOLR_HOST=solr_container_name" \
  -d dspace/dspace
