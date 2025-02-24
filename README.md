# Lab 27: Updating Applications and Rolling Back Changes (Declarative)

## Objective
This lab demonstrates how to deploy an NGINX application with three replicas, expose it, update the image to Apache, and roll back changes—all using declarative YAML manifests.

## Prerequisites
- A Kubernetes cluster (local or remote).
- `kubectl` CLI installed and configured.
- Basic knowledge of YAML manifest files.

---

## 1. Create the Deployment (NGINX) and Service

### 1.1. **Deployment YAML**: `nginx-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### 1.2. **Service YAML**: `nginx-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

#### Apply the YAML manifests:
```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

#### Verify the Deployment and Service:
```bash
kubectl get deployments
kubectl get services
```

---

## 2. Access NGINX Service (Optional: Port Forwarding)
If you’re running on a local cluster (like Minikube) or in a managed environment with NodePort, you can either:

- **Minikube**: Use `minikube service nginx-service` to open it in your browser.
- **Port Forward**:
  ```bash
  kubectl port-forward service/nginx-service 8080:80
  ```
  Then visit [http://localhost:8080](http://localhost:8080).

---

## 3. Update the Deployment from NGINX to Apache (HTTPD)

### 3.1. Modify the Deployment YAML
Open `nginx-deployment.yaml` and change the `image` field from `nginx:latest` to `httpd:latest`.
For example:
```yaml
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        # Previously: image: nginx:latest
        image: httpd:latest
        ports:
        - containerPort: 80
```

### 3.2. Re-apply the Updated YAML
```bash
kubectl apply -f nginx-deployment.yaml
```

### 3.3. Check the Rollout Status
```bash
kubectl rollout status deployment/nginx-deployment
```

---

## 4. View the Deployment’s Rollout History
Kubernetes keeps track of revisions when using declarative manifests (with a label selector). You can view the history with:
```bash
kubectl rollout history deployment/nginx-deployment
```

---

## 5. Roll Back to the Previous Image Version

### 5.1. Undo the Deployment
```bash
kubectl rollout undo deployment/nginx-deployment
```
This command will revert to the last successful version (which was the `nginx:latest` image).

### 5.2. Monitor Pod Status
```bash
kubectl get pods -w
```
You will see the old pods (Apache) terminating and the new pods (NGINX) spinning up.

---

## 6. Verification
1. **Confirm the Rollback**:
   ```bash
   kubectl get deployment nginx-deployment -o wide
   ```
   You should see the `IMAGE` field back to `nginx:latest`.

2. **Test the Service**:
   - If port forwarding is still active, visit [http://localhost:8080](http://localhost:8080).
   - Otherwise, use `minikube service nginx-service` or your external NodePort URL.

---

## 7. Cleanup (Optional)
If you want to remove the resources after testing:
```bash
kubectl delete -f nginx-deployment.yaml
kubectl delete -f nginx-service.yaml
```

---

**Congratulations!** You have successfully demonstrated how to **deploy**, **update**, and **roll back** an application using declarative Kubernetes YAML manifests.


