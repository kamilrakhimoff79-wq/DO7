# Этап сборки: Maven + JDK 8 (в pom.xml указан java.version=8)
FROM maven:3.8-openjdk-8 AS build

WORKDIR /app

# Сначала копируем только pom.xml и mvnw — зависимости кэшируются в отдельном слое
COPY pom.xml mvnw ./
RUN chmod +x mvnw && ./mvnw dependency:go-offline -B

# Затем копируем исходный код и собираем jar
COPY src ./src
COPY wait-for-it.sh ./
RUN ./mvnw package -DskipTests -B

# Финальный (рантайм) образ — только JRE, чтобы сократить размер
FROM openjdk:8-jre-alpine

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar
COPY --from=build /app/wait-for-it.sh wait-for-it.sh
RUN chmod +x wait-for-it.sh

EXPOSE 8085

# Сервису для старта нужна PostgreSQL (ждём её через wait-for-it.sh)
CMD ["sh", "-c", "./wait-for-it.sh ${POSTGRES_HOST}:5432 -t 60 -- java -jar app.jar"]
