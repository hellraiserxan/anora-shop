FROM maven:3.9.6-eclipse-temurin-21 AS build
WORKDIR /build

COPY pom.xml .

COPY apps/backend/auth-service apps/backend/auth-service
RUN mvn -f apps/backend/auth-service clean package -DskipTests

FROM eclipse-temurin:21-jre-jammy
WORKDIR /app

COPY --from=build /build/apps/backend/auth-service/target/auth-service-*.jar app.jar


EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]



