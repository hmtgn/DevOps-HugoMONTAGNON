# Première étape : Utiliser l'image JDK pour compiler le code Java
FROM eclipse-temurin:17-jdk-alpine as builder

# Copier le code source Java dans le conteneur
COPY Main.java /usr/src/

# Définir le répertoire de travail
WORKDIR /usr/src/

# Compiler le code Java en bytecode
RUN javac Main.java

# Deuxième étape : Utiliser l'image JRE pour exécuter l'application Java
FROM eclipse-temurin:17-jre-alpine

# Copier le fichier de classe compilé depuis l'étape précédente
COPY --from=builder /usr/src/Main.class .

# Définir le répertoire de travail
WORKDIR /usr/src/

# Exécuter l'application Java avec le JRE
CMD ["java", "Main"]
