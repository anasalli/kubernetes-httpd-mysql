To create a frontend-backend application using **Apache HTTP Server (httpd)** and **MySQL** in a Kubernetes cluster under the `grras-project` namespace, and to expose the frontend so it can be accessed from outside the cluster, follow these steps:

### 1. **Create the Namespace:**

First, create the `grras-project` namespace:

```bash
kubectl create namespace grras-project
```

### 2. **Deploy the MySQL Backend:**

Create a MySQL deployment and service within the `grras-project` namespace.

**Create a MySQL Deployment YAML file:**

```yaml
# mysql-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-backend
  namespace: grras-project
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql-backend
  template:
    metadata:
      labels:
        app: mysql-backend
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rootpassword"
        - name: MYSQL_DATABASE
          value: "mydatabase"
        - name: MYSQL_USER
          value: "myuser"
        - name: MYSQL_PASSWORD
          value: "mypassword"
        ports:
        - containerPort: 3306
```

**Create a MySQL Service YAML file:**

```yaml
# mysql-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-backend
  namespace: grras-project
spec:
  ports:
  - port: 3306
  selector:
    app: mysql-backend
  clusterIP: None  # Headless service for direct pod-to-pod communication
```

Apply the MySQL deployment and service:

```bash
kubectl apply -f mysql-deployment.yaml
kubectl apply -f mysql-service.yaml
```

### 3. **Deploy the Apache HTTP Server (httpd) Frontend:**

Now, create the Apache HTTP Server (httpd) deployment and service.

**Create an HTTPD Deployment YAML file:**

```yaml
# httpd-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-frontend
  namespace: grras-project
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd-frontend
  template:
    metadata:
      labels:
        app: httpd-frontend
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/local/apache2/htdocs/
      volumes:
      - name: html
        configMap:
          name: httpd-html-config
```

**Create a ConfigMap for HTML Content:**

```yaml
# httpd-html-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: httpd-html-config
  namespace: grras-project
data:
  index.html: |
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Frontend-Backend Example</title>
    </head>
    <body>
        <h1>Welcome to the Frontend!</h1>
        <p>This is a simple frontend-backend application.</p>
        <p>Backend is running on MySQL.</p>
    </body>
    </html>
```

**Create an HTTPD NodePort Service YAML file:**

```yaml
# httpd-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-frontend
  namespace: grras-project
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # External port to access from outside the cluster
  selector:
    app: httpd-frontend
```

Apply the ConfigMap, deployment, and service:

```bash
kubectl apply -f httpd-html-config.yaml
kubectl apply -f httpd-deployment.yaml
kubectl apply -f httpd-service.yaml
```

### 4. **Access the Frontend Application:**

With the NodePort service, the frontend can be accessed from outside the cluster using the IP address of any node in the cluster and the specified NodePort (`30080` in this case).

**Example**:
- If your node’s IP is `192.168.1.100`, access the frontend by going to `http://192.168.1.100:30080` in your browser.

### 5. **Verify Connectivity between Frontend and Backend:**

To ensure that the frontend can communicate with the backend, you can update the frontend HTML to fetch data from the MySQL backend or log connection status (this part would require additional scripting or application logic).

### Conclusion:

You now have a frontend-backend application deployed in the `grras-project` namespace in Kubernetes. The frontend (Apache HTTP Server) is exposed to the outside world via a NodePort service, while the backend (MySQL) is accessible within the cluster.
