# Змінні
DSPACE_CONTAINER_NAME="dspace_service"
DB_ALIAS="repo"
# Розгортання контейнеру Dspace
docker run -d \
  --name $DSPACE_CONTAINER_NAME \
  -p 8080:8080 \
  --link $POSTGRES_DB:$DB_ALIAS \
  dspace/dspace-api:latest
