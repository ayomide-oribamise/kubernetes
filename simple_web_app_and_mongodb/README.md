# Deploying Web App with MongoDB on Kubernetes

This guide outlines the steps to configure and deploy a web application along with a MongoDB database on a Kubernetes cluster using the provided Kubernetes YAML files.

## Prerequisites

Kubernetes Cluster: Ensure you have a working Kubernetes cluster up and running.
Kubectl: Install the kubectl command-line tool to interact with your Kubernetes cluster.
Docker: Install Docker to build and push Docker images.

## Configuration

### 1. Encode Your Secret Data 
For sensitive data like your DB credentials, encode these using base64 encoding. base64 encoding is a basic obfuscation method. For stronger security, explore encryption options and secrets management tools. In a terminal, run:

```
echo -n 'your-mongo-username' | base64
echo -n 'your-mongo-password' | base64
```

### 2. Create Secret YAML:
Create a file named mongo-secret.yaml with the following content:

```
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque
data:
  mongo-user: base64-encoded-username
  mongo-password: base64-encoded-password
```

***Replace base64-encoded-username and base64-encoded-password with the outputs from the base64 encoding commands.***

### 3. Create ConfigMap YAML:

Create a file named mongo-config.yaml with the following content:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-service
data:
  mongo-url: your-mongo-url
```

***Replace your-mongo-url with your actual MongoDB URL.***

### 4. Apply the Secret

Apply the Secret to your cluster by running this command in your terminal:

```
kubectl apply -f mongo-secret.yaml
```

Now you have a Kubernetes Secret named mongo-secret with your MongoDB credentials securely encoded. You can reference this Secret in your deployment configurations.

### 5. Apply the ConfigMap:
Apply the ConfigMap to your cluster by running this command in your terminal:

```
kubectl apply -f mongo-config.yaml
```

Now you have a Kubernetes ConfigMap named mongo-service with your MongoDB URL. You can reference this ConfigMap in your deployment configurations.

ConfigMaps are useful for storing non-sensitive configuration data. If you need to store sensitive data, consider using Secrets instead.

Remember that ConfigMaps and Secrets make managing configuration and sensitive data easier in Kubernetes deployments.

### 6. Create MongoDB YAML File
Create a file named mongo-deployment.yaml with the following content:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  labels:
    app: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:5.0.20
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-user
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-password

---

apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  selector:
    app: mongo
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

### 7. Update MongoDB Image
In the mongo-deployment.yaml file, update the image tag to match your desired version of MongoDB:

```
 containers:
      - name: mongo
        image: mongo:<your-desired-version>
```

### 8. Create Web App YAML File
Create a file named webapp-deployment.yaml with the following content:

```
# Define the API version for the Kubernetes resource.
apiVersion: apps/v1
# Specify the type of Kubernetes resource being created, in this case, a Deployment.
kind: Deployment
# Metadata includes information about the resource, like its name and labels.
metadata:
  # Name of the Deployment resource.
  name: webapp-deployment
  # Labels associated with the Deployment.
  labels:
    app: mongo
# Spec section defines the desired state of the Deployment.
spec:
  # Number of desired replicas for the Deployment.
  replicas: 1
  # Selector to match the Pods controlled by this Deployment.
  selector:
    matchLabels:
      app: webapp
  # Template for the Pods created by this Deployment.
  template:
    metadata:
      # Labels for the Pods.
      labels:
        app: webapp
    # Spec for the Pods.
    spec:
      # Containers section defines the container(s) within the Pod.
      containers:
        - name: webapp
          # Docker image to use for the container.
          image: nanajanashia/k8s-demo-app:v1.0
          # Ports to expose on the container.
          ports:
            - containerPort: 3000
          # Environment variables for the container.
          env:
            - name: USER_NAME
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: mongo-user
            - name: USER_PWD
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: mongo-password
            - name: DB_URL
              valueFrom:
                configMapKeyRef:
                  name: mongo-service
                  key: mongo-url

              
---

# Define the API version for the Kubernetes resource.
apiVersion: v1
# Specify the type of Kubernetes resource being created, in this case, a Service.
kind: Service
# Metadata includes information about the resource, like its name and labels.
metadata:
  # Name of the Service resource.
  name: webapp-service
# Spec section defines the desired state of the Service.
spec:
  # Type of the Service (NodePort in this case).
  type: NodePort
  # Selector to target the Pods for the Service.
  selector:
    app: webapp
  # Ports to expose on the Service.
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      # Node port for external access.
      nodePort: 30000
```

### 9. Update Web App Image
In the webapp-deployment.yaml file, update the image tag to match the version of your web app:

```
containers:
  - name: webapp
    image: <your-webapp-image>:<your-image-version>
```

### 10. Update nodePort (optional):
In the webapp-deployment.yaml file, you can update the nodePort. A NodePort in Kubernetes is a type of service that exposes an application to the outside world or to other services within the cluster.

Kubernetes assigns a port from a predefined range (e.g., 30000-32767) on each node in the cluster. You can choose a port value within this range that suits your needs.

To update this port number, simply choose within the range 30000-32767:

```
- protocol: TCP
      port: 3000
      targetPort: 3000
      # You can update this port number
      nodePort: 30000
```
## Deployment

### 1. Deploy MongoDB
Apply the MongoDB deployment and service YAML file:

```
kubectl apply -f mongo-deployment.yaml
```

### 2. Deploy Web App
Apply the web app deployment and service YAML files:

```
kubectl apply -f webapp-deployment.yaml
```

## Accessing the Web App

Retrieve the NodePort assigned to the webapp-service:

```
minikube ip
```

Access the web app by opening a web browser and navigating to http://<your-cluster-ip>:<node-port>. The <your-cluster-ip> is usually the IP of one of your cluster nodes, and <node-port> is the NodePort specified in the webapp-service.yaml file.

## Cleaning Up

To view all your running kubernetes components, run:

```
kubectl get all
```
To clean up the deployed resources, run:

```
kubectl delete deployment webapp-deployment
kubectl delete service webapp-service
kubectl delete deployment mongo-deployment
kubectl delete service mongo-service
kubectl delete secret mongo-secret
kubectl delete configmap mongo-service
```

## Conclusion

You've successfully deployed a web application with MongoDB on Kubernetes using the provided YAML files. Feel free to customize the configurations according to your needs.

Please note that this README is a template, and you should replace placeholders such as <your-mongo-username>, <your-mongo-password>, <your-mongo-url>, <your-webapp-image>, <your-image-version>, <your-cluster-ip>, and <node-port> with your actual values.





