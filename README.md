Containerization with Docke:

INSTALL DOCKER 

https://docs.docker.com/engine/install/ubuntu/


Add Docker's official GPG key:

sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

Add the repository to Apt sources:

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

Install docker package

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

docker --version


IMAGES in DOCKER
PULL a Base IMAGE


# docker pull ubuntu 

# docker pull ubuntu:22.04 

# docker images

# docker rmi ubuntu 

# docker rmi ubuntu:22.04

Docker Containers - Runtime Options

Container should be created from an image  -> container should be running

We can use Modes with docker run command 

1. Foreground mode (-it)

-i = interative 
-t = terminal

  --> Container will created, container will running
  --> root user will be attached to the terminal of the container

# docker run -it ubuntu

# docker ps -a

Come out of the container: 
keep the container running: press CTRL pq

Go inside the container -> attach to the terminal of the container 
# docker attach <containerid>

Come out of the container: 
Stop the container: exit 


2. Detached mode (-d):

--> Container will created, container will running
--> root user will NOT be attached to the container


# docker run -d nginx



# docker ps -a

# docker info


3.Execute a command on the container from the host server

# docker exec -it <container id> bash 


4. Delete all containers

# docker rm -f $(docker ps -aq)

5. Clean up the docker host (Delete stopped containers, delete all images)

# docker system prune --all

Give y 

It will delete all stopped containers and delete all images
6. Port Mapping or Port Forwarding

By default an application on a docker container is available on its exposed port.

It is the users internet that cannot reach the container port

For this we have do port forwarding or mapping in docker

i.e. we have to map a VM free system port with container port

example = 8989:80

systemport:containerport

> If a container is already created, we cannot do port forwarding for the container

> port forwarding will be applied at the docker run command itself

> flag for port forwarding (-p) and -P

# docker run --name web1 -d -p 8989:80 nginx

> if you want docker to do port mapping then use flag -P

# docker run --name web2 -d -P httpd



7. Docker Volumes

NAMED VOLUMES
Use case 1: Preserve data of the container on the docker host, even if container is deleted, Data should be persistent

Create a volume on the host machine where we will preserve data of container 

# docker volume create myvol 

         It creates a directory in /var/lib/docker/volumes folder

See details of the volume : 

#  docker volume inspect myvol

Mount the volume on the container : 
  # docker run -it -v myvol:/tmp ubuntu

You will be on the container

Create 2 files in /tmp directory 

# cd /tmp
# touch contfile1 contfile2

Come out the container 

Go to volume directory : 

# cd /var/lib/docker/volumes/myvol/_data 

# ls 

Containers data will be available.

Now delete the container 

# docker rm -f <container id> 

But data will still be there in the volumes directory.


Use Case 2: Mount the files from host machine to the containers directory 

Already data is present in the volumes directory. Just mount it on the container using -v option in docker run command.

# docker run -it -v myvol:/tmp ubuntu

In /tmp directory you will see the data. 
Come out of the container. 

Use Case 3: Sharing of data between the 2 containers using volume 


Create an ubuntu container, run a command that generates date -> send output to a file (index.html) 

Preserve this file on the volume

# docker run -it -v myvol1:/tmp ubuntu

# cd /var/lib/docker/volumes/myvol1/_data 

# ls

We will see the index.html 

Create a new container, that will read data from volume.

 # docker run -d -P -v myvol1:/usr/local/apache2/htdocs/  httpd


BIND MOUNT VOLUME

In this type of volume, any directory on the VM can be taken and mounted on the container during runtime.

We have a repository where developers are maintaining the application code.

https://github.com/Sonal0409/ecomm.git


We want to deploy this application on a httpd container. Get the above code on the docker host machine. We also want to access the ecomm application from the browser 


# git clone https://github.com/Sonal0409/ecomm.git

#  docker run --name myweb -d -P  -v /root/ecomm:/usr/local/apache2/htdocs/    httpd

