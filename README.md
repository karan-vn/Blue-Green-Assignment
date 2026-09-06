# Blue-Green Deployment using Docker, Kubernetes and Minikube

This project demonstrates a Blue-Green Deployment strategy for a full-stack registration application using Docker, Kubernetes, and Minikube.

The application consists of:

- Node.js and Express backend
- MongoDB Atlas database
- Blue frontend version
- Green frontend version
- Docker containers
- Kubernetes Deployments and Services
- Kubernetes Ingress
- Minikube
- Blue-Green traffic switching

## Architecture

```text
                         User / Browser
                               |
                               v
                    +---------------------+
                    | Kubernetes Ingress  |
                    +----------+----------+
                               |
                 +-------------+-------------+
                 |                           |
              /api                          /
                 |                           |
                 v                           v
        +----------------+          +----------------+
        | Backend Service |          | Frontend       |
        +-------+--------+          | Service        |
                |                   +-------+--------+
                v                           |
        +----------------+           Selector:
        | Backend Pods   |           version=blue
        | Node.js        |                  |
        +-------+--------+                  v
                |                    +-------------+
                v                    | Blue Pods   |
        +----------------+           +-------------+
        | MongoDB Atlas  |
        +----------------+           During deployment,
                                     selector changes to
                                     version=green
                                            |
                                            v
                                     +-------------+
                                     | Green Pods  |
                                     +-------------+

```
# Project Structure
```
Blue-green-Deployment/
│
├── backend/
│   ├── models/
│   │   └── user.js
│   ├── routes/
│   │   └── users.js
│   ├── server.js
│   ├── package.json
│   ├── .env
│   └── Dockerfile
│
├── frontend-blue/
│   ├── public/
│   │   └── index.html
│   ├── server.js
│   ├── package.json
│   └── Dockerfile
│
├── frontend-green/
│   ├── public/
│   │   └── app.js
│   ├── server.js
│   ├── package.json
│   └── Dockerfile
│
├── k8s/
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-blue-deployment.yaml
│   ├── frontend-green-deployment.yaml
│   ├── frontend-service.yaml
│   └── ingress.yaml
│
└── README.md
```
# Technologies Used
* Node.js
* Express.js
* MongoDB Atlas
* Docker
* Kubernetes
* Minikube
* kubectl
* Helm
* Git
* PowerShell

# Prerequisites

Install the following tools:
* Docker Desktop
* Node.js
* Git
* Minikube
* kubectl
* Helm
# Verify the installations:
```
docker --version
minikube version
kubectl version --client
helm version
node --version
npm --version
git --version
```
Make sure Docker Desktop is running before building images or starting Minikube.

1. Clone the Repository
```
git clone <repository-url>
cd Blue-green-Deployment
```
2. Backend Configuration

Navigate to the backend directory:
```
cd backend
```
Install dependencies:
```
npm install
```
Create a .env file:
```
PORT=5000
MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/registration
```
Do not commit the .env file or expose database credentials.

Start the backend:
```
npm start
```
The backend runs on:
```
http://localhost:5000
```
Backend Health Check

Open:
```
http://localhost:5000/health
```
Expected response:
```
{
  "status": "ok",
  "message": "Backend API is running"
}
```
3. Frontend Blue

Navigate to the Blue frontend:
```
cd ..\frontend-blue
```
Install dependencies:
```
npm install
```
Start the application:
```
npm start
```
Blue frontend runs on:
```
http://localhost:3100
```
The Blue version represents the currently stable application version.

4. Frontend Green

Navigate to the Green frontend:
```
cd ..\frontend-green
```
Install dependencies:
```
npm install
```
Start the application:
```
npm start
```
Green frontend runs on:
```
http://localhost:3200
```
The Green version represents the new version that can be tested independently before receiving production traffic.

5. API Configuration

For Kubernetes deployment, frontend API requests use the relative path:
```
fetch('/api/users')
```
instead of:
```
fetch('http://localhost:5000/api/users')
```
This is required because localhost in a browser refers to the user's machine rather than the Kubernetes backend pod.

