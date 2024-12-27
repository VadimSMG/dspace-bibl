# Доступ до віртуальної машини з Dspace (PW 8WYVxMVFUS)
ssh dspace@10.1.0.70
# Створення мінімального дампу бази даних PostgreSQL (для тестування)
pg_dump -U <username> -d <database_name> > backup-test.sql
# Створення бекапу налаштувань Dspace
dspace metadata-export -f </path/to/your/metadata.bak>
# Знайти файл конфігурацїї Dspace
## Зробити копію файлу конфігурації local.cfg та dspace.cfg
find /home/dspace -name "local.cfg"
cp </path/to/local.cfg> <path/to/new/file>
find /home/dspace -name "dspace.cfg"
cp </path/to/dspace.cfg> <path/to/new/file>
## Знайти каталог bitsream (не обов'язково)
find <path/to/search> -type d -name "bitstream"
# Завантаження дампу бази та файлів на локальне сховище
scp </path/to/local/file> username@remote.server:</path/to/remote/directory>