# docker ps -a

Any change the developer makes on the code, we have to fetch the latest code and the container should have the new changes. 

# docker pull origin master 

Changes will reflect on container,

========================================

8. Custom Images


Commit a container in to Image


When ever we have to create our custom image using dockerfile, docker will always create a temporary container -> execute the argument of dockerfile → commit container into an image -> thereby getting a custom image layer → we can run this custom image → we get another container → in this again the next step of dockerfile is executed→ this container is again committed into an Image -> thereby getting next custom image layer-> the process continues until all the steps of dockerfile are executed.
These temporary containers are deleted 
The Final Image is collection of all the image layers

Do not do this demo- it is just for understanding the build process of docker.



# docker run -it --name=cont1 ubuntu


We will  be on the container 

Customize the container by installing some packages 

# apt-get update && apt-get install git -y

# apt-get install curl -y


Come out of the container - press CTRL pq 

Commit the container into an Image 

# docker commit cont1 myubuntu

# docker images 


Now delete the container and we create a new container using Custom image 


# docker rm -f $(docker ps -aq)

# docker run -it --name=mycont myubuntu

On the containers run the command to validate 

# curl --version
# git --version

Come out of container -> exit

Pushing image to dockerhub, so others can access your image

# docker login -u <docker-hub-username>

Give password when asked 

Login will be successful

Change the image name, so as to include you account library 

# docker tag myubuntu  sonal04/myubuntu
Push the image to your dockerhub account

# docker push sonal04/myubuntu 


You can go to dockerhub → your hub → see the custom image is present



Dockerfile
==============================

A dockerfile is a simple text file 
It consist of instructions of what you want on your container
The name of the file will always be dockerfile or Dockerfile 
There is no extension to this file 
To write this file no special coding skills are required
This file is written using 12 docker keywords and we arguments to these keywords 

Syntax: 

keyword argument/command 

====================================


          
Keywords to write dockerfile:
=================================== 

1. FROM
A dockerfile will always start with FROM keyword 
FROM which base image you want to customize your own image
What is the base image over which you will do your customization
FROM keyword also indicates what will be the base of your container 
This is mandatory keyword 

2. RUN
run linux commands to install/upgrade/uninstall packages on the container 
run linux commands to create directories, create files, users, groups and permissions
This keyword can be repeated multiple times in the dockerfile 
the container when created form this dockerfile image-- all these linux commands will be pre-executed on it 
This is not a mandatory keyword 

3. ENV
to create variables and give values them, we use this keyword 
The scope of these variables is only in dockerfile 
the variables can be called in dockerfile with synatx as ${VAR_NAME}
This is not a mandatory keyword 

4. LABEL 
example:
 LABEL key= "value"
LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"
This is not a mandatory keyword 
The LABEL instruction adds metadata to an image. 
A LABEL is a key-value pair. 
To include spaces within a LABEL value, use quotes and backslashes as you would in command-line parsing. 
A few usage examples:
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
An image can have more than one label. You can specify multiple labels on a single line.


5. COPY 
It is used to copy a file from docker host to the containers directory 
But it cannot copy .tar files on container directory 

6. ADD 
It is used to copy a file from docker host to the containers directory
But it can copy .tar files, by extracting the files and copy it on container directory
If there is a file on a URL -> copy keyword can connect to the url and download the file on container


7. EXPOSE 
using this keyword, we provide port number
this will be target port of container, where the application is exposed.

8. CMD 
using this keyword we will provide the  command that will run once the container is launched
this command will be responsible to start/stop an application/sever on container or execute any file
The default command on the container can be replaced by a new command passed at runtime 

9. ENTRYPOINT
using this keyword we will provide the  command that will run once the container is launched
The default command on the container CANNOT be replaced by a new command passed at runtime rather the new command passed at runtime goes as an argument to the entrypoint command.



Build an image from dockerfile 

