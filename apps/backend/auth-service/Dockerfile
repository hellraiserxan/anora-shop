FROM maven:3.9.6-eclipse-temurin-21 AS build
WORKDIR /build

COPY . /build

RUN mvn -f /build/apps/backend/auth-service/pom.xml clean package -DskipTests

FROM eclipse-temurin:21-jre-jammy
WORKDIR /app

COPY --from=build /build/apps/backend/auth-service/target/auth-service-*.jar app.jar


EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]



