# Dockerizing-flask-API-CACHING-App

A three tier flask api-caching application including 
 
 - Frontend    - An application which displays the geolocation information of an IP in a graphical web-interface
 - Backend     - A Redis database or Elasticache cluster which acts as a caching server and stores the IP informations
 - API-Service - A flask application which displays the IP information in json format

## Requirements

- [Install Docker](https://docs.docker.com/engine/install/)
- [Install Docker Compose](https://docs.docker.com/compose/install/)

## Prerequisite

- [API-Key](https://app.ipgeolocation.io/)  - A unique authentication key which is used to gain access to ipgeolocation API for fetching IP informations
- [IAM Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) - An AWS identity with SecretsManagerReadWrite permission policy
- [AWS Secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)

## Building Docker images for three layers

Here you need to build images for the 3 tiers Frontend, Backend and API-services

 1. Building image for Frontend
 
  ```
  docker build -t swathikarun/ipgeolocation-front:v1 .
  ```
 - Tagging the image
  
  ```
   docker tag swathikarun/ipgeolocation-front:v1 swathikarun/ipgeolocation-front:latest
  ```
 - Pushing the image to the repository

 ```
  docker image push swathikarun/ipgeolocation-front:latest
  docker image push swathikarun/ipgeolocation-front:v1
 ```

2. Building image for API-Service

 ```
  docker build -t swathikarun/ipgeolocation-api:v1 .
 ```
  - Tagging the image
  
  ```
  docker tag swathikarun/ipgeolocation-api:v1 swathikarun/ipgeolocation-api:latest
  ```
 - Pushing the image to the repository

 ```
 docker image push swathikarun/ipgeolocation-api:latest
 docker image push swathikarun/ipgeolocation-api:v1
 ```

## Provisioning

You can deploy the containers using Docker commands or via docker-compose. Before that you should attach the IAM role to the ec2-instance for accessing secrets

 ## Method 1 :: Using Docker commands

  -  Create network

     ```
      $ docker network create api-ipgeolocation-net
     ```

  - Creating caching container using redis image

     ```
      docker container run \
      -d \
      --name redis \
      --network api-ipgeolocation-net \
      --restart always \
      redis:latest
     ```

   - Creating container for API Service

     ```
      docker container run \
      -d \
      -p 8080:8080 \
      --name ipgeolocation-api-cache \
      --restart always \
      --network api-ipgeolocation-net \
      -e REDIS_HOST="redis" \
      -e APP_PORT="8080" \
      -e API_KEY_FROM_SECRETSMANAGER=True \
      -e SECRET_NAME="ipstack" \
      -e SECRET_KEY="api_key" \
      -e REGION_NAME="ap-south-1" \
      swathikarun/ipgeolocation-api:v1
     ```
    -  Creating container for Frontend

     ```
      docker container run \
      -d \
      -p 80:8080 \
      --restart always \
      --network api-ipgeolocation-net \
      --name ipgeolocation-frontend \
      -e API_SERVER="ipgeolocation-api-cache" \
      -e API_SERVER_PORT="8080" \
      -e API_PATH="/api/v1/" \
      -e APP_PORT="8080" \
      swathikarun/ipgeolocation-front:v1
     ```

 ## Method 2 
 
   - Check the syntax of docker-compose file
   
     ```
     $ docker-compose config
     ```
    
   - Deploy the containers using docker-compose

     ```
      $ docker-compose up -d
     ```
    
## Result

All the 3 containers have been deployed and now you can get the IP location information by calling : 

Frontend    - http://13.233.48.79/ip/55.38.123.7

JSON format - http://domain.com:8080/api/v1/143.67.43.80
    
