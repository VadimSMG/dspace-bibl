# Завантажити офіційний репозиторій DSpace
git clone https://github.com/DSpace/DSpace.git
# Перейти до каталогу
cd Dspace/
# Запустити білд файлу образу
docker build -t dspace-image .
# Запустити контейнер зі створеного образу
docker run -d --name dspace-main -p 8081:8081 dspace-image