# docker build -t myimagenginx .


Create a dockerfile to deploy HTMl code on nginx server 

# mkdir mydockerfiles 

# cd mydockerfiles 

# vim dockerfile 



# this is our nginx deployment dockerfile

FROM ubuntu
RUN apt-get update
RUN apt-get install nginx -y
LABEL description="This is nginx deployment"
COPY index.html /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

Save the file 

# vim index.html 

<h1> This file is from docker </h1> 


Build the dockerfile to image 

# docker build -t myimagenginx . 

# docker images 

# docker run -d -P myimagenginx

# docker ps -a 



CMD and ENTRYPOINT Difference - DEMO

Create a shell script 

# vim script.sh 

#!/bin/sh

echo "running commands using container"
echo "passing arguments using CMD keyword"

echo "Argument passed using CMD : $@"


# vim dockerfile1


FROM ubuntu
WORKDIR /myfiles
COPY script.sh /myfiles/script.sh
RUN chmod 777 /myfiles/script.sh
ENTRYPOINT ["sh", "/myfiles/script.sh"]
CMD ["Image", "container"]

Build the image
# docker build -t myimage01 -f dockerfile1 .

# docker run myimage01 

Output will be : 

Script will run and arguments will be added to script output 

# docker run myimage01  docker kubernetes 

Output will be : 

Script will run and new arguments will be added to script output 


Build a python Image
https://github.com/Sonal0409/Jenkins-Python-Pipeline-demo.git

Build a nodejs image
https://github.com/Sonal0409/nodejsappDockerfile.git


Multi Stage Build dockerfile

Many a times docker admins create the docker file, using the keywords like FROM,RUN,COPY,ADD
in which they would install build tools, compilers, dependencies , testing tools on the container
And perform steps, compiling, testing, packing on the container --> then deploy
By doing these steps on the container unnecessary we are increasing the size of the image 

FROM ubuntu
RUN apt install java
RUN apt install maven 
COPY src /
RUN mvn compile 
RUN mvn test 
RUN mvn package
RUN apt install tomcat 
COPY target/*.war /usr/share/local/webapps
CMD["catilina.sh"; "run"]


Multiple stages in a dockerfile allows you to break your dockerfile with multiple FROM keyword
There bycreating a stage in the dockerfile 
The output of a stage will be an input to next stage 
Advantage : 
only the last stage of dockerfile will be built into your Final application Image 
We would not leak build tools, artifacts via the images
The size image will be reduced significantly

Ever stage in the docker file will get a name using AS keyword 

Every FROM keyword in the same dockerfile represents a stage

Demo:
==================
Clone thai repository and run the dockerfiles

https://github.com/Sonal0409/multistagebuild-GoLang.git


Explanation of dockerfile:
=======================

FROM ubuntu AS build
   
    FROM ubuntu: Pull the ubuntu image form docker hub
      -> this will be the base of the container

    AS build : AS is keyword that assigns a name to the stage
    Here name of this stage is build 

    Whenever we see the FROM keyword in the dockerfile -> that means it is         the beginning of the stage .

RUN apt-get update && apt-get install -y golang-go

Updates apt repo on the container and install golang-go build tool 

ENV GO111MODULE=off
Environment variable of go, to disable go modules  

COPY . .

Copy the current code to container current directory 

RUN go build -o /app .

Run the go build command, build the code in current directory and name the artifact/compiled file as /app 
The /app is a self-sufficient file that consists of os lib, go  compiled binary that can run by itself.

Start the second stage now 

FROM scratch 

This pull the scratch image from dockerhub and creates an empty container

COPY --from=build /app /app

--from=build /app : here 
--from: go to the previous stage in dockerfile 
Build = name of the stage
/app = get the compiled file form the build stage 
/app = copy it on the container 

ENTRYPOINT ["/app"]
Execute the self-sufficent /app and deploy the artifact on the scratch container(empty Container)


Voting app dockerfiles:

https://github.com/Sonal0409/Docker-Stack-VotingApp/tree/main/example-voting-app
.net project - multistage

# vim dockerfile



# Base image to build the code
FROM mcr.microsoft.com/dotnet/sdk:7.0 as build

# Creates a directory with name /App
WORKDIR /App

# copy code form current folder to container /APP
COPY . ./

# Compile and build the code, place output in directory out
# output is going to be an artifact with name as Woker.dll
# Worker is the name given in program.cs file
# .net project build exptension is .dll

RUN dotnet publish -c release -o out


# app image - stage 2
# take image for run .net application
FROM mcr.microsoft.com/dotnet/runtime:7.0
WORKDIR /app
# fetch artifact from the previous build - Worker.dll
COPY --from=build /App/out .
ENTRYPOINT ["dotnet", "Worker.dll"]


Python project - multistage docker file explanation

# Define a base stage that uses the official python runtime base image
# first stage download the requirements

FROM python:3.11-slim AS base

# Set the application directory
WORKDIR /usr/local/app

# Install our requirements.txt
COPY requirements.txt ./requirements.txt
# this step will install all the dependencies mentione din requirements.txt file
RUN pip install -r requirements.txt


# Define the final stage that will bundle the application for production
# Take the above stage(Base) Image
# We will get container with python3.11 and requirements installed
FROM base AS final

# Copy our code from the current folder to the working directory inside the container
COPY . .

# Make port 80 available for links and/or publish
EXPOSE 80

# Define our command to be run when launching the container
# When the container starts, start gunicorn server, run app.py and method in it called as app,
# user cna access the app on port 80, generate logs and any user can access the logs using docker logs
# 4 parallel user req can be handled by gunicorn
# as user req is completed , break the connection
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]



Dangling Images


The images with name as <none>:<none> are dangling images 
These images are left over images after the multi stage build 
Only the last stage build is the final or actual image

Remove these dangling images:

# docker image prune 

# docker image prune -f

# docker image prune -a


===================================================================

Assignment : AWS -ECR -Recording

https://vimeo.com/781458714/3c31cf3073


CICD - JENKINS DOCKER PIPELINE:

FOR Your information:

The easiest way to run this application is simply to move the war file to your CATALINA_HOME/webapps directory. Tomcat will automatically expand and deploy the application for you. You can view it with the following URL (assuming that you're running tomcat on port 8080 as is the default):
<publicIP>:<oirtnumber>/addressbook

Run this command on terminal: With this command we are letting jenkins user to run docker commands 

# chmod -R 777 /var/run/docker.sock

ALSO WATCH THE RECORDING FOR HWO TO ADD CREDNETIALS FOR DOCKERHUB
==========================================================

pipeline{
    agent any
    tools{
        maven 'mymaven'
    }
    stages{
        stage('clone the repo'){
            steps{
                git 'https://github.com/Sonal0409/DevOpsCodeDemo.git'
            }
        }
        stage('Build Code'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('Build Image'){
            steps{
                sh 'docker build -t myjavajenkins .'
            }
        }
        stage('login and push Image'){
            steps{
                withCredentials([string(credentialsId: 'DOCKERHUB_TOKEN', variable: 'DOCKERHUB_PASSWORD')]) {
               // Now write the docker login comamand which will fetch apssword form variable DOCKERHUB_PASSWORD
                sh 'docker login -u sonal04 -p ${DOCKERHUB_PASSWORD}'
               }
               sh 'docker tag myjavajenkins sonal04/myjavajenkins'
               sh 'docker push sonal04/myjavajenkins'
               
            }
        }
        stage('Deploy the Image'){
            steps{
                sh 'docker run -d -P sonal04/myjavajenkins'
            }
        }
    }
}


CICD - GITHUB ACTIONS

https://github.com/Sonal0409/MavenBuild-Docker-GitHubActions.git


Demo:


