# 🗺️ Tour Website Deployment on Kubernetes with Monitoring

This project showcases the deployment of a **static tour website** using **HTML, CSS, and JavaScript**. It is containerized with **Docker**, orchestrated using **Kubernetes**, and monitored using **Prometheus and Grafana**. The deployment runs on **AWS EC2 instances**.

---

## 📁 Project Structure

```
.
├── index.html
├── styles.css
├── script.js
├── Dockerfile
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── prometheus-deployment.yaml
│   ├── grafana-deployment.yaml
│   └── prometheus-config.yaml
└── README.md
```

---

## 🚀 Technologies Used

- **Frontend**: HTML, CSS, JavaScript
- **Web Server**: Nginx
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **Monitoring & Observability**: Prometheus & Grafana

---

## 📦 Dockerfile

```Dockerfile
FROM nginx:alpine
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY . .
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## ☸️ Kubernetes Manifests

### 🚀 Website Deployment (`deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tour-website
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tour-website
  template:
    metadata:
      labels:
        app: tour-website
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "80"
    spec:
      containers:
      - name: tour-website
        image: your-dockerhub-username/tour-website:latest
        ports:
        - containerPort: 80
```

### 🌐 Service (`service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tour-website-service
spec:
  type: NodePort
  selector:
    app: tour-website
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### 📊 Prometheus (`prometheus-deployment.yaml`)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'tour-website'
        static_configs:
          - targets: ['tour-website-service:80']

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus
          args:
            - "--config.file=/etc/prometheus/prometheus.yml"
          ports:
            - containerPort: 9090
          volumeMounts:
            - name: config-volume
              mountPath: /etc/prometheus/
      volumes:
        - name: config-volume
          configMap:
            name: prometheus-config
```

### 📈 Grafana (`grafana-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana
          image: grafana/grafana
          ports:
            - containerPort: 3000
```

### 🛰️ Monitoring Services (Prometheus + Grafana)

```yaml
# Prometheus service
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
spec:
  type: NodePort
  selector:
    app: prometheus
  ports:
    - port: 9090
      targetPort: 9090
      nodePort: 30090

---
# Grafana service
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30030
```

---

## 🌐 Accessing Services

| Service       | URL                            | NodePort |
|---------------|---------------------------------|----------|
| Tour Website  | `http://<NodeIP>:30080`        | 30080    |
| Prometheus    | `http://<NodeIP>:30090`        | 30090    |
| Grafana       | `http://<NodeIP>:30030`        | 30030    |

> 🔐 **Grafana Default Login**: `admin` / `admin`

---

## 🔍 Grafana Setup

1. Open Grafana in your browser
2. Go to **Settings > Data Sources**
3. Add **Prometheus** with URL: `http://prometheus-service:9090`
4. Create dashboards to monitor pod performance, uptime, and traffic

---

## ✅ Commands for Verification

```bash
kubectl get pods
kubectl get svc
kubectl describe deployment tour-website
```

---

## 🧹 Cleanup

```bash
kubectl delete -f k8s/
```

---

## 📦 Push Docker Image (Optional)

```bash
docker build -t your-dockerhub-username/tour-website:latest .
docker push your-dockerhub-username/tour-website:latest
```

---

## 📝 Notes

- Ensure EC2 security group allows ports: 30080, 30090, 30030
- For HTTPS or custom domain, consider adding an Ingress with TLS termination

---

## 💬 Questions?

Feel free to raise issues or contact for help configuring advanced dashboards, auto-scaling, or ingress controllers.

