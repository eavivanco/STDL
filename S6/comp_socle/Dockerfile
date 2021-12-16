# syntax=docker/dockerfile:1
FROM python:3.7-alpine

FROM maven:3.8.3-jdk-11-alpine AS build
COPY src /home/app/src
COPY pom.xml /home/app
RUN mvn -f /home/app/pom.xml clean package

WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

FROM openjdk:11-jre-alpine
COPY --from=build /home/app/target/demo-0.0.1-SNAPSHOT.jar /usr/local/lib/demo.jar
EXPOSE 8080

COPY . .
ENTRYPOINT ["java","-jar","/usr/local/lib/demo.jar"]
CMD ["flask", "run"]