# Запуску пакетів Dspace через Docker compose (офіційний репозиторій)
docker-compose up -d .
# Відновлення бази даних з бекапу
## Виконати копіювання файлів з локального сховища у Docker VOlume, який пов'язано з контейенром (за замовчуванням контейнер "dspacedb" шлях "pgdata:/pgdata")
docker cp backup-test.sql dspacedb:/pgdata/backup.sql
## Підключитися до контейенра
docker exec -it dspacedb bash
## Виконати відновлення бази даниз з бекапу
psql -U dspace -d dspace -f /pgdata/backup.sql
