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

# cloning private it repo

# run `ssh-keygen`  in instance .

```
# Use an official base image
FROM alpine:latest

# Install necessary packages
RUN apk add --no-cache git openssh

# Set build-time variable for Git repository URL
ARG REPO_URL

# Add SSH keys and set permissions
# You will need to copy your SSH key to the Docker build context
COPY id_rsa /root/.ssh/id_rsa
COPY id_rsa.pub /root/.ssh/id_rsa.pub

RUN mkdir -p /root/.ssh && \
    chmod 600 /root/.ssh/id_rsa && \
    chmod 600 /root/.ssh/id_rsa.pub

# Add GitHub's known_hosts to avoid host verification prompt
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts

# Clone the private repository
RUN git clone $REPO_URL /app

# Set the working directory
WORKDIR /app
```
