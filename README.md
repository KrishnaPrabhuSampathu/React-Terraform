# React-Terraform

Jenkins server: jenkins-app-server

Application by Terraform


Developer Pushes Code to GitHub
            ↓
       GitHub Webhook t
            ↓
          Jenkins
            ↓
   ┌─────────────────────┐
   │ 1. Checkout Code    │
   │ 2. Terraform Apply  │
   │ 3. Build React App  │
   │ 4. Build Docker Img │
   │ 5. Push to DockerHub│
   │ 6. Deploy to EKS    │
   └─────────────────────┘

Jenkins Server
1. Install Jenkins On Ubuntu EC2

http://<ec2-public-ip>:8080
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Jenkins Plugins to Install

Configure Jenkins Tools and Credentials
    Credentials Required
    GitHub Personal Access Token
    Docker Hub Username/Password
    AWS Access Key and Secret Key
    kubeconfig (optional)
Add under: Manage Jenkins → Credentials

Configure GitHub Webhook
   http://<jenkins-public-ip>:8080/github-webhook/
   
2. Jenkins Prerequisites for Terraform
    Install Terraform on the Jenkins server:
    Docker
    AWS CLI
    kubectl
    Node.js (if needed for tests)
    Git
    Java (for Jenkins itself)

----------------------------------------------------------
Required flow:

PHASE 1: Terraform Infrastructure
 create main.tf that provisions:
    VPC
    IAM roles
    EC2 instance (Jenkins server)

PHASE 2: Jenkins Setup
On EC2 (created by Terraform):
 Install:
    Jenkins
    Docker
    Git
    Kubernetes CLI (kubectl)

 Install Jenkins plugins:
    Git
    Docker
    Pipeline
    Kubernetes    

PHASE 3: Kubernetes on AWS EKS  
 Create EKS cluster (Terraform / eksctl / console)
 Verify cluster:  
    kubectl get nodes


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
 EXTERNAL-IP: a1b2c3d4e5.us-east-1.elb.amazonaws.com


6. Monitoring Setup (Prometheus + Grafana)
Install via Helm
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  helm repo update
  helm install monitoring prometheus-community/kube-prometheus-stack
kubectl get pods -n default
kubectl port-forward svc/monitoring-grafana 3001:80
Open: http://localhost:3001
 Default username: admin
 Retrieve password:
  kubectl get secret monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
    

-------------------------------------------