# Змінні середовища
## Контейнер PostgreSQL
POSTGRES_USER="dspace"
POSTGRES_PASSWORD="BibL24%usr"
POSTGRES_DB="biblbase"
POSTGRES_CONTAINER_NAME="postgres_service"
## Шляхи до бекапів
POSTGRES_BAK="./backup-files/backup-test.sql"
# Створення контейнеру з сервісом PostgreSQL
docker run -d \
  --name $POSTGRES_CONTAINER_NAME \
  --network $DSPACE_NETWORK \
  -e POSTGRES_USER=$POSTGRES_USER \
  -e POSTGRES_PASSWORD=$POSTGRES_PASSWORD \
  -e POSTGRES_DB=$POSTGRES_DB \
  -v $DB_VOLUME_NAME:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:latest
<!--
-d - запускає контейнер у фоновому режимі
--name - визначає ім'я контейнера
-e - визначення змінних системи для контейнера
-v - монтує створений docker volume до контейнера
-p - відкриває порти для спілкування з другими контейнерами
-->
# Перевірка роботи контейнера за БД
docker exec -it $POSTGRES_CONTAINER_NAME psql -U $POSTGRES_USER -d $POSTGRES_DB
# Створення розширення pgcrypto ( при підключенні до БД)
CREATE EXTENSION IF NOT EXISTS pgcrypto;
# Налаштування БД
docker exec -it $POSTGRES_CONTAINER_NAME bash -c "echo 'host dspace dspace 127.0.0.1 255.255.255.255 md5' >> /var/lib/postgresql/data/pg_hba.conf"
# Перезавантаження контейенру після зміни конфігурації
docker restart $POSTGRES_CONTAINER_NAME
# Відновлення БД з бекапу (SQL) у контейнері 
cat $POSTGRES_BAK | docker exec -i $POSTGRES_CONTAINER_NAME psql -U $POSTGRES_USER -d $POSTGRES_DB
<!-- При відновленні БД з бекапу, кодування буде налаштовано відповідно до БД -->









