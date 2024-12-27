# Змінні
DSPACE_NETWORK="dspace-network"
# Створення мережі для об'єднання контейнерів
docker network create $DSPACE_NETWORK
# Перевірка наявних мереж
docker network ls
<!--
# Додавання запущеного контейнера до мережі
docker network connect <network_name> <container_name>
# Перевірка контейнерів, під'єднаних до мережі
docker network inspect <network_name>
-->
