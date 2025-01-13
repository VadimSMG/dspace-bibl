# Запуску пакетів Dspace через Docker compose (офіційний репозиторій)
docker-compose up -d .
# Відновлення бази даних з бекапу
## Виконати копіювання файлів з локального сховища у контейнер Docker (за замовчуванням контейнер "dspacedb")
docker cp backup-test.sql dspacedb:/backup.sql
## Підключитися до контейнеру БД як користувач dspace
docker exec -it dspacedb psql -U dspace
## Перевірити всі наявні БД
\l
## Підключитися до БД, що відрізняється від "dspace"
\c postgres
## Виконати виділення існуючої БД
DROP DATABASE dspace;
## Підключитися до контейнеру у оболонку bash
docker exec -it dspacedb bash
## Виконати відновлення бази даниз з бекапу
psql -U dspace -d dspace -f backup.sql
