## Deployment

### Edit Deployment
```bash
kubectl edit deployment/nginx-deployment
``` 
```yaml
# It will show the deployment metadata in yaml format.
# This is not the full metadata, and just look for container image and set it from butch/sdc-nginx:1.0.0 to nginx:alpine
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2023-01-06T06:41:20Z"
  generation: 2
  labels:
    app: nginx-deployment
  name: nginx-deployment
  namespace: default
  resourceVersion: "229345"
  uid: 7b249702-c162-47ef-b55d-a9c2d8265f7d
spec:
  progressDeadlineSeconds: 600
  replicas: 5
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:alpine
        imagePullPolicy: IfNotPresent
        name: sdc-nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

### Verify Rollout Status
Observation: Rollout happens in a rolling update model, so no downtime.

```bash
kubectl rollout status deployment/nginx-deployment
# Output: deployment "nginx-deployment" successfully rolled out
```

### Verify Rollout History
```bash
kubectl rollout history deployment/nginx-deployment
```
```bash
# Output
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

### Access Application
```bash
kubectl get nodes -o wide
```
```bash
# Output
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube   Ready    control-plane   23d   v1.25.3   192.168.49.2   <none>        Ubuntu 20.04.5 LTS   5.15.0-57-generic   docker://20.10.20
```

```bash
# Get nginx-svc assigned NodePort
kubectl get service nginx-svc
```
```bash
# Output
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-svc   NodePort   10.110.230.53   <none>        80:30075/TCP   5m9s
```
```bash
 curl http://192.168.49.2:30075/ -vvv
```
```bash
# Output
    *   Trying 192.168.49.2:30075...
    * Connected to 192.168.49.2 (192.168.49.2) port 30075 (#0)
    > GET / HTTP/1.1
    > Host: 192.168.49.2:30075
    > User-Agent: curl/7.81.0
    > Accept: */*
    > 
    * Mark bundle as not supporting multiuse
    < HTTP/1.1 200 OK
    < Server: nginx/1.18.0
    < Date: Fri, 06 Jan 2023 06:34:19 GMT
    < Content-Type: text/html
    < Content-Length: 612
    < Last-Modified: Sun, 07 Jun 2020 07:48:56 GMT
    < Connection: keep-alive
    < ETag: "5edc9be8-264"
    < Accept-Ranges: bytes
    < 
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    * Connection #0 to host 192.168.49.2 left intact
``` 

### Clean Up
```bash
kubectl delete service nginx-svc
# Output: service "nginx-svc" deleted
```
```bash
kubectl delete deployment nginx-deployment
# Output: deployment.apps "nginx-deployment" deleted
```