# Biosoftv8 code deployment
This project is a sample application built with PHP and Laravel. It demonstrates how to use Docker and various AWS cloud services, such as Amazon EKS, Amazon Fargate, Amazon EC2 as load balancers, and Amazon Elastic Container Registry (ECR) to deploy and run the application in a production environment.


## Technologies Used
- PHP
- Laravel
- Docker
- AWS EKS
- AWS Fargate
- AWS EC2
- AWS ECR
- Getting Started

## Getting Started
1. Clone the repository (ask for credentials):
```bash 
git clone https://bitbucket.org/BiosoftCL/biosoft-core/src/
```
2. Build the Docker image:

```bash
docker build -t biosoft_image .
```

3. Push the Docker image:
This will push the image to the Docker registry, making it available for use by others who have access to the registry.
```bash
docker login
docker tag biosoft_image docker.io/my-repo/biosoft-image
docker push docker.io/my-repo/biosoft-image
```
## Usage
The following AWS services are used to deploy the Biosoftv8 application:

- Amazon Elastic Kubernetes Service (EKS) to manage the Kubernetes cluster
- Docker Container Registry (DCR) to store the Docker containers
- Amazon Elastic Compute Cloud (EC2) and AWS Fargate to serve as load balancers
- To deploy the Biosoftv8 application, follow these steps:

**Create an EKS cluster in your AWS account**.
- Install the kubectl command-line tool and configure it to connect to your EKS cluster.
- Create an DCR repository to store the Docker images.
- Build the Docker image for the Biosoftv8 application and push it to the ECR repository.
- Apply the Kubernetes configuration files in this repository to deploy the Biosoftv8 application.
- Kubernetes Configuration

The `deployment.yaml` file contains the Kubernetes Deployment configuration for the Biosoftv8 application. It specifies the Docker container image to use, the number of replicas to run, and the ports to expose.

The `autoscaler.yaml` file contains the Kubernetes Horizontal Pod Autoscaler configuration for the Biosoftv8 application. It specifies the scaling rules based on CPU utilization.

The `service.yaml` file contains the Kubernetes Service configuration for the Biosoftv8 application. It specifies the load balancer type and the port to expose.

## Deployment

To deploy the Biosoftv8 application, apply the Kubernetes configuration files in `k8_biosoft` directory of this repository:
```bash
$ kubectl apply -f deployment.yaml
$ kubectl apply -f autoscaler.yaml
$ kubectl apply -f service.yaml
```
This will create a Kubernetes Deployment, a Horizontal Pod Autoscaler, and a LoadBalancer Service for the Biosoftv8 application.
