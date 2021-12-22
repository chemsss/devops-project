   
  
# Web application - DevOps Project

Welcome to the repository of our web application DevOps project. The goal of this project is to implement to a simple API web application that uses storages in a Redis database softwares that cover the whole DevOps cycle. The softwares help us automate the building, testing, deployment and running of our project.

In this repository is explained how to set up : 
* The User API web application
* A CI/CD pipeline with GitHub Actions and Heroku
* A Vagrant configured virtual machine provisionned with Ansible
* A Docker image of our application
* Container orchestration using Docker Compose
* Docker orchestration using Kubernetes
* A service mesh using Istio
* Monitoring with Prometheus and Grafano to the application containerized in a K8s cluster
  
  

# 1. Web application

The app is a basic NodeJS web application exposing REST API that creates and stores user parameters in a [Redis database](https://redis.io/).

## Functionality

1. Start a web server
2. Create a user
2. Get a user

## Installation

This application is written on NodeJS and it uses a Redis database.

1. [Install NodeJS](https://nodejs.org/en/download/)

2. [Install Redis](https://redis.io/download)

3. Install application

Go to the root directory of the application (where `package.json` file located) and run:

```
npm install 
```

## Usage

1. Start a web server

From the /userapi directory of the project run:

```
npm run start
```

It will start a web server available in your browser at http://localhost:3000.

2. Create a user

Send a POST (REST protocol) request using terminal:

```bash
curl --header "Content-Type: application/json" \
  --request POST \
  --data '{"username":"sergkudinov","firstname":"sergei","lastname":"kudinov"}' \
  http://localhost:3000/user
```

It will output:

```
{"status":"success","msg":"OK"}
```

Another way to test your REST API is to use [Postman](https://www.postman.com/).

## Testing

From the root directory of the project, run:

```
npm run test
```



# 2. CI/CD pipeline with GitHub Actions and Heroku

* The continuous integration workflow has been setup with GitHub Actions. 
The workflow automates building and tests of our NodeJS project. Before every deployment we check if the workflow tests have passed successfully to make sure the code runs fine. 

* The continuous deployment is done with Heroku. 

To create the workflow we went into the "Actions" tab of our project and created the workflow from the yaml template provided by GitHub.
![image](https://user-images.githubusercontent.com/61418782/147162931-0b970768-e4eb-456a-895a-647fc00f439f.png)


#DOCKER 

Dockerfile:

docker build -t userapi .

docker images

docker run -p 3000:3000 -d userapi


Docker Compose:

docker-compose up



