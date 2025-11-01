# Dockerizing a Maven webapp

Minimal, copy-ready guide to build and run a Maven-based Java web application in Docker.

## Prerequisites
- Docker installed
- Maven (for local builds) or use multi-stage Docker build
- JDK version matching your app (example uses JDK 11)

## Project layout (typical)
- pom.xml
- src/main/java
- src/main/resources
- src/main/webapp (for WAR)
- target/ (build output)

## Project WOrk Proceduer
- Push the codes to Github - **git push origin masin**
- Pull the project in to server - **git pull origin main**
- build the project - **mvn clean package**
- build the Docker image in the project - **docker build -t canara_image .**
- Run the container - **docker run -it -d --name canara_container -p 8080:8080 canara_image**

### Optional that you can run using docker-compose also
- to build and run the container - **docker-compose up -d --build**
- to remove the container - **docker-compose down**
---
