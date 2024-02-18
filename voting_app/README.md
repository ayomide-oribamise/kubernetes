# Introduction
The Cloud Native Voting Application is a web-based system designed to facilitate voting on programming languages. It comprises a frontend user interface, a backend API, and a MongoDB database for data storage.

## Application Components:

### Frontend:

Developed in React, the frontend provides the user interface for the application.
Users can view the available programming languages and vote for their preferred language by clicking on the corresponding buttons.
The frontend interacts with the backend API to record user votes and retrieve language information.

### API (Backend):

Developed in Go, the API serves as the backend for the application.
It exposes CRUD-based endpoints for managing programming languages, such as retrieving language details and voting.
The API communicates with the MongoDB database to store and retrieve language data.

### MongoDB Database:

MongoDB is used as the persistence layer for the application.
It stores information about programming languages, including their names, details, and vote counts.
The database is deployed as a replica set for high availability and reliability.

## Deployment Process:

### Follow Along:

To deploy the Cloud Native Voting Application:

1. Clone this repository to your local machine.
2. Follow the steps below

### Prerequisite
Ensure that you already have a running cluster created, and your kubeconfig updated.

To deploy the Cloud Native Voting Application, follow these steps:

### Namespace Creation:

*Create a Kubernetes namespace.* 

1. Replace all instances of "ayoori" in **namespace.yaml** with your desired namespace name to contain all application resources.
2. Run this code:

```
kubectl create -f namespace.yaml
kubectl config set-context --current --namespace ayoori
```

### MongoDB Deployment:
*Deploy a MongoDB replica set consisting of three pods.*

1. Change the namespace name in **mongo-stateful.yaml** to match your defined namespace
2. You can change the number of replicas and request settings to match your need.
3. Run the command below

```
kubectl create -f mongo-stateful.yaml
```

* Create a service for MongoDB to enable communication within the cluster*

1. Change the namespace name in **mongo-service.yaml** to match your defined namespace
2. Run the command below

```
kubectl create -f mongo-service.yaml
```

*Initialize the Mongo DB Replica set by running the command below*
```
cat << EOF | kubectl exec -it mongo-0 -- mongo
rs.initiate();
sleep(2000);
rs.add("mongo-1.mongo:27017");
sleep(2000);
rs.add("mongo-2.mongo:27017");
sleep(2000);
cfg = rs.conf();
cfg.members[0].host = "mongo-0.mongo:27017";
rs.reconfig(cfg, {force: true});
sleep(5000);
EOF
```

*Load initial data into the DB. Run the command below in your terminal*
```
cat << EOF | kubectl exec -it mongo-0 -- mongo
use langdb;
db.languages.insert({"name" : "csharp", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 5, "compiled" : false, "homepage" : "https://dotnet.microsoft.com/learn/csharp", "download" : "https://dotnet.microsoft.com/download/", "votes" : 0}});
db.languages.insert({"name" : "python", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 3, "script" : false, "homepage" : "https://www.python.org/", "download" : "https://www.python.org/downloads/", "votes" : 0}});
db.languages.insert({"name" : "javascript", "codedetail" : { "usecase" : "web, client-side", "rank" : 7, "script" : false, "homepage" : "https://en.wikipedia.org/wiki/JavaScript", "download" : "n/a", "votes" : 0}});
db.languages.insert({"name" : "go", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 12, "compiled" : true, "homepage" : "https://golang.org", "download" : "https://golang.org/dl/", "votes" : 0}});
db.languages.insert({"name" : "java", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 1, "compiled" : true, "homepage" : "https://www.java.com/en/", "download" : "https://www.java.com/en/download/", "votes" : 0}});
db.languages.insert({"name" : "nodejs", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 20, "script" : false, "homepage" : "https://nodejs.org/en/", "download" : "https://nodejs.org/en/download/", "votes" : 0}});

db.languages.find().pretty();
EOF
```

*Store MongoDB connection credentials securely using a Kubernetes Secret. (This is not a safe way to store secrets, this is for demo purposes)*
1. Encrypt your mongodb username and password in base 64. Replace "admin" in the code snippet below with your username and "password" with what you want your password to be. Copy the values and keep them.

```
echo -n 'admin' | base64
echo -n 'password' | base64
```

2. Change the namespace name in **mongo-secret.yaml** to match your defined namespace
3. Replace the values in lines 7 and 8 of **mongo-secret.yaml** with your stored encrypted values from (1) above.
4. Run the command below

```
kubectl create -f mongo-secret.yaml
```

### API Deployment:

*Deploy the API backend as a set of pods to handle requests.*

1. Change the namespace name in **api-deploy.yaml** to match your defined namespace
2. Replace container image with one of your choice in **api-deploy.yaml**
3. Run the command below

```
kubectl create -f api-deploy.yaml
```
*Create a LoadBalancer service to expose the API externally.*

1. Run the command below in your terminal to expose the deployment created above as a service

```
kubectl expose deploy api \
 --name=api \
 --type=LoadBalancer \
 --port=80 \
 --target-port=8080
```

### Frontend Deployment:

*Obtain the Fully Qualified Domain Name of the API Loadbalancer*

1. Obtain the FQDN to tell the frontend where to send the API calls by running this command:

```
{
API_ELB_PUBLIC_FQDN=$(kubectl get svc api -ojsonpath="{.status.loadBalancer.ingress[0].hostname}")
echo API_ELB_PUBLIC_FQDN=$API_ELB_PUBLIC_FQDN
}
```

*Deploy the frontend user interface as a set of pods.*

1. Change the namespace name in **frontend-deploy.yaml** to match your defined namespace
2. Replace container image with one of your choice in **frontend-deploy.yaml**
3. Run the command below

```
kubectl create -f frontend-deploy.yaml
```

*Create a LoadBalancer service to expose the frontend externally.*

1. Run the command below in your terminal to expose the deployment created above as a service

```
kubectl expose deploy frontend \
 --name=frontend \
 --type=LoadBalancer \
 --port=80 \
 --target-port=8080
```

### Test your newly deployed end to end service

*Obtain the URL of your deployed application*

1. Type this command into the terminal. Copy the terminal output

```
echo http://$FRONTEND_ELB_PUBLIC_FQDN
```

2. Go to your browser and paste in the copied output from the step above.

## Additional Steps:

1. Monitor the health and performance of the deployed resources using Kubernetes and other monitoring tools.
2. Explore scaling options for the frontend and API components based on traffic patterns.
3. Consider implementing authentication and authorization mechanisms for secure access to the application.


By following these steps, you can deploy and interact with the Cloud Native Voting Application, gaining insights into cloud-native development practices and Kubernetes deployment workflows.