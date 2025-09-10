# Reddit Clone App on Kubernetes with Ingress
This project demonstrates how to deploy a Reddit clone app on Kubernetes with Ingress and expose it to the world using Minikube as the cluster.
Below is an overview of the architecture of this Reddit Clone App running on Kubernetes with Ingress.
![Architecture Diagram](https://github.com/LondheShubham153/reddit-clone-k8s-ingress/assets/71492927/e1eec5f2-1983-445b-8966-e9acfdea7f8e)

## Prerequisites
Before you begin, you should have the following tools installed on your local machine: 

- Docker
- Minikube cluster ( Running )
- kubectl
- Git

You can install Prerequisites by doing these steps. [click here & complete all steps one by one]().


## Installation
Follow these steps to install and run the Reddit clone app on your local machine:

1) Clone this repository to your local machine: `git clone https://github.com/LondheShubham153/reddit-clone-k8s-ingress.git`
2) Navigate to the project directory: `cd reddit-clone-k8s-ingress`
3) Build the Docker image for the Reddit clone app: `docker build -t reddit-clone-app .`
4) Deploy the app to Kubernetes: `kubectl apply -f deployment.yaml`
1) Deploy the Service for deployment to Kubernetes: `kubectl apply -f service.yaml`
5) Enable Ingress by using Command: `minikube addons enable ingress`
6) Expose the app as a Kubernetes service: `kubectl expose deployment reddit-deployment --type=NodePort --port=5000`
7) Create an Ingress resource: `kubectl apply -f ingress.yaml`


## Test Ingress DNS for the app:
- Test Ingress by typing this command: `curl http://domain.com/test`

## Cluster Monitoring using Prometheus & Grafana

Key Components :

- Prometheus server - Processes and stores metrics data
- Alert Manager - Sends alerts to any systems/channels
- Grafana - Visualize scraped data in UI

Pre Requisites :
- EKS Cluster is setup already
- Install Helm
- EC2 instance to access EKS cluster

Installation Steps 
```sh
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm search repo prometheus-community
kubectl create namespace prometheus
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
kubectl get pods -n prometheus
kubectl get svc -n prometheus
```

Edit Prometheus Service (Edit type : LoadBalancer)
```sh
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```

Edit Grafana Service (Edit type : LoadBalancer) 
```sh
kubectl edit svc stable-grafana -n prometheus
```

Verify if service is changed to LoadBalancer and also to get the Load Balancer URL.
```sh
kubectl get svc -n prometheus
```

Access Grafana Dashboard
```sh
UserName: admin 
Password: prom-operator
```


For creating a dashboard to monitor the cluster:

```sh
Click '+' button on left panel and select ‘Import’.
Enter 12740 dashboard id under Grafana.com Dashboard.
Click ‘Load’.
Select ‘Prometheus’ as the endpoint under prometheus data sources drop down.
Click ‘Import’.
```


### Images For reference



<img width="1396" alt="image" src="https://user-images.githubusercontent.com/110477025/227587553-7163c709-85cf-4e23-a00b-823b08758859.png">



<img width="1400" alt="image" src="https://user-images.githubusercontent.com/110477025/227587788-06ce33dd-3a09-4f36-9bbd-aff0925615ed.png">


**This is on the EKS environment in AWS:**

# Reddit Clone App on Kubernetes with Ingress on EKS

This project demonstrates how to deploy a Reddit clone app on Kubernetes with Ingress and expose it to the world using Amazon EKS (Elastic Kubernetes Service) as the cluster. The original instructions were tailored for Minikube, but here we've updated them for EKS, which is more suitable for production-like environments. We'll cover creating an EKS cluster, building and pushing the Docker image to Amazon ECR (Elastic Container Registry), deploying the application using Kubernetes manifests, setting up an Ingress controller (using NGINX Ingress for compatibility with the original setup), testing, and adding cluster monitoring with Prometheus and Grafana. Below is an overview of the architecture of this Reddit Clone App running on Kubernetes with Ingress.

**Architecture Diagram** (Conceptual, based on original):  
- The app runs in a Deployment with replicas.  
- A Service exposes the Deployment internally.  
- An Ingress routes external traffic to the Service, using an AWS Load Balancer (provisioned by the NGINX Ingress Controller).  
- Monitoring: Prometheus scrapes metrics, Grafana visualizes them.

## Prerequisites

Before you begin, ensure you have the following tools and setup on your local machine:

