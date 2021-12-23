   
  
# Web application - DevOps Project

Welcome to the repository of our web application DevOps project. The goal of this project is to implement softwares covering the whole DevOps cycle to a simple API web application that uses storages in a Redis database. The softwares help us automate the building, testing, deployment and running of our project.

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

The app is a basic NodeJS web application exposing REST API where you can create and store user parameters in a [Redis](https://redis.io/) database.  
You can create a user by sending a curl POST method to the application with the user data, and access that data in the app in the http://localhost:3000/user route and adding to the route /username with the username corresponding the user data you want to access.  

## Installation

This application is written on NodeJS and it uses a Redis database.

1. [Install NodeJS](https://nodejs.org/en/download/)
2. [Install Redis](https://redis.io/download)
3. Install application

Go to the userapi/ directory of the cloned repository and run:

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
After, if you go to http://localhost:3000/user/sergkudinov, with "sergkudinov" being the username that you had in your POST data, it will display in the browser the following, with correspondance to the data that you posted:  
```
{"status":"success","msg":{"firstname":"sergei","lastname":"kudinov"}}
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

* The continuous deployment has been done with Heroku.  
Heroku helps us deploying our project and allows automatic deployment. We had to add Heroku to our GitHub Actions workflow.   
  
To create the workflow we went into the "Actions" tab of our project and created the workflow from the yaml template provided by GitHub.
![image](https://user-images.githubusercontent.com/61418782/147162931-0b970768-e4eb-456a-895a-647fc00f439f.png)
  
  
We then in the main.yaml have put this code:
```yaml
# This workflow will do a clean install of node dependencies, cache/restore them, build the source code and run tests across different versions of node
# For more information see: https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions

name: Main CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  # CI part
  test:
    runs-on: ubuntu-latest
    # Define `working-directory` if you application is in a subfolder
    defaults:
      run:
        working-directory: userapi
     # Service containers to run with `runner-job`
    services:
      # Label used to access the service container
      redis:
        # Docker Hub image
        image: redis
        ports:
          # Opens tcp port 6379 on the host and service container
          - 6379:6379
    strategy:
      matrix:
        node-version: [16.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/
    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
        cache-dependency-path: '**/package-lock.json'
    - run: npm ci
    - run: npm test
  # CD part
  deploy:
    needs: test # Requires CI part to be succesfully completed
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      # Read instructions how to configure this action: https://github.com/marketplace/actions/deploy-to-heroku#getting-started
      - uses: akhileshns/heroku-deploy@v3.12.12 # This is the action
        with:
          heroku_api_key: ${{secrets.HEROKU_API_KEY}}
          heroku_app_name: "warm-woodland-30605" # Must be unique in Heroku
          heroku_email: "chemseddine.brahimkhlil@edu.ece.fr" # Heroku account email
          appdir: userapi # Define appdir if you application is in a subfolder
```

  The heroku_app_name is the name of the Heroku app that we have created in Heroku's website. The API key was retreaved from the Heroku account setting and had to be added in the GitHub secrets. This allows for automatic deployment on Heroku after we push our project to GitHub.  
    
  We have also, in the Heroku app's deployment settings, enabled automatic deploys and ticked the "Wait for CI to pass before deploy" checkbox.  
    
We can check for ourselves the tests in the CI/CD workflow of our GitHub project: 
![image](https://user-images.githubusercontent.com/61418782/147166449-2814a5f8-87bf-4b9b-ae07-2d25d2f0c426.png)

After deployment, we can access in the Heroku website the deployment of our project:  
![image](https://user-images.githubusercontent.com/61418782/147166599-a15cf24b-c0d5-422e-b4f1-c4b72f9fb984.png)
  
![image](https://user-images.githubusercontent.com/61418782/147166662-b78044d1-e7a0-4ec9-904b-3f7c19c4e3a5.png)  

However, we have only our NodeJS app and not the Redis database, so the deployment on Heroku is only partially functionnal, as the user API does not work.  
While the Redis service is free on Heroku, it requires adding credit card information, that is why we have not bothered adding it.  
  
  
  
  

# 3. Configuring and provisionning a virtual environment using the IaC approach

To use the Infrastructure as code (IaC) approach, we have used Vagrant to configure and manage our virtual machine and used Ansible to provision the virtual machine. 


## Installation

For this, in addition to installing Vagrant, you have to make sure you have installed VirtualBox (or another [virtualization software that is accepted by Vagrant](https://www.vagrantup.com/docs/providers)).

1. [Install VirtualBox](https://www.virtualbox.org/wiki/Downloads) (or other)
2. [Install Vagrant](https://www.vagrantup.com/downloads.html)  
  

## Creating and provisionning the virtual machine (VM)  

* Go to the [/IaC](https://github.com/chemsss/devops-project/tree/main/IaC) directory (where there is the [Vagrantfile](https://github.com/chemsss/devops-project/blob/main/IaC/Vagrantfile)) and run in the terminal:

```bash
vagrant up
```  
  It should start initializing and booting the VM.  
    
  The VM's OS is hashicorp/bionic64, which is a basic, highly optimized and small in size version of Ubuntu 18.04 64-bit available for minimal use cases and made by HashiCorp. You can choose whatever OS you want to use in your VM by modifying the VM box property. Ressouces are available online about the [Vagrantfile](https://www.vagrantup.com/docs/vagrantfile) and how to change [boxes](https://www.vagrantup.com/docs/boxes).  
    
  Then it will download automatically Ansible and will start the provisionning set up by the Ansible playbooks. The playbooks' tasks set up the downloading and enabling of the packages and services that are needed to run the userapi project on the VM. 
    However, the installation of the Redis database is not complete as only its downloading could work for us but not its installation.  
  
* After the downloads have ended, you can enter your VM via SSH with the following Vagrant command:  
```bash
vagrant ssh nodejs_server
```  
  
  The folder userapi located in the repository's clone of your host is shared with the VM thanks to the "synced_folder" folder property in the [Vagrantfile](https://github.com/chemsss/devops-project/blob/main/IaC/Vagrantfile).  
  * When connecting by SSH, you can find the folder by typing the following commands in the terminal: 
  
```bash
cd ../..
cd home/userapi/
/home/userapi ls
```  
You can see that the files being showen by the terminal are the same than the one in the host's folder.   
   
![image](https://user-images.githubusercontent.com/61418782/147171272-2b733fe4-e324-4a06-ad7d-a585d11cc2ea.png)


You can keep working in the host's folder and the guest machine's synced folder will keep automatically the files up to date with the host's files.

![image](https://user-images.githubusercontent.com/61418782/147171866-d86bce9f-7f1a-4ecd-9912-a6b84af06440.png)  ![image](https://user-images.githubusercontent.com/61418782/147171552-6c54de3e-7388-4065-b715-56f5880a12a4.png)  
  

We have also kept in the roles folders a "main.yaml" file with tasks for the installation and launch of GitLab on the VM that works fine. If you want to use it you have to add a role with the path of the tasks file and with the "install" tag in the "run.yml" file in the playbooks folder. When installed and launched on the VM, you will be able to access the GitLab page through the 20.20.20.2 address on your host machine thanks to the server.vm.network and ip properties in the [Vagrantfile](https://github.com/chemsss/devops-project/blob/main/IaC/Vagrantfile).






# 4. Docker image of the app

To be able to "containerize" our application we created a Docker image of it. Docker enables us to run our app in the exact environment(s) that we want.

## Installation
Install [Docker Desktop](https://www.docker.com/get-started)  
  

* In the root directory of the repository's clone (where there is the [Dockerfile](https://github.com/chemsss/devops-project/blob/main/Dockerfile)), run the following command to build image (don't forget the dot):
```bash
docker build -t userapi .
```    
  
We have also pushed our Docker image to DockerHub.  
![image](https://user-images.githubusercontent.com/61418782/147173441-fe8ed732-6454-4147-ae47-04f358c9d5b9.png)  

* So, instead, you can simply pull the image from DockerHub: 
```bash
docker pull chemss/userapi
```    

* You can check if it appears in your local Docker images:
```bash
docker images
```    

* Then, run the container:
```bash
docker run -p 3000:3000 -d chemss/userapi
```   
  
* Check if it the container is running:
```bash
docker ps
```  

* By going to http://localhost:3000/, the welcome page of the app will showup:  

![image](https://user-images.githubusercontent.com/61418782/147173843-253018e9-31e8-4b6f-a46b-7d1e1212a36b.png)  


* Stop the container:
```bash
docker stop <CONTAINER_ID>
```  



# 5. Container orchestration using Docker Compose

The image we have built with the Dockerfile runs only a container which has our app but not the database.  
  
Docker Compose allows us to run multi-container Docker applications. The services and images are set up in the [docker-compose.yaml](https://github.com/chemsss/devops-project/blob/main/docker-compose.yaml) file.  

* Run the docker-compose command to create and start the redis and web services from the configuration file:  
```bash
docker-compose up
```   

* By going to http://localhost:3000/, the welcome page of the app will showup:  

![image](https://user-images.githubusercontent.com/61418782/147173843-253018e9-31e8-4b6f-a46b-7d1e1212a36b.png)  

* You can delete the containers with:
```bash
docker-compose rm
```   




# 5. Docker orchestration using Kubernetes 

Kubernetes is an open-source system for automating the deployment, scaling and management of containerized applications. Compared to Kubernetes, Docker Compose has limited functionnality.  
  

## Install Minikube
Minikube is a tool that makes it easy tu run Kubernetes locally. 
  
[Install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) following the instructions depending on your OS.  

* Start Minikube: 
```bash
minikube start
```   

* Check that everything is OK:
```bash
minikube status
```  

## Running the Kubernetes deployments

* Go to the [/k8s](https://github.com/chemsss/devops-project/tree/main/k8s) directory and run this command for every file:
```bash
kubectl apply -f <file_name.yaml>
```  
* The deployment.yaml file describes the desired states of the redis and userapi deployments.  
* The service.yaml file exposes the redis and userapi apps as network services and * gives them the right ports.
* The persistentvolume.yaml file creates a piece of storage in the cluster which has a lifecycle independent of any individual Pod that uses the PersistentVolume.
* The persistentvolumeclaim.yaml file create a request for storage by a user.  
  
## Check that everything is running 
* Check that the deployments are running:
```bash
kubectl get deployments
```  
The result should be as following:  
  
![image](https://user-images.githubusercontent.com/61418782/147182096-6c5e4f86-72a7-4062-9323-3893180c6db7.png)  

* Check that the services are running:
```bash
kubectl get services
```
Should output the following:
  
![image](https://user-images.githubusercontent.com/61418782/147182248-561421d0-020a-40ad-8d07-1e9589da29fd.png)  
  
* Check that the PersistentVolume is running:
```bash
kubectl get pv
```
Outputs the following:
  
![image](https://user-images.githubusercontent.com/61418782/147182440-566d5b12-591d-45ae-9f51-0a21d39ce965.png)  
  
* Check that the PersistentVolumeClaim is running:
```bash
kubectl get pvc
```
Outputs the following:  
  
![image](https://user-images.githubusercontent.com/61418782/147182537-ece8b5ed-d51c-4335-b89d-299ea912583c.png)    
    
We can see in the outputs that the PersistentVolumeClaim is bound to the PersistentVolume. The claim requests at least 3Gi from our hostPath PersistentVolume.  
  

* You can also check that everything is running through the minikube dashboard:  
```bash
minikube dashboard
```  
  
![image](https://user-images.githubusercontent.com/61418782/147188835-41bea159-c928-4b9a-a724-cccbb7f96d08.png)



## Accessing the containerized app

* Run the following command to the userapi service:
```bash
 kubectl port-forward service/userapi-deployment 3000:3000
```  
  
The home page of our app should display when going to http://localhost:3000/ on your browser.

* Run the following command:
```bash
kubectl get pods
```  
Outputs the following:
   
![image](https://user-images.githubusercontent.com/61418782/147184947-bfedbd4a-1e16-4a56-95d0-441fd79f991c.png)  
  
* You can send a bash command to one of the 3 pod replicas created with the userapi deployment with the following command:
```bash
 kubectl exec <POD_NAME> -- <COMMAND>
 #or
 kubectl exec -it <POD_NAME> -- <COMMAND>
```  
  


