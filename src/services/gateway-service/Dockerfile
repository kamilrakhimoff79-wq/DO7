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

EXPOSE 8087

# Gateway — фасад, ждёт все остальные микросервисы
CMD ["sh", "-c", "\
./wait-for-it.sh ${SESSION_SERVICE_HOST}:8081 -t 60 && \
./wait-for-it.sh ${HOTEL_SERVICE_HOST}:8082 -t 60 && \
./wait-for-it.sh ${BOOKING_SERVICE_HOST}:8083 -t 60 && \
./wait-for-it.sh ${PAYMENT_SERVICE_HOST}:8084 -t 60 && \
./wait-for-it.sh ${LOYALTY_SERVICE_HOST}:8085 -t 60 && \
./wait-for-it.sh ${REPORT_SERVICE_HOST}:8086 -t 60 && \
java -jar app.jar"]
