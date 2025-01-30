# Get Dspace Fronend source files
git clone https://github.com/DSpace/dspace-angular.git
# Build image from Dockerfile
cd ./dspace-angular
docker build -t dspace-frontend .
# Run DSpace frontend container
docker run -d -P dspace-frontend
