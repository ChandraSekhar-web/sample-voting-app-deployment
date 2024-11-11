# Voting Application Deployment on GCP using Kubernetes

This project demonstrates how to deploy a real-time voting application on Google Cloud Platform (GCP) using Kubernetes. The application is built on a microservices architecture and allows users to vote and view real-time results from anywhere with internet access.

## Project Overview

The Voting Application enables real-time voting and result display through a structured, microservices-based architecture. The architecture includes the following components:

1. **Voting-App Microservice**: Allows users to cast votes, which are sent to Redis for temporary processing.
2. **Redis**: Stores votes temporarily before forwarding them to the Worker microservice.
3. **Worker Microservice**: Processes votes received from Redis and stores them in PostgreSQL.
4. **PostgreSQL**: A database that securely stores votes for result aggregation.
5. **Result-App Microservice**: Displays real-time voting results by fetching data from PostgreSQL.

## Architecture

![Microservice Architecture](https://raw.githubusercontent.com/ChandraSekhar-web/sample-voting-app-deployment/refs/heads/main/microservice%20architecture.jpg)

## Setup and Installation

Follow the steps below to deploy the Voting Application on GCP using Kubernetes:

### 1. Fork the Repository
Fork the Voting Application repository:
[https://github.com/ChandraSekhar-web/sample-voting-app-deployment.git](https://github.com/ChandraSekhar-web/sample-voting-app-deployment.git)

### 2. Set up GCP
- Ensure your GCP account is activated and go to the GCP console.
- Create a Kubernetes cluster:
  - Go to **Kubernetes Engine** → **Create Cluster**.
  - Select **Standard Cluster** → **Location Type: Zonal** → **Number of Nodes: 2**.
  - Click **Create**.
  
### 3. Connect to Kubernetes Cluster
Once the cluster is created, connect to it via Cloud Shell:
- Select **Actions(⫶)** → **Connect** → **Copy the command** → **Run in Cloud Shell**.
- After authorization, use the Kubernetes CLI to create resources.

## Create Kubernetes Namespace
You first need to Create a namespace in Kubernetes to organize resources and manage resources within a Kubernetes cluster, create a `vote` namespace in Kubernetes:
```bash
kubectl create ns vote 
```

## Creating DB
- Next, you need to create the Database that will store the votes for the application.To store votes for the application, you need to create db-deployment and db-service resources by referring to the repository. 
- The application requires PostgreSQL version 9.4, accessible only to other microservices within the internal network (behind the firewall). It listens for connections on port 5432.
- The database will store its data in /var/lib/postgresql/data and needs persistent storage. A local directory mount will suffice; network-based persistent storage is not required.


### Follow these steps to set up the database:
1. Open the Cloud Shell editor and create the deployment resource as follows:
   - Use the following command to create the db-deployment.yml file by referring the  repository: 
     ```bash
     vi db-deployment.yml
     ```
   - After defining the deployment configuration, run the file with:
     ```bash
     kubectl apply -f db-deployment.yml
     ```
2. Similarly, for the service resource:
   - Use the following command to create the db-service.yml file by referring the  
     repository:
     ```bash
     vi db-service.yml
     ```
   - After defining the service configuration, run the file with:
     ```bash
     kubectl apply -f db-service.yml
     ```

## Creating Redis
- To collect new votes for the application, you need to create redis-deployment and redis-service resources.
- The application requires the most recent minimal version of Redis, accessible only to other microservices within the internal network (behind the firewall). 
  Redis listens for connections on port 6379.
- Redis stores its data in /data and only requires a local directory mount for  persistence; network-based storage is not necessary


### Follow these steps to set up the redis:
1. Create the deployment resource as follows:
   - Use the following command to create the redis-deployment.yml file by referring the repository:
     ```bash
     vi redis-deployment.yml
     ```
   - After defining the deployment configuration, run the file with:
     ```bash
     kubectl apply -f redis-deployment.yml
     ```
2.  Similarly, for the service resource:
    - Use the following command to create the redis-service.yml file by referring the   
      repository:
      ```bash
      vi redis-service.yml
      ```
    - After defining the service configuration, run the file with:
      ```bash
      kubectl apply -f redis-service.yml
      ```

## Creating Result Microservice:
- To present voting results to end-users, you need to create result-deployment and result-service resources.
- The application requires a specific version of the Result microservice (kodekloud/examplevotingapp_result:before), which should be accessible both internally to other microservices and externally to end-users.
- Configuration Details:
  - Internal Access: Other microservices will connect to the Result microservice using the name "result" on port 5001.
  - External Access: End-users should access the Result microservice on port 31001.
  - Application Port: The microservice itself listens on port 80.

### Follow these steps to configure the Result microservice:
1. Create the deployment resource as follows
   - Use the following command to create the result-deployment.yml file by referring the      repository:
     ```bash
     vi result-deployment.yml
     ```
   - After defining the deployment configuration, run the file with:
     ```bash
     kubectl apply -f result-deployment.yml
     ```
2. Similarly, for the service resource:
   - Use the following command to create the result-service.yml file by referring the repository:
     ```bash
     vi result-service.yml
     ```
   - After defining the service configuration, run the file with:
     ```bash
     kubectl apply -f result-service.yml
     ```

## Creating vote microservice
- The Vote microservice allows end-users to vote for their favorite person. To set it up, you need to create vote-deployment and vote-service resources.
- Configuration Details:
  - Image Version: The application requires kodekloud/examplevotingapp_vote:before
  - Internal Access: Other microservices in the Voting App will connect to the Vote microservice using the name "vote" on port 5000.
  - External Access: End-users should access the Vote microservice on port 31000.
  - Application Port: The microservice itself listens on port 80.

### Follow these steps to configure the Vote microservice:
1. Create the deployment resource as follows
   - Use the following command to create the vote-deployment.yml file by referring the repository:
     ```bash
     vi vote-deployment.yml
     ```
   - After defining the deployment configuration, run the file with:
     ```bash
     kubectl apply -f vote-deployment.yml
     ```
2. Similarly, for the service resource:
   - Use the following command to create the vote-service.yml file by referring the repository:
     ```bash
     vi vote-service.yml
     ```
   - After defining the service configuration, run the file with:
     ```bash
     kubectl apply -f vote-service.yml
     ```

## Creating the worker microservice:
- The Worker microservice is responsible for receiving votes and storing them in the database. This microservice does not need to be accessible to end-users.
- Configuration Details:
  - Image Version: The application requires kodekloud/examplevotingapp_worker.
  - External Access: The Worker microservice does not need to be accessible to end-users, so it will only be available internally within the cluster.
  - Internal Access: Other microservices can communicate with it as needed, but there is no need for external exposure.

### Follow these steps to configure the Worker microservice:
1. Create the deployment resource as follows
   - Use the following command to create the worker-deployment.yml file by referring the repository:
     ```bash
     vi worker-deployment.yml
     ```
   - After defining the deployment configuration, run the file with:
     ```bash
     kubectl apply -f worker-deployment.yml
     ```

## Debugging the voting application:
- Now, you should have a total of 5 Deployments and 4 Services at minimum. Create these objects using the Kubernetes CLI.
- To see all 5 deployments give command: 
  ```bash
  kubectl get deploy -n vote
  ```
- To see all 4 services give command:
  ```bash
  kubectl get svc -n vote 
  ```
## Usage
- Now to know the external ip address of nodes give command: 
  ```bash
  kubectl get node -o wide 
  ```
- For external access of the vote page give: <External IP address of 1st or 2nd  node>:31000
- For external access of the result page give: <External IP address of 1st or 2nd node>:31001
- At this point, you should be able to access the Voting App via port 31000 and 31001. However, if it isn't working properly, you need to debug.
- Now cast votes in vote pages from different google accounts by giving <external ip address of 1st or 2nd node>:31000  to register the votes and you can see the real-time result percentage  in the result page.

## Contributing
- Your contributions are highly valued! Simply fork the repository, make your improvements, and submit a pull request for review. 
