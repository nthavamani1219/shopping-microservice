FROM maven as build
WORKDIR /app
COPY . .
RUN mvn install

FROM openjdk:17.0-jre
WORKDIR /app
COPY --from=build /app/target/offers-0.0.1-SNAPSHOT.jar /app 

CMD ["java", "-jar", "offers-0.0.1-SNAPSHOT.jar"] 

