## Deployment

### Create Deployment
```bash
kubectl create deployment nginx-deployment --image butch/sdc-nginx:1.0.0
# Output: deployment.apps/nginx-deployment created
```

### Verify Created Deployment
 - Get Deployment: nginx-deployment
    ```bash
    kubectl get deployment nginx-deployment
    ```
    ```bash
    # Output:
        NAME               READY   UP-TO-DATE   AVAILABLE   AGE
        nginx-deployment   1/1     1            1           2m3s
    ```
 - Describe Deployment
    ```bash
    kubectl describe deployment nginx-deployment
    ```
    ```bash
    # Output:
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Fri, 06 Jan 2023 14:11:57 +0800
    Labels:                 app=nginx-deployment
    Annotations:            deployment.kubernetes.io/revision: 1
    Selector:               app=nginx-deployment
    Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
    Labels:  app=nginx-deployment
    Containers:
    sdc-nginx:
        Image:        butch/sdc-nginx:1.0.0
        Port:         <none>
        Host Port:    <none>
        Environment:  <none>
        Mounts:       <none>
    Volumes:        <none>
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-7d6c9564c (1/1 replicas created)
    Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  4m1s  deployment-controller  Scaled up replica set nginx-deployment-7d6c9564c to 1
    ```
 - Verify created ReplicaSet
   ```bash
   kubectl get replicaset
   ```
   ```bash
   # Output
   NAME                         DESIRED   CURRENT   READY   AGE
   nginx-deployment-7d6c9564c   1         1         1       7m36s
   ``` 

### Scaling Up/Down Deployment
```bash
kubectl scale --replicas=5 deployment/nginx-deployment
# Output: deployment.apps/nginx-deployment scaled
```
```bash
# Verify the deployments
kubectl get deployment
```
```bash
# Output
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           13m
```

### Expose Deployment as Service
```bash
kubectl expose deployment nginx-deployment --type=NodePort --port=80 --target-port=80 --name=nginx-svc
# Output: service/nginx-svc exposed
```
```bash
# Get Service Info
kubectl get service nginx-svc
```
```bash
# Output
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-svc   NodePort   10.110.230.53   <none>        80:30075/TCP   84s
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
