# React-Terraform
Application by Terraform

1. Dockerize the React Application
docker build -t trend-app:latest .
docker run -d -p 3000:80 --name trend-app trend-app:latest
Access application at: http://localhost:3000
docker ps

2. Push Image to Docker Hub
docker login
docker tag trend-app:latest <dockerhub-username>/trend-app:latest
docker push <dockerhub-username>/trend-app:latest

3. Configure kubectl for EKS
aws eks update-kubeconfig --region us-east-1 --name trend-eks-cluster
kubectl get nodes
kubectl get pods -A

4. Deploy to Kubernetes
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

5. Verify Deployment
kubectl get deployments
kubectl get pods
kubectl get svc
kubectl get svc trend-app-service

6. Jenkins Prerequisites for Terraform
Install Terraform on the Jenkins server:
