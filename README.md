**EKS DevOps Deployment Project 🚀**
This project demonstrates how to deploy a Spring Boot application on an Amazon EKS (Elastic Kubernetes Service) Cluster using Docker, Helm, and ArgoCD. The deployment process involves containerizing the application, pushing it to AWS ECR, and automating the deployment using Kubernetes and Helm charts.

**Project Overview**
This project follows a structured approach to deploy a Spring Boot application to EKS:

Set up an EKS Cluster using eksctl
Develop a Spring Boot application with a simple REST API
Containerize the application using Docker
Push the container image to AWS Elastic Container Registry (ECR)
Deploy the application to EKS using Helm
Use ArgoCD for continuous deployment

**1. Prerequisites**
Ensure you have the following tools installed before proceeding:

AWS CLI → aws --version
kubectl → kubectl version --client
eksctl → eksctl version
Docker → docker --version
Helm → helm version
ArgoCD (for GitOps deployment)

**2. Setting Up EKS Cluster**
To create an Amazon EKS cluster with t3.medium instances, use the following command:

sh
Copy
Edit
eksctl create cluster --name demo-cluster \
  --region us-east-1 \
  --nodegroup-name demo-nodes \
  --node-type t3.medium \
  --nodes 2
Once the cluster is created, verify the worker nodes:

sh
Copy
Edit
kubectl get nodes

**3. Spring Boot Application Development**
We create a simple Spring Boot REST API.

3.1 Create a Spring Boot Application
Generate the project using Spring Initializr:

sh
Copy
Edit
curl https://start.spring.io/starter.zip -d dependencies=web -d name=demo-app -o demo-app.zip
unzip demo-app.zip && cd demo-app

3.2 Add a Simple REST API
Edit DemoController.java and add the following code:

java
Copy
Edit
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class DemoController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello, DevOps!";
    }
}
3.3 Build and Run Locally
sh
Copy
Edit
mvn clean package
java -jar target/demo-app-0.0.1-SNAPSHOT.jar
Test API:

sh
Copy
Edit
curl http://localhost:8080/api/hello
Stop the application before containerizing it.

4. Dockerizing the Application
To deploy in EKS, we need to containerize the application.

4.1 Create Dockerfile
Create a Dockerfile inside demo-app:

Dockerfile
Copy
Edit
# Use an official OpenJDK runtime as a base image
FROM openjdk:17-jdk-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the built JAR file into the container
COPY build/libs/demo-0.0.1-SNAPSHOT.jar app.jar

# Expose port 8080 for the application
EXPOSE 8080

# Command to run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
4.2 Build and Test Docker Image
sh
Copy
Edit
docker build -t demo-app .
docker run -p 8080:8080 demo-app
Test API:

sh
Copy
Edit
curl http://localhost:8080/api/hello
5. Push Image to AWS ECR
sh
Copy
Edit
aws ecr create-repository --repository-name demo-app --region us-east-1
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 891612571458.dkr.ecr.us-east-1.amazonaws.com

docker tag demo-app:latest 891612571458.dkr.ecr.us-east-1.amazonaws.com/demo-app:latest

docker push 891612571458.dkr.ecr.us-east-1.amazonaws.com/demo-app:latest

**6. Deploy to Amazon EKS using Helm**

6.1 Create Helm Chart
sh
Copy
Edit
helm create demo-chart
Modify the values.yaml file:

yaml
Copy
Edit
image:
  repository: <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/demo-app
  tag: latest
  pullPolicy: Always

service:
  type: LoadBalancer
  port: 8080

livenessProbe:
  httpGet:
    path: /
    port: 8080
readinessProbe:
  httpGet:
    path: /
    port: 8080
6.2 Deploy the Application using Helm
sh
Copy
Edit
helm install demo-app ./demo-chart --namespace default
kubectl get pods -n default
kubectl get svc -n default
Once deployed, get the LoadBalancer URL:

sh
Copy
Edit
kubectl get svc -n default
Test API:

sh
Copy
Edit
curl http://ad7a27385fd9f4d8a982624f06491255-1676808059.us-east-1.elb.amazonaws.com:8080/api/hello

**7. Use ArgoCD for GitOps Deployment**
Instead of manually upgrading deployments, we use ArgoCD to automatically sync changes from GitHub to EKS.

7.1 Install ArgoCD
sh
Copy
Edit
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Check the status:

sh
Copy
Edit
kubectl get pods -n argocd
7.2 Access ArgoCD UI
sh
Copy
Edit
kubectl port-forward svc/argocd-server -n argocd 8080:443
Login using:

sh
Copy
Edit
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
7.3 Create an Application in ArgoCD
Create a new application in ArgoCD pointing to your GitHub repository where the Helm chart is stored.

Use the following configurations:

Repository URL: https://github.com/NagaAjay1812/EKS-DevOpsProject.git
Path: demo-chart
Cluster URL: https://0F48D49388D0A62500F65EBDAB61E765.gr7.us-east-1.eks.amazonaws.com
Sync Policy: Automatic
Once configured, ArgoCD will automatically deploy changes whenever you push new Helm charts.

Conclusion
This project covers: ✅ Creating an Amazon EKS Cluster
✅ Developing a Spring Boot API
✅ Containerizing the application with Docker
✅ Deploying the application using Helm & Kubernetes
✅ Automating deployments using ArgoCD

Now, your Spring Boot application is fully automated and running on AWS EKS! 🎉

