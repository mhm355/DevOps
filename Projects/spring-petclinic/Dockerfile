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


