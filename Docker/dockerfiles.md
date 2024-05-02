## multistage dockerfile

```
FROM openjdk:11 AS BUILD
RUN apt update -y && apt install maven -y
COPY ./  vprofile-project/
RUN cd vprofile-project && mvn clean package 

FROM tomcat:9-jre11
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=BUILD /vprofile-project/target/vprofile-v2.war  /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]

```
## springboot dockerfile

```
FROM adoptopenjdk/openjdk11:alpine-jre

ARG JAR_FILE=target/spring-boot-web.jar

WORKDIR /opt/app

COPY ${JAR_FILE} spring-boot-web.jar

ENTRYPOINT ["java","-jar","spring-boot-web.jar"]

```

## node js docker file

```
FROM node:latest

WORKDIR /usr/src/app

# Copy package.json and package-lock.json (if available)
COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]

```

### python docker file
```
# Use an official Python runtime as a parent image
FROM python:latest

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy the current directory contents into the container at /usr/src/app
COPY . .

# Install any needed dependencies specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]

```
