# Udacity-Udagram-Microservice
This repository is associated with Cloud Developer ND - Course 03 - Monolith to Microservices. There are 6 lessons in this course. There is separate directory for each lesson.
Clone the starter code from here:
https://github.com/udacity/nd990-c3-microservices-v1

## About the Project - Udagram Image Filtering Microservice
Udagram is a simple cloud application developed alongside the Udacity Cloud Engineering Nanodegree. It allows users to register and log into a web client, post photos to the feed, and process photos using an image filtering microservice. Following are the services involved in this project:

* “user” - allows users to register and log into a web client, 
* “feed” - allows users to post photos, and process photos using image filtering 
* “frontend” - acts as an interface between the user and the backend-services
* "reverseproxy" - For resolving multiple services running on same port in separate containers

Correspondingly, the project is split into following parts:
1. The RestAPI Feed Backend, a Node-Express feed microservice.
1. The RestAPI User Backend, a Node-Express user microservice.
1. The Simple Frontend - A basic Ionic client web application which consumes the RestAPI Backend.
1. Nginx as a reverse-proxy server, when different backend services are running on the same port, then a reverse proxy server directs client requests to the appropriate backend server and retrieves resources on behalf of the client.  


## Project:

Project consists of multiple parts:

### 1. Convert monolith udagram application to microservice

Applying the rules and conventions of microservices and keeping in mind the advantages and disadvantages of both monolith and microservice, I've split the udagram monolith application's backend restApi implementation into 2 parts - Feed and User.
We already had frontend separate and we add a nginx proxy - reverseproxy service to handle and route the incoming traffic to the appropriate service. Nginx proxy configuration is specified in nginx.conf file.

### 2. Building the services locally:

After separting the 2 services and resolving the code dependencies and setting environment variables in profile, I have installed npm packages using npm i.
Post that, I've run npm run dev for the service to test them locally. After that I ran the services as npm run build to check for any other errors in the build process.
The application won't run in local as both the service are configured to run on port 8080. To resolve this, we use the reverseproxy. After we run the reverseproxy service, the application starts working in localhost in UI.

![Alt text](/screenshots/Local-run.PNG "Local Deployment")

### 3. Build Docker Images and push to docker registry:

Once dev test is complete, we will build the docker images. After installing docker, I verified using: 

>$ docker -v

Docker version: 19.03.8, build afacb8b7f0 

I built the docker images individually first using:

>$ docker build -t user .

I verified that the images were created using:

>$ docker images

![Alt text](/screenshots/docker-images.PNG "Docker Images")

Started a container using:

>$ docker run --rm --publish 8080:8080 -v $HOME/.aws:/root/.aws --env POSTGRESS_HOST=$POSTGRESS_HOST --env POSTGRESS_USERNAME=$POSTGRESS_USERNAME --env POSTGRESS_PASSWORD=$POSTGRESS_PASSWORD --env POSTGRESS_DATABASE=$POSTGRESS_DATABASE --env AWS_REGION=$AWS_REGION --env AWS_PROFILE=$AWS_PROFILE --env AWS_MEDIA_BUCKET=$AWS_MEDIA_BUCKET --env JWT_SECRET=$JWT_SECRET --name feed rahulpal210991/udacity-restapi-feed

Finally, I configued docker-compose and built all the images together using docker-compose using:

>$ docker-compose -f docker-compose-build.yaml build

Started the application using:

>$ docker-compose up

![Alt text](/screenshots/docker-compose%20up.PNG "Docker compose up")

Verified my changes. Then I pushed the docker images to my docker repository:

>$ docker push rahulpal210991/udacity-restapi-feed

![Alt text](/screenshots/dockerhub.PNG "DockerHub")



### 4. Deploy into Kubernetes:

I setup AWS EKS cluster and added 2 nodes in node group. Configured kubectl in my local and updated kubeconfig to interact with my k8s cluster.

![Alt text](/screenshots/eks-cluster.PNG "AWS EKS Cluster")

![Alt text](/screenshots/nodegrp.PNG "Worker Nodes")

I created the env-configMap.yaml, aws-secret.yaml and env-secret.yaml files for configuration of environment variables and credentials which will be used by the containers running on pods.
Created the deployment and service yaml files for the 4 microservices - reverseproxy, frontend, feed and user. 
After applying the secrets and env files, the deployment files were applied to create the pods on worker nodes using:

>$ kubectl apply -f backend-feed-deployment.yaml

Applied the service configuration on top of it.

>$ kubectl apply -f backend-feed-service.yaml

Did the same for all the 4 services. Then checked the environment using the following commands:

```
$ kubectl get nodes

$ kubectl get pods

$ kubectl get rs

$ kubectl get svc

$ kubectl get deploy
```

![Alt text](/screenshots/k8s-details.PNG "Kuberneter Commands")


Finally, started the UI and verified that the application is working.

In order to verify rolling update, updated a deployment yaml file with updated version and applied that deployment.
Old pods got terminated and new pods were spring up. Old replicaSet still exists with 0 pods giving the option of rollback.

![Alt text](/screenshots/rolling-update.PNG "Rolling Update")

### 5. Continuous Integration using Travis CI:

Added the .travis.yml file in the root scope of the project which lists the steps to execute for build and deploy.
Travis CI is integrated with my Github profile and so when this file is checked into Github, build is triggered automatically which build the project, builds the docker images and then finally pushed the docker images to Docker repository.

![Alt text](/screenshots/travis-1.PNG "TravisCI UI")

![Alt text](/screenshots/UI.PNG "Application UI")