- **AWS Account**: With permissions for EKS, ECR, IAM, and VPC resources.  
- **AWS CLI**: Installed and configured with your credentials (`aws configure`). Version 2.x recommended.  
- **kubectl**: Installed (version compatible with EKS, e.g., 1.28+). Install via `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`.  
- **eksctl**: For creating EKS clusters. Install via `curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_$(uname -m).tar.gz" | tar xz -C /tmp && sudo mv /tmp/eksctl /usr/local/bin`.  
- **Docker**: For building images. Install from official Docker docs.  
- **Helm**: For installing NGINX Ingress and Prometheus/Grafana. Install via `curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 && chmod 700 get_helm.sh && ./get_helm.sh`.  
- **Git**: For cloning the repo.  
- **Optional: AWS EC2 Instance**: If you prefer to run commands from an EC2 instance in the same region for lower latency (mentioned in monitoring section).  

Ensure your AWS region is set (e.g., `us-west-2`). You can follow AWS documentation for any installation: [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html), [kubectl](https://kubernetes.io/docs/tasks/tools/), [eksctl](https://eksctl.io/), [Docker](https://docs.docker.com/engine/install/), [Helm](https://helm.sh/docs/intro/install/).

## Installation

Follow these detailed steps to install and run the Reddit clone app on EKS:

### Step 1: Clone the Repository
Clone the updated repository to your local machine:  
```
git clone https://github.com/sandeep-vangala/reddit-clone.git
```  
Navigate to the project directory:  
```
cd reddit-clone
```  
(Note: If the repo is a fork, ensure it includes the necessary files like `Dockerfile`, `deployment.yaml`, `service.yaml`, and `ingress.yaml`. If missing, you may need to add them based on the original fork from LondheShubham153/reddit-clone-k8s-ingress.)

### Step 2: Create an EKS Cluster
If you don't have an EKS cluster, create one using eksctl. This creates a managed cluster with worker nodes. Replace `my-eks-cluster` with your cluster name and `us-west-2` with your preferred region.  
```
eksctl create cluster \
  --name my-eks-cluster \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```  
- This process takes 10-20 minutes.  
- It creates a VPC, security groups, and IAM roles automatically.  
- Verify: `eksctl get cluster --name my-eks-cluster`.  

Update your kubeconfig to connect to the cluster:  
```
aws eks update-kubeconfig --name my-eks-cluster --region us-west-2
```  
Verify connection:  
```
kubectl get nodes
```  
You should see 2 nodes listed.

### Step 3: Create an ECR Repository and Build/Push the Docker Image
The app requires a Docker image. We'll build it and push to ECR for EKS to pull securely.

1. Create an ECR repository:  
   ```
   aws ecr create-repository --repository-name reddit-clone-app --region us-west-2 --image-scanning-configuration scanOnPush=true
   ```  
   Note the repository URI, e.g., `123456789012.dkr.ecr.us-west-2.amazonaws.com/reddit-clone-app`.

2. Authenticate Docker to ECR:  
   ```
   aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-west-2.amazonaws.com
   ```

3. Build the Docker image (assuming a `Dockerfile` exists in the repo root; if not, create one for a typical Node.js/Flask app on port 5000):  
   ```
   docker build -t 123456789012.dkr.ecr.us-west-2.amazonaws.com/reddit-clone-app:latest .
   ```

4. Push the image:  
   ```
   docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/reddit-clone-app:latest
   ```

5. Update `deployment.yaml` to use this image: Open `deployment.yaml` and set `spec.template.spec.containers[0].image` to your ECR URI with `:latest`.

Ensure your EKS node IAM role has ECR read permissions (attach `AmazonEC2ContainerRegistryReadOnly` policy if needed).

### Step 4: Install NGINX Ingress Controller
EKS doesn't have Ingress enabled by default. Install NGINX Ingress using Helm for AWS Load Balancer integration.

1. Add Helm repo:  
   ```
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   helm repo update
   ```

2. Create a namespace:  
   ```
   kubectl create namespace ingress-nginx
   ```

3. Install the controller (this provisions an AWS Classic Load Balancer):  
   ```
   helm install ingress-nginx ingress-nginx/ingress-nginx \
     --namespace ingress-nginx \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-type"="classic"
   ```  
   (For NLB, use `"nlb"` instead of `"classic"`.)  

4. Verify:  
   ```
   kubectl get pods -n ingress-nginx
   kubectl get svc -n ingress-nginx
   ```  
   Wait for the EXTERNAL-IP (Load Balancer DNS) to appear.

### Step 5: Deploy the App to Kubernetes
Apply the Kubernetes manifests (update paths/names if needed).

1. Deploy the Deployment:  
   ```
   kubectl apply -f deployment.yaml
   ```  
   This creates the `reddit-deployment` with replicas running the app on port 5000.

2. Deploy the Service:  
   ```
   kubectl apply -f service.yaml
   ```  
   This creates a ClusterIP or NodePort service targeting port 5000.

3. (Optional) Expose if not using Ingress immediately:  
   ```
   kubectl expose deployment reddit-deployment --type=LoadBalancer --port=5000
   ```  
   But we'll use Ingress for external access.

4. Update `ingress.yaml` if needed: Ensure `spec.rules.host` is set to your domain (or `*` for testing), and `ingressClassName: nginx`. Then apply:  
   ```
   kubectl apply -f ingress.yaml
   ```

5. Verify deployments:  
   ```
   kubectl get deployments
   kubectl get services
   kubectl get ingress
   kubectl get pods
   ```  
   Describe any issues: `kubectl describe pod <pod-name>`.

## Test Ingress DNS for the App

1. Get the Ingress Load Balancer URL:  
   ```
   kubectl get svc -n ingress-nginx ingress-nginx-controller
   ```  
   Note the EXTERNAL-IP (e.g., `a123.elb.us-west-2.amazonaws.com`).

2. Test the app:  
   ```
   curl http://a123.elb.us-west-2.amazonaws.com/test
   ```  
   Replace `/test` with your app's path. If using a custom domain, update DNS to point to the ELB and set `host` in `ingress.yaml`.

For HTTPS, integrate AWS Certificate Manager (ACM) by adding annotations to the Ingress like `service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:...`.

## Cluster Monitoring using Prometheus & Grafana

Key Components:  
- Prometheus server: Processes and stores metrics data.  
- Alert Manager: Sends alerts to any systems/channels.  
- Grafana: Visualize scraped data in UI.

Prerequisites:  
- EKS Cluster is already set up (from Step 2).  
- Helm installed.  
- Optional: EC2 instance to access EKS cluster (connect via SSH and run commands there for better performance).

Installation Steps:  

1. Add Helm repositories:  
   ```
   helm repo add stable https://charts.helm.sh/stable
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   ```

2. Search for charts (optional verification):  
   ```
   helm search repo prometheus-community
   ```

3. Create a namespace for Prometheus:  
   ```
   kubectl create namespace prometheus
   ```

4. Install the kube-prometheus-stack:  
   ```
   helm install stable prometheus-community/kube-prometheus-stack --namespace prometheus
   ```

5. Check pods and services:  
   ```
   kubectl get pods -n prometheus
   kubectl get svc -n prometheus
   ```

6. Edit Prometheus service to use LoadBalancer (for external access):  
   ```
   kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
   ```  
   Change `spec.type` to `LoadBalancer`.

7. Edit Grafana service to use LoadBalancer:  
   ```
   kubectl edit svc stable-grafana -n prometheus
   ```  
   Change `spec.type` to `LoadBalancer`.

8. Verify services and get Load Balancer URLs:  
   ```
   kubectl get svc -n prometheus
   ```  
   Note the EXTERNAL-IP for Grafana.

Access Grafana Dashboard:  
- URL: http://<grafana-external-ip>:80 (or port from service).  
- Username: admin  
- Password: prom-operator  

For creating a dashboard to monitor the cluster:  
1. Click the '+' button on the left panel and select ‘Import’.  
2. Enter 12740 as the dashboard ID under Grafana.com Dashboard.  
3. Click ‘Load’.  
4. Select ‘Prometheus’ as the endpoint under Prometheus data sources drop down.  
5. Click ‘Import’.  

This dashboard will show cluster metrics like CPU, memory, pod status, etc.

### Images For Reference
(Refer to original repo images or generate similar: One for architecture diagram showing EKS nodes, Ingress LB, Pods; another for Grafana dashboard.)

## Contributing

If you'd like to contribute to this project, please open an issue or submit a pull request on https://github.com/sandeep-vangala/reddit-clone.

## Cleanup
To avoid costs:  
- Delete resources: `kubectl delete -f ingress.yaml -f service.yaml -f deployment.yaml`.  
- Uninstall Helm releases: `helm uninstall stable -n prometheus`.  
- Delete EKS cluster: `eksctl delete cluster --name my-eks-cluster --region us-west-2`.  
- Delete ECR repo: `aws ecr delete-repository --repository-name reddit-clone-app --region us-west-2 --force`.
- 


To enhance the `Ingress` resource for the Reddit clone app on an Amazon EKS cluster with NGINX Ingress Controller, you can add annotations to configure advanced behaviors such as SSL/TLS termination, AWS Load Balancer settings, rate limiting, or session affinity. Since you're deploying on EKS, we'll focus on annotations relevant to the AWS Load Balancer (used by NGINX Ingress) and common NGINX configurations. Below, I'll explain how to set up annotations for your provided `ingress.yaml` file, tailored for the Reddit clone app, ensuring compatibility with the NGINX Ingress Controller and AWS-specific features.

### Understanding Annotations
Annotations in the `Ingress` resource are key-value pairs in the `metadata.annotations` section that provide additional configuration to the Ingress controller. For NGINX Ingress on EKS, annotations can control:
- **AWS Load Balancer settings**: Type (Classic/NLB/ALB), SSL certificates, health checks.
- **NGINX-specific settings**: URL rewrites, timeouts, rate limiting.
- **Security**: TLS/SSL configuration.

### Updated `ingress.yaml` with Annotations
Here’s the modified `ingress.yaml` with annotations for EKS and NGINX Ingress, including explanations for each annotation. The service port is adjusted to 5000 (based on the Reddit clone project, which uses port 5000, not 3000 as in your provided file). Replace `domain.com` with your actual domain or leave it as `*` for testing with the Load Balancer’s hostname.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
  namespace: default  # Ensure this matches your deployment namespace
  annotations:
    # NGINX Ingress Controller class (required to specify NGINX)
    kubernetes.io/ingress.class: "nginx"
    # AWS Load Balancer annotations
    service.beta.kubernetes.io/aws-load-balancer-type: "external"
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"  # For NLB, use 'ip' for pod-level routing
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
    # SSL/TLS with AWS Certificate Manager (ACM) - replace with your certificate ARN
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:us-west-2:123456789012:certificate/xxxx-xxxx-xxxx-xxxx-xxxx"
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
    # NGINX-specific configurations
    nginx.ingress.kubernetes.io/rewrite-target: /  # Rewrite /test to root if needed
    nginx.ingress.kubernetes.io/ssl-redirect: "true"  # Force HTTPS
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"  # Connection timeout in seconds
    nginx.ingress.kubernetes.io/proxy-read-timeout: "30"  # Read timeout in seconds
    nginx.ingress.kubernetes.io/proxy-send-timeout: "30"  # Send timeout in seconds
    nginx.ingress.kubernetes.io/limit-rps: "10"  # Rate limit to 10 requests per second
    nginx.ingress.kubernetes.io/affinity: "cookie"  # Session affinity using cookies
    nginx.ingress.kubernetes.io/affinity-mode: "persistent"  # Persistent session stickiness
    nginx.ingress.kubernetes.io/session-cookie-name: "REDDITCLONESESSION"  # Custom cookie name
spec:
  ingressClassName: nginx  # Explicitly set NGINX Ingress class
  tls:  # Enable TLS for secure connections
  - hosts:
    - domain.com
    - "*.domain.com"
    secretName: reddit-clone-tls  # Kubernetes Secret for TLS (optional if using ACM)
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 5000  # Reddit clone app uses port 5000 per repo
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 5000
```

### Explanation of Annotations
1. **Ingress Controller Specification**:
   - `kubernetes.io/ingress.class: "nginx"`: Specifies that this Ingress uses the NGINX Ingress Controller. This is required for older Kubernetes versions or if multiple Ingress controllers are present. For Kubernetes 1.18+, `ingressClassName: nginx` in the spec is preferred (included above).
   
2. **AWS Load Balancer Annotations**:
   - `service.beta.kubernetes.io/aws-load-balancer-type: "external"`: Indicates that the NGINX Ingress Controller should use an AWS-managed Load Balancer (Classic, NLB, or ALB).
   - `service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"`: For Network Load Balancer (NLB), routes traffic directly to pod IPs, which is efficient for EKS. Use `"instance"` for Classic ELB if preferred.
   - `service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"`: Makes the Load Balancer accessible over the internet (use `"internal"` for private access).
   - `service.beta.kubernetes.io/aws-load-balancer-ssl-cert`: Specifies the ARN of an SSL certificate from AWS Certificate Manager (ACM) for HTTPS. Replace with your certificate’s ARN (get from AWS Console > ACM).
   - `service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"`: Enables SSL on port 443 for HTTPS traffic.

3. **NGINX-Specific Annotations**:
   - `nginx.ingress.kubernetes.io/rewrite-target: /`: Rewrites `/test` to the root (`/`) of the backend service, useful if the app expects requests at `/`. Adjust based on app behavior.
   - `nginx.ingress.kubernetes.io/ssl-redirect: "true"`: Redirects HTTP requests to HTTPS.
   - `nginx.ingress.kubernetes.io/proxy-connect-timeout`, `proxy-read-timeout`, `proxy-send-timeout`: Set timeouts to 30 seconds to handle slow connections (adjust as needed).
   - `nginx.ingress.kubernetes.io/limit-rps: "10"`: Limits requests to 10 per second per IP to prevent abuse (adjust for your traffic).
   - `nginx.ingress.kubernetes.io/affinity: "cookie"`, `affinity-mode: "persistent"`, `session-cookie-name: "REDDITCLONESESSION"`: Enables sticky sessions using cookies, ensuring users stay on the same pod for consistent experience (useful for stateful features).

4. **TLS Configuration**:
   - The `tls` section specifies hosts (`domain.com`, `*.domain.com`) and a Kubernetes Secret (`reddit-clone-tls`) for storing TLS certificates. If using ACM, the secret is optional since the Load Balancer handles SSL termination. If not using ACM, create a Secret with `kubectl create secret tls reddit-clone-tls --cert=path/to/cert.pem --key=path/to/key.pem`.

### Prerequisites for Annotations
- **NGINX Ingress Controller**: Must be installed in your EKS cluster. Follow the installation steps from the Reddit clone deployment guide:
  ```
  helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
  helm repo update
  kubectl create namespace ingress-nginx
  helm install ingress-nginx ingress-nginx/ingress-nginx \
    --namespace ingress-nginx \
    --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-type"="nlb"
  ```
- **AWS Certificate Manager (ACM)**: If using HTTPS, create or import a certificate in ACM for `domain.com` and `*.domain.com`. Copy the ARN for the `aws-load-balancer-ssl-cert` annotation.
- **DNS Setup**: If using a custom domain (`domain.com`), update your DNS provider to point to the NGINX Ingress Load Balancer’s EXTERNAL-IP (find via `kubectl get svc -n ingress-nginx ingress-nginx-controller`). For testing, use the Load Balancer’s hostname directly.

### Applying the Ingress
1. Save the updated `ingress.yaml` in your project directory (`reddit-clone/ingress.yaml`).
2. Apply it to your EKS cluster:  
   ```
   kubectl apply -f ingress.yaml
   ```
3. Verify the Ingress:  
   ```
   kubectl get ingress ingress-reddit-app
   kubectl describe ingress ingress-reddit-app
   ```
   Check the `Address` field for the Load Balancer hostname.

4. Get the Load Balancer URL:  
   ```
   kubectl get svc -n ingress-nginx ingress-nginx-controller
   ```
   Note the EXTERNAL-IP (e.g., `a123.elb.us-west-2.amazonaws.com`).

### Testing the Ingress
Test the app using the Load Balancer URL or your domain:  
```
curl https://domain.com/test
curl https://a123.elb.us-west-2.amazonaws.com/test
```  
- Ensure the Reddit clone service (`reddit-clone-service`) is running on port 5000 (verify with `kubectl get svc reddit-clone-service`).  
- If the app doesn’t respond, check pod logs (`kubectl logs <pod-name>`) or Ingress events (`kubectl describe ingress ingress-reddit-app`).

### Troubleshooting
- **Ingress not routing**: Verify `ingressClassName: nginx` and NGINX controller is running (`kubectl get pods -n ingress-nginx`).  
- **SSL errors**: Ensure the ACM certificate ARN is correct and covers `domain.com` and `*.domain.com`.  
- **Service not found**: Confirm `reddit-clone-service` exists and matches the port (5000).  
- **Rate limiting issues**: Adjust `limit-rps` if too restrictive.  
- **Load Balancer not provisioned**: Check AWS Console > EC2 > Load Balancers and ensure the NGINX controller pod is healthy.

### Additional Notes
- **Port Correction**: The original Reddit clone project uses port 5000 (per `service.yaml` and app code), not 3000 as in your `ingress.yaml`. Ensure consistency across `service.yaml` and `ingress.yaml`.  
- **Production Considerations**: Use an Application Load Balancer (ALB) with AWS Load Balancer Controller for more advanced routing (replace NGINX if needed). Add `cert-manager` for automatic TLS certificate management.  
- **Cost Management**: Monitor EKS nodes and Load Balancer costs. Delete resources with `kubectl delete -f ingress.yaml` and `eksctl delete cluster --name my-eks-cluster` when done.

This setup integrates the Reddit clone app with EKS-specific features while maintaining the original project’s structure. If you need further customization (e.g., ALB, custom rewrite rules, or specific NGINX settings), provide details, and I can refine the configuration!



