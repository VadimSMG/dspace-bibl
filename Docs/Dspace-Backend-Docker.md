# Download DSpace Backend source
git clone https://github.com/DSpace/DSpace.git
cd ./DSpace
# Run docker compose conatiners 
<!-- Deails of container config see in DSpace/docker-compose.yml -->
docker compose up -d
# Check services is OK
## Solr
localhost:8983
## DSpace backend
localhost:8080/server