Kubernetes Ingress routes:
```
/api  -> Backend Service
/     -> Frontend Service
```
This allows both Blue and Green frontends to use the same backend API.

6. Docker Images

The project contains three Dockerfiles:
```
backend/Dockerfile
frontend-blue/Dockerfile
frontend-green/Dockerfile
```
Build the backend image:
```
docker build -t bluegreen-backend:1.0 ./backend
```
Build the Blue frontend image:
```
docker build -t bluegreen-frontend-blue:1.1 ./frontend-blue
```
Build the Green frontend image:
```
docker build -t bluegreen-frontend-green:1.1 ./frontend-green
```
Verify the images:
```
docker images
```
Expected images:
```
bluegreen-backend:1.0
bluegreen-frontend-blue:1.1
bluegreen-frontend-green:1.1
```
7. Start Minikube

Check Minikube:
```
minikube status
```
If it is not running:
```
minikube start
```
Verify:
```
minikube status
```
Expected status:
```
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
8. Enable Minikube Addons

Enable Ingress:
```
minikube addons enable ingress
```
Enable Metrics Server:
```
minikube addons enable metrics-server
```
Verify:
```
minikube addons list
```
The following addons should be enabled:
```
ingress
metrics-server
```
9. Load Docker Images into Minikube

Load the images into Minikube:
```
minikube image load bluegreen-backend:1.0
minikube image load bluegreen-frontend-blue:1.1
minikube image load bluegreen-frontend-green:1.1
```
Verify:
```
minikube image ls
```
The application images should appear in the output.

10. Kubernetes Deployment

Apply the backend Deployment:
```
kubectl apply -f k8s/backend-deployment.yaml
```
Apply the backend Service:
```
kubectl apply -f k8s/backend-service.yaml
```
Apply the Blue frontend Deployment:
```
kubectl apply -f k8s/frontend-blue-deployment.yaml
```
Apply the Green frontend Deployment:
```
kubectl apply -f k8s/frontend-green-deployment.yaml
```
Apply the frontend Service:
```
kubectl apply -f k8s/frontend-service.yaml
```
Apply the Ingress:
```
kubectl apply -f k8s/ingress.yaml
```
Alternatively, all manifests can be applied together:
```
kubectl apply -f k8s/
```
11. Verify Kubernetes Resources

Check all pods:
```
kubectl get pods
```
Check Deployments:
```
kubectl get deployments
```
Check Services:
```
kubectl get services
```
Check Ingress:
```
kubectl get ingress
```
Check pod labels:
```
kubectl get pods --show-labels
```
12. Blue-Green Deployment

The frontend Service uses a selector to determine which frontend version receives traffic.

Initial Blue configuration:
```
selector:
  app: frontend
  version: blue
```
This sends traffic to the Blue frontend pods.

Verify Blue

Check the Service:
```
kubectl get service frontend-service -o yaml
```
Verify that the selector contains:
```
version: blue
```
Access the application through the Minikube Ingress.

The Blue frontend should be serving production traffic.

13. Deploy Green

The Green Deployment runs independently from Blue.

Verify Green pods:
```
kubectl get pods -l app=frontend,version=green
```
Check the Green frontend health endpoint:

/health

Green should be tested before switching production traffic.

14. Switch Traffic from Blue to Green

Patch the frontend Service selector:
```
kubectl patch service frontend-service -p '{"spec":{"selector":{"app":"frontend","version":"green"}}}'
```
Verify:
```
kubectl get service frontend-service -o yaml
```
The selector should now contain:
```
version: green
```
New frontend traffic is now routed to the Green deployment.

15. Verify Green Traffic

Check the frontend endpoints:
```
kubectl get endpoints frontend-service
```
Check the pods receiving traffic:
```
kubectl get pods -l app=frontend,version=green
```
The application should now display the Green version.

16. Rollback to Blue

If a problem is discovered in Green, traffic can be switched back to Blue without redeploying the application.

Run:
```
kubectl patch service frontend-service -p '{"spec":{"selector":{"app":"frontend","version":"blue"}}}'
```
Verify:
```
kubectl get service frontend-service -o yaml
```
The selector should show:

version: blue

Traffic is now routed back to Blue.

17. Health Checks

The application provides health endpoints.

Backend:
```
/health
````
Blue frontend:
```
/health
```
Green frontend:
```
/health
```
Kubernetes readiness and liveness probes should use these endpoints to ensure that traffic is sent only to healthy pods.

