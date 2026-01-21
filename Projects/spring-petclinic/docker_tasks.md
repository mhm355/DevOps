* base image : eclipse-temurin:17-jdk-alpine-3.23@sha256:252d0c785daa393b42b5c1ec3deed20432eba234fc4274dec723eb5f66b3817c
* runtime image : eclipse-temurin:17-jre-alpine-3.23@sha256:5e07a03ec4cb28213596507e398ff00cf128ba18133c67d8e6bbdb20fe69ad67

## Task1

- build images maually without using dockerfile or docker-compose

1. use `eclipse-temurin:17-jdk-alpine-3.23` as base image
2. create a container from the base image and install the needed tools

`docker run --name spring-build -it  eclipse-temurin:17-jdk-alpine-3.23@sha256:252d0c785daa393b42b5c1ec3deed20432eba234fc4274dec723eb5f66b3817c sh`

3. copy the project files to the container

```bash
docker exec spring-build mkdir spring-petclinic
docker cp $(pwd)/src spring-build:/spring-petclinic/src
docker cp $(pwd)/pom.xml spring-build:/spring-petclinic/
docker exec spring-build ls /spring-petclinic
```

4. download maven and dependencies

```bash
docker exec spring-build apk add --no-cache maven

docker exec spring-build sh -c "cd /spring-petclinic && mvn dependency:go-offline -B"
```

5. build the artifact

```bash
docker exec spring-build sh -c "cd /spring-petclinic && mvn clean package -DskipTests"
```

6. copy the artifact to new container and create the image "to decrease the image size"

```bash
docker run --name spring-run -p 8080:8080 -d eclipse-temurin:17-jre-alpine-3.23@sha256:5e07a03ec4cb28213596507e398ff00cf128ba18133c67d8e6bbdb20fe69ad67 

# Copy from source container to host
docker cp spring-build:/spring-petclinic/target/spring-petclinic-*.jar ./

# Copy from host to destination container
docker cp ./spring-petclinic-*.jar spring-run:/app/
```

7. commit the container to an image and run a container from this image

```bash
docker commit --change='CMD ["java", "-jar", "/app/spring-petclinic-4.0.0-SNAPSHOT.jar"]' spring-run commit-v
docker run --name commit-v -p 8081:8080 -d commit-v
```

## Task2

* build images using dockerfile

1. create a dockerfile

```dockerfile
FROM eclipse-temurin:17-jdk-alpine-3.23@sha256:252d0c785daa393b42b5c1ec3deed20432eba234fc4274dec723eb5f66b3817c  AS build

WORKDIR /app

RUN apk add --no-cache maven

COPY pom.xml ./

RUN mvn dependency:go-offline -B 

COPY src ./src

RUN mvn clean package -DskipTests  

#---- runtime stage ----#

FROM eclipse-temurin:17-jre-alpine-3.23@sha256:5e07a03ec4cb28213596507e398ff00cf128ba18133c67d8e6bbdb20fe69ad67 AS runtime

WORKDIR /app

COPY --from=build /app/target/*.jar /app/app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

2. build the image and run a container from this image

```bash
docker build -t spring-image .
docker run --name spring -p 8082:8080 -d spring-image
```

## Task3

* build images to run in multiple platforms such as intel and arm

1. create a dockerfile as the previous one
2. build the image for multiple platforms using dockerfile

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t mhmdocker1/spring-petclinic:v2.0 --push .
```

## Task4,5

* build images using docker-compose

1. create a docker-compose.yml file

* [docker-compose.yml](docker-compose.yml)

2. create nginx config file

* [nginx.conf](nginx.conf)

3. build the image and run a container from this image

```bash
TAG=v3.0 REPLICAS=4 MYSQL_PASSWORD=app docker compose up -d
```
