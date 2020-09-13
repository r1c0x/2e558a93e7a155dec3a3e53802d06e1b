### Overview

This is a demo wordpress application with AWS EKS (Kubernetes) + Helm.
It creates:
- Wordpress application
- MySQL database
- Separate Persistence storage for Wordpress and MySQL
- AWS LoadBalancer for Wordpress access

### Why Helm?
https://helm.sh/docs/topics/charts/
Helm uses a packaging format called charts. A chart is a collection of files that describe a related set of Kubernetes resources. A single chart might be used to deploy something simple, like a memcached pod, or something complex, like a full web app stack with HTTP servers, databases, caches, and so on.

### Helm file structure
```
.
├── Chart.yaml
├── templates
│   ├── mysql-deployment.yaml
│   ├── mysql-pvc-volumes.yaml
│   ├── mysql-secret.yaml
│   ├── mysql-service.yaml
│   ├── wp-configmap.yaml
│   ├── wp-deployment.yaml
│   ├── wp-pvc-volumes.yaml
│   ├── wp-secret.yaml
│   ├── wp-service.yaml
│   └── wp-serviceaccount.yaml
└── values-test.yaml

1 directory, 12 files
```

### Prerequisites
AWS EKS 1.14 or later
https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html

Helm 3
https://helm.sh/docs/intro/install/

kubectl 
https://kubernetes.io/docs/tasks/tools/install-kubectl/

### Installation
1. Install wordpress application using Helm, assume that it's going to the `default` namespace
```
cd helm
helm install wordpress . -f values-test.yaml -n default
```
2. Get the AWS LoadBalancer url from k8s service
```
kubectl get svc/wordpress-svc-5-5-1-php7-4-apache -n default
```
You will get something like:
```
NAME                                TYPE           CLUSTER-IP    EXTERNAL-IP                                                                   PORT(S)        AGE
wordpress-svc-5-5-1-php7-4-apache   LoadBalancer   10.100.8.40   a663ded6d4d694avcsbd11600fbfe111-12345678.ap-southeast-1.elb.amazonaws.com   80:31578/TCP   105s
```
3. Access the URL through the LoadBalancer url with your browser, the default protocol is `http` and port `80`

### Clean up
1. Use Helm unistall command
```
helm uninstall wordpress -n default
```
2. Everything related to the templates will be cleaned up

## What? You don't like Helm?
1. You can also locally render templates to grab the k8s templates
```
helm template wordpress . -f values-test.yaml
```
2. You will get something like this, save it as a usual k8s yaml template:
```
---
# Source: wordpress/templates/wp-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-config-5-5-1-php7-4-apache
  labels:
    app: wordpress
    version: "5.5.1-php7.4-apache"
    environment: test
data:
  WORDPRESS_DB_HOST: wordpress-mysql-svc
---
# Source: wordpress/templates/mysql-pvc-volumes.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-mysql-pvc
  labels:
    app: wordpress-mysql
    version: "5.7"
    environment: test
spec:
  accessModes:
    - ReadWriteOnce
...
```

### Roadmap
I did finish this assignment kinda in rush due to my exam last week, there are pending tasks which can make the service better.
- Make Wordpress scalable, with properly k8s HPA + Metics-server + AWS EFS
- Register a proper domain name for it, adds SSL certificate to the loadbalancer by service annotations
- Use Helm _helpers.tpl for more clean code and easy to maintain.
- Implement secret storage service like AWS serect manager or Hashicorp Vault for secret storage and pull 

### Questions?
Welcomed to Whatsapp me