18. Monitoring

Metrics Server can be used to inspect resource usage.

Check node resources:
```
kubectl top nodes
```
Check pod resources:
```
kubectl top pods
```
If metrics are not immediately available after enabling the addon, wait briefly and run the commands again.

19. Troubleshooting
Check pod logs
```
kubectl logs <pod-name>
```
Describe a pod
```
kubectl describe pod <pod-name>
```
Check Deployment status
```
kubectl rollout status deployment/<deployment-name>
```
Check Service
```
kubectl describe service frontend-service
```
Check Ingress
```
kubectl describe ingress
```
Check all resources
```
kubectl get all
```
20. Useful Docker Commands

List images:
```
docker images
```
List containers:
```
docker ps -a
```
Run a container:
```
docker run -p 5000:5000 bluegreen-backend:1.0
```
Stop a container:
```
docker stop <container-id>
```
Remove a container:
```
docker rm <container-id>
```
21. Useful Kubernetes Commands

List pods:
```
kubectl get pods
```
List Deployments:
```
kubectl get deployments
```
List Services:
```
kubectl get services
```
List Ingress:
```
kubectl get ingress
```
View pod labels:
```
kubectl get pods --show-labels
```
View logs:
```
kubectl logs <pod-name>
```
Delete all resources created from the manifests:
```
kubectl delete -f k8s/
```
22. Cleanup

Remove Kubernetes resources:
```
kubectl delete -f k8s/
```
Stop Minikube:
```
minikube stop
```
To completely remove the Minikube cluster:
```
minikube delete
```
23. Blue-Green Deployment Benefits

Blue-Green Deployment provides:

* Zero or minimal downtime during releases
* Independent validation of the new version
* Fast rollback
* Reduced deployment risk
* Ability to test the new version before production traffic is switched
* Simple traffic management using Kubernetes Services

The key mechanism in this project is the Kubernetes Service selector:
```
version=blue
```
or:
```
version=green
```
Changing this selector switches production traffic between the two frontend versions.

24. Best Practices
* Never commit database credentials.
* Store secrets using Kubernetes Secrets in production environments.
* Use readiness and liveness probes.
* Configure CPU and memory requests/limits.
* Validate Green before switching production traffic.
* Monitor application logs and resource usage.
* Keep Blue available until Green has been successfully validated.
* Maintain a tested rollback procedure.
* Use versioned Docker image tags.
* Avoid using latest for production deployments.
25. Assignment Outcome

This project demonstrates a complete Blue-Green Deployment workflow:
```
Build Application
       |
       v
Build Docker Images
       |
       v
Load Images into Minikube
       |
       v
Deploy Backend
       |
       v
Deploy Blue
       |
       v
Deploy Green
       |
       v
Test Green
       |
       v
Switch Service
       |
       v
Green Receives Traffic
       |
       v
Rollback to Blue if Required
```

# Screen Shots

<img width="956" height="532" alt="Blue-Green" src="https://github.com/user-attachments/assets/86bbfa5e-be1f-4328-9b6c-5ec73949fc13" />

<img width="959" height="533" alt="Blue-Green 2" src="https://github.com/user-attachments/assets/dba9fc13-6c73-4c2f-b96e-72f321044927" />

<img width="473" height="156" alt="Screenshot 2026-09-06 163323" src="https://github.com/user-attachments/assets/c90dd16a-9b01-4e46-b503-d524164f977f" />

<img width="770" height="217" alt="Screenshot 2026-09-06 163335" src="https://github.com/user-attachments/assets/bedb4377-d34d-4e2e-ab2e-1ae6362c6f40" />

<img width="474" height="304" alt="Screenshot 2026-09-06 164251" src="https://github.com/user-attachments/assets/a85b03e6-204e-4b0a-9ed4-ca677e18b523" />
