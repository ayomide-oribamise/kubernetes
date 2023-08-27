# kubernetes

This repository contains Kubernetes configuration files organized into subfolders for different applications. Each subfolder contains YAML scripts for deploying and configuring applications on a Kubernetes cluster.

## Prerequisites

1. **Kubernetes Cluster**: Ensure you have a functional Kubernetes cluster ready. If not, refer to the documentation of your chosen Kubernetes distribution for installation instructions.

2. **Kubectl**: Install the `kubectl` command-line tool to interact with your Kubernetes cluster.

## Structure

The repository is structured as follows:

```
kubernetes/
|-- subfolder1/
|   |-- README.md
|   |-- app.yaml
|   |-- service.yaml
|   |-- config.yaml
|   |-- secret.yaml
|-- subfolder2/
|   |-- README.md
|   |-- deployment.yaml
|   |-- service.yaml
|   |-- config.yaml
|   |-- secret.yaml
|-- ...
|-- README.md
```

Each application subfolder contains the following:

- `README.md`: Instructions for deploying and configuring the application using the provided YAML scripts.
- `app.yaml`: Deployment configuration for the application.
- `service.yaml`: Service configuration for the application.
- `config`: Config Map configuration for the application
- `secret.yaml`: Secrets configuration for the application

## Deployment Steps

For each subfolder, follow these general steps:

1. **Navigate to Sub Folder**: Enter the directory of the application you want to deploy.

2. **Read the README**: Open the `README.md` file in the subfolder to get detailed instructions for deploying and configuring the application.

3. **Apply YAML Files**: Use `kubectl apply` to deploy the application:

   ```
   kubectl apply -f app.yaml
   ```

   Replace `app.yaml` with the actual filenames.

4. **Accessing the Application**: If applicable, follow the instructions in the application's README to access the deployed application.

5. **Cleaning Up**: When done, delete the deployed resources:

   ```
   kubectl delete -f app.yaml
   ```

## Notes
