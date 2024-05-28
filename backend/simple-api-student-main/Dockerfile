# Étape de construction
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build

# Définir le répertoire de travail
WORKDIR /opt/myapp

# Copier les fichiers de configuration Maven
COPY pom.xml .

# Copier le code source
COPY src ./src

# Construire l'application avec Maven
RUN mvn package -DskipTests

# Étape de déploiement
FROM amazoncorretto:17

# Définir le répertoire de travail
WORKDIR /opt/myapp

# Copier le fichier JAR généré à partir de l'étape précédente
COPY --from=myapp-build /opt/myapp/target/*.jar myapp.jar

# Commande d'exécution de l'application
ENTRYPOINT ["java", "-jar", "myapp.jar"]
