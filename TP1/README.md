# devops TP1 :

## 1-1
### Commands database : 
`docker build -t ewanc/database .`

`docker run -d --network app-network -v datadir:/var/lib/postgresql/data --name database ewanc/database`

### Dockerfile database : 

```bash
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

COPY docker-entrypoint-initdb.d /docker-entrypoint-initdb.d
```

### commands backend api :

`docker build -t ewanc/backend .`

`$ docker run -d  -p 8080:8080 --name backend ewanc/backend`

### Dockerfile backend :

```bash
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```
### commands server frontend : 
`docker build -t ewanc/httpd .`

`docker run -dit --network app-network --name httpd -p 8000:80 ewanc/httpd`

### Dockerfile frontend : 
```bash
FROM httpd:2.4
COPY ./ /usr/local/apache2/htdocs/
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf
```
## 1-2 : 

### Why do we need a multistage build : 
A multistage build is used to reduce the size of the final image. Here it only contains the JAR And the JRE instead of the whole JDK.
Since it contains only the JRE, it is also more secure.  
The build process is also simplified since we don't need to install the JDK and the JRE in the final image, and the final image is smaller and faster to build.

### Explanation of the Dockerfile :

`FROM maven:3.8.6-amazoncorretto-17 AS myapp-build`: This line specifies the base image for the runtime environment. 

`ENV MYAPP_HOME /opt/myapp`: This line sets the environment variable MYAPP_HOME to /opt/myapp.

`WORKDIR $MYAPP_HOME`: This line sets the working directory to /opt/myapp.

`COPY pom.xml .`: This line copies the pom.xml file from the current directory to the working directory.

`COPY src ./src`: This line copies the src directory from the current directory to the working directory.

`RUN mvn package -DskipTests`: This line runs the mvn package command to build the application, skipping the tests.

`FROM amazoncorretto:17`: This line specifies the base image for the final image.

`ENV MYAPP_HOME /opt/myapp`: This line sets the environment variable MYAPP_HOME to /opt/myapp.

`WORKDIR $MYAPP_HOME`: This line sets the working directory to /opt/myapp.

`COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar`: This line copies the JAR file from the build stage to the final image.

`ENTRYPOINT java -jar myapp.jar`: This line specifies the command to run when the container starts.

## 1-3 : 
`docker-compose up`: Builds, (re)creates, starts, and attaches to containers for a service. When run without the -d flag, it provides an interactive terminal to monitor the output.

`docker-compose down`: Stops and removes containers, networks, volumes, and images created by up.

`docker-compose build`: Builds or rebuilds services specified in the docker-compose.yml file.

`docker-compose logs`: Displays log output from services.

`docker-compose ps`: Lists all running containers related to the services defined in the docker-compose.yml file.

`docker-compose restart`: Restarts all stopped and running services.

`docker-compose stop` / `docker-compose start`: Stops/Starts services.

`docker-compose run`: Runs a one-time command against a service.

## 1-4 Docker-compose.yml:
```bash
version: '3.8'

services:
    backend:
        build: './backend/simple-api-student/'
        environment:  
          - SPRING_DATASOURCE_URL=jdbc:postgresql://database/db
          - SPRING_DATASOURCE_USERNAME=usr
          - SPRING_DATASOURCE_PASSWORD=pwd
        networks:
          - my-network

        depends_on:
          - database

    database:
        build: './database'
        networks:
          - my-network

    httpd:
        build: './http-server'
        ports:
          - 80:80
        networks:
          - my-network
        depends_on:
          - backend

networks:
    my-network:
```
This docker-compose file defines a multi-container Docker application with three services: backend, database, and httpd, all interconnected within a custom network named my-network.

`backend`: A service that seems to be a Spring Boot application (given the SPRING environment variables). It is built from a Dockerfile located in ./backend/simple-api-student/. The service is configured with environment variables to connect to a PostgreSQL database (database service) using specific credentials. It depends on the database service, ensuring the database service is started before the backend service.

`database`: This service builds from a Dockerfile in ./database. It's likely a PostgreSQL database given the JDBC URL in the backend service's environment variables. It's also part of the my-network network, allowing it to communicate with the other services.

`httpd`: Represents a web server, built from a Dockerfile in ./http-server, which probably serves as a reverse proxy or static file server. It exposes port 80 to the host, allowing HTTP traffic to reach the service. This service depends on the backend service, ensuring the backend is started before the HTTP server.

`Networks`: The my-network custom network enables isolated communication between the services.

## 1-5 Publication commands :

`docker login`: Log in to a Docker registry.

`docker tag devops-backend ottera/devops-backend`: Tag an image so that it can be pushed to a registry.

`docker push ottera/devops-backend`: Push an image to a registry.

Then we do the same with `devops-httpd` and `devops-database`.

Link of repository : https://hub.docker.com/u/ottera
