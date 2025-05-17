# Preparing the build image

## Base image
On this guidline, it used tomcat:9.0.105-jdk11-corretto-al2 for the base image

## Preparing DockerFile
### Ant based project
DockerFile
```
FROM tomcat:9.0.105-jdk11-corretto-al2
ARG BUILD_NUMBER="1.0.0"
LABEL image.name="{APPLICATION_NAME}
LABEL image.version="${BUILD_NUMBER}"
COPY {TOMCAT_RESOURCE}/context.xml /usr/local/tomcat/config/ 
COPY {TOMCAT_RESOURCE}/server.xml /usr/local/tomcat/config/
COPY {BUILD_PATH}/{APPLICATION_NAME}.war /usr/local/tomcat/webapp/
COPY setenv.sh /usr/local/tomcat/bin/
COPY {DEPENDENCY_LIB} /usr/local/tomcat/lib/
EXPOSE {SERVER_PORT}
```
Note:
<li>Replacing <b>APPLICATION_NAME</b></li>
<li>Replaceing <b>BUILD_PATH</b> with the actual build directory on your project</li>
<li>Replacing <b>DEPENDENCY LIB</b> with the actual 3rd part library/depedendency. If you already bundled all required dependency inside the build package(war), you can exclude the line for copying the lib from DockerFile </li>
<li>Replacing <b>TOMCAT_RESOURCE</b> with the actual directory for the tomcat resource</li>
<li>If your application define the tomcat server port with other port, you can replace <b>SERVER_PORT</b> with the actual port number. Otherwise, replace it with 8080<li> 

### Maven based project
DockerFile
```
FROM tomcat:9.0.105-jdk11-corretto-al2
ARG BUILD_NUMBER="1.0.0"
LABEL image.name="{APPLICATION_NAME}
LABEL image.version="${BUILD_NUMBER}"
COPY {TOMCAT_RESOURCE}/context.xml /usr/local/tomcat/config/ 
COPY {TOMCAT_RESOURCE}/server.xml /usr/local/tomcat/config/
COPY target/{APPLICATION_NAME}.war /usr/
COPY target/libs/*  /usr/local/tomcat/lib/
EXPOSE {SERVER_PORT}
```

Note:
<li>Replacing <b>APPLICATION_NAME</b></li>
<li>If your application define the tomcat server port with other port, you can replace <b>SERVER_PORT</b> with the actual port number. Otherwise, replace it with 8080<li> 


## Build the image
For the image for your application, please run this command
```sh
docker build --no-cache --build-arg BUILD_NUMBER={BUILD_NUMBER} -t {APPLICATION_NAME}:{BUILD_NUMBER} .
```
Note:
<li>Replacing both APPLICATION_NAME and BUILD_NUMBER with the actual application name and build number</li>

## Push the image to the Container Registry

### AWS ECR
#### Login to ECR
To login ECR, you can execute this command 
```sh
aws ecr get-login-password --region region | docker login --username AWS --password-stdin {AWS_ACCOUNT_ID}.dkr.ecr.{AWS_REGION}.amazonaws.com/{REPOSITORY_NAME}
```

Note:
<ol>
<li>Replacing <b>AWS_ACCOUNT_ID</b> and <b>AWS_REGION</b> with the actual information for both information</li>
<li>Replacing <b>REPOSITORY_NAME</b> with the actual repository name</li>
</ol>

If you don't create the repository before, you can run this command on your console
```sh
aws ecr create-repositry --repository-name="{REPOSITORY_NAME}"
```

### Tag the image
To tag the image, you can run this command
```sh
docker tag {APPLICATION_NAME}:${BUILD_NUMBER} {AWS_ACCOUNT_ID}.dkr.ecr.{AWS_REGION}.amazonaws.com/{REPOSITORY_NAME}/{APPLICATION_NAME}:{BUILD_NUMBER}
```

### Push the image
To push the image, you can run this command
```sh
docker push {AWS_ACCOUNT_ID}.dkr.ecr.{AWS_REGION}.amazonaws.com/{REPOSITORY_NAME}/{APPLICATION_NAME}:{BUILD_NUMBER}
```



