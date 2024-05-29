# Étape de construction
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
WORKDIR /opt/myapp
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests
FROM amazoncorretto:17
WORKDIR /opt/myapp
COPY --from=myapp-build /opt/myapp/target/*.jar myapp.jar
ENTRYPOINT ["java", "-jar", "myapp.jar"]
