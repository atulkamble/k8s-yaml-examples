# Kubernetes YAML Templates: Basic to Advanced

A complete reference of Kubernetes YAML configuration examples with real-world use cases and operations.

---

## Create EKS Cluster 
```
eksctl create cluster \
  --name demo-cluster \
  --region us-east-1 \
  --nodegroup-name demo-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 2 \
  --managed
```

## 🟢 BASIC LEVEL

### 1. Pod

**Scenario**: Deploy a standalone container (nginx) to test Kubernetes setup.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80
```

**Operations:**

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod my-pod
```

---

### 2. Service (ClusterIP)

**Scenario**: Expose a pod or deployment internally within the cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

**Operations:**

```bash
kubectl apply -f service.yaml
kubectl get svc
kubectl describe svc my-service
```

---

### 3. Deployment

**Scenario**: Run multiple replicas of a web app with managed rollouts.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

**Operations:**

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl rollout status deployment nginx-deployment
```

---

## 🟡 INTERMEDIATE LEVEL

### 4. ConfigMap

**Scenario**: Inject configuration data into pods.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
```

**Operations:**

```bash
kubectl apply -f configmap.yaml
kubectl describe configmap app-config
```

---

### 5. Secret

**Scenario**: Store sensitive information (e.g., DB credentials).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 for 'admin'
  password: cGFzc3dvcmQ=  # base64 for 'password'
```

**Operations:**

```bash
kubectl apply -f secret.yaml
kubectl describe secret db-secret
```

---

### 6. Ingress

**Scenario**: Route external traffic to internal services.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

**Operations:**

```bash
kubectl apply -f ingress.yaml
kubectl get ingress
```

---

### 7. Persistent Volume Claim

**Scenario**: Provision dynamic storage for a pod.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

**Operations:**

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
```

---

## 🔴 ADVANCED LEVEL

### 8. CronJob

**Scenario**: Run a periodic task (e.g., cleanup job every minute).

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox
              args:
                - /bin/sh
                - -c
                - date; echo Hello from CronJob
          restartPolicy: OnFailure
```

**Operations:**

```bash
kubectl apply -f cronjob.yaml
kubectl get cronjob
kubectl get jobs --watch
```

---

### 9. Horizontal Pod Autoscaler

**Scenario**: Scale deployment replicas based on CPU utilization.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

**Operations:**

```bash
kubectl apply -f hpa.yaml
kubectl get hpa
```

---

### 10. Pod with Init Containers & Volume Mount

**Scenario**: Prepare data using init container before main app starts.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  volumes:
    - name: shared-data
      emptyDir: {}
  initContainers:
    - name: init-myservice
      image: busybox
      command: ['sh', '-c', 'echo preparing app > /data/init.txt']
      volumeMounts:
        - name: shared-data
          mountPath: /data
  containers:
    - name: main-container
      image: busybox
      command: ['sh', '-c', 'cat /data/init.txt && sleep 3600']
      volumeMounts:
        - name: shared-data
          mountPath: /data
```

**Operations:**

```bash
kubectl apply -f init-container.yaml
kubectl logs init-demo -c init-myservice
```

---

### 11. NetworkPolicy

**Scenario**: Allow traffic to a backend pod only from frontend pods.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-only-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
```

**Operations:**

```bash
kubectl apply -f network-policy.yaml
kubectl get networkpolicy
```

---

### 12. Advanced Deployment with Probes & Resource Limits

**Scenario**: Deploy a resilient web app with health checks and resource quotas.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web-container
          image: myorg/webapp:latest
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 3
            periodSeconds: 5
```

**Operations:**

```bash
kubectl apply -f deployment-advanced.yaml
kubectl get pods
kubectl describe pod <pod-name>
```

---

### 📂 REPO IDEA

Structure:

```
k8s-yaml-examples/
├── basic/
├── intermediate/
├── advanced/
```
