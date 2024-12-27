# Для перенесення налаштувань з Dspace 6 на Dspace 8 необхідно збілдити образ з Dockerfile
DSPACE_CFG_BAK="/backup-files/dspace.cfg.backup"
LOCAL_CFG_BAK="/backup-files/local.cfg.backup"
DSPACE_IMG_NAME="dspace:test"
# Завантажити Dockerfile з джерела
https://github.com/DSpace/DSpace/blob/main/Dockerfile
# Виконати збірку образу
docker build -t $DSPACE_IMG_NAME .



