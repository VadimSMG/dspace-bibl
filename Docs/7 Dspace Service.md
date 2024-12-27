# Змінні
DSPACE_CONTAINER_NAME="dspace service"
# Запуск контейнера dspace з образу, створеного раніше
docker run -d --name $DSPACE_CONTAINER_NAME \
  --network $DSPACE_NETWORK \
  -e DATABASE_URL=jdbc:postgresql://$POSTGRES_CONTAINER_NAME:5432/$POSTGRES_DB \
  -e SOLR_URL=http://$SOLR_CONTAINER_NAME:8983/solr/gettingstarted \
  $DSPACE_IMG_NAME
  

