# Змінні середовища
TOMCAT_CONTAINER_NAME="tomcat_service"
# Запуск контейнера з сервісом Tomcat
docker run -d \
  --name $TOMCAT_CONTAINER_NAME \
  --network $DSPACE_NETWORK \
  -v $TOMCAT_VOLUME_NAME:/usr/local/tomcat/logs \
  -p 8080:8080 \
  tomcat:latest
