## PODS

### Create a Pod (Imperative Approach)
```bash
kubectl run pod-nginx --image butch/sdc-nginx:1.0.0
```

### Get List of Pods
```bash
kubectl get pods
```
```bash
Output: 
NAME        READY   STATUS    RESTARTS   AGE
pod-nginx   1/1     Running   0          2m17s
```

### To Get More Information
```bash
kubectl describe pod pod-nginx
```
```bash
# Output
Name:         pod-nginx
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Fri, 06 Jan 2023 12:29:39 +0800
Labels:       run=pod-nginx
Annotations:  <none>
Status:       Running
IP:           172.17.0.17
IPs:
  IP:  172.17.0.17
Containers:
  pod-nginx:
    Container ID:   docker://4cf374fe28c7cfea72e9c9e099de1fcafdf9929e9376d10f90e0a1464d5f654d
    Image:          butch/sdc-nginx:1.0.0
    Image ID:       docker-pullable://butch/sdc-nginx@sha256:6550554427248228c13cf8f8a95dac922c142ddb947db938add3d4637a6be9cd
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 06 Jan 2023 12:29:52 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wljl2 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-wljl2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m27s  default-scheduler  Successfully assigned default/pod-nginx to minikube
  Normal  Pulling    4m27s  kubelet            Pulling image "butch/sdc-nginx:1.0.0"
  Normal  Pulled     4m16s  kubelet            Successfully pulled image "butch/sdc-nginx:1.0.0" in 11.486351163s
  Normal  Created    4m15s  kubelet            Created container pod-nginx
  Normal  Started    4m15s  kubelet            Started container pod-nginx
```

### Access Application
 - To Access application, we need to create K8s Service.
  ```bash
  kubectl expose pod pod-nginx --type=NodePort --port=80 --name=svc-nginx 
  ```
 - Get Service Info
  ```bash
  kubectl get service
  ```
  ```bash
  # Output
  NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
  svc-nginx   NodePort   10.110.22.228   <none>        80:30821/TCP   88s
  ```
 - Get Public IP of Worker Node 
  ```bash
  kubectl get nodes -o wide
  ```
  ```bash
  # Output
  NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
  minikube   Ready    control-plane   23d   v1.25.3   192.168.49.2   <none>        Ubuntu 20.04.5 LTS   5.15.0-57-generic   docker://20.10.20
  ```
 - Access Using Node Public IP + NodePort
  ```bash
  curl -vvv http://192.168.49.2:30821
  ```
  ```bash
  # Output
  *   Trying 192.168.49.2:30821...
  * Connected to 192.168.49.2 (192.168.49.2) port 30821 (#0)
  > GET / HTTP/1.1
  > Host: 192.168.49.2:30821
  > User-Agent: curl/7.81.0
  > Accept: */*
  > 
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.18.0
  < Date: Fri, 06 Jan 2023 04:44:49 GMT
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

### Check Pod Logs
```bash
# Normal checking the pods.
kubectl logs pod-nginx
```
```bash
# Unfortunately, we're not sending logs to console, instead check the actual log file.
kubectl exec pod-nginx -- cat /var/log/nginx/access.log
```
```bash
# Output
172.17.0.1 - - [06/Jan/2023:04:42:50 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36"
172.17.0.1 - - [06/Jan/2023:04:42:50 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://192.168.49.2:30821/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36"
172.17.0.1 - - [06/Jan/2023:04:44:49 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.81.0"
```

### Get YAML Output of Pod & Service (Declarative Approach)
 - Pod: pod-nginx
  ```bash
  # Get pod definition YAML output
  kubectl get pod pod-nginx -o yaml > pod-nginx.yaml

  # Check pod-nginx.yaml
  cat pod-nginx.yaml
  ```
  ```bash
  # Output
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: "2023-01-06T04:29:39Z"
      labels:
        run: pod-nginx
      name: pod-nginx
      namespace: default
      resourceVersion: "222530"
      uid: 913212ea-78ac-43e4-becb-434f86e2324c
    spec:
      containers:
      - image: butch/sdc-nginx:1.0.0
        imagePullPolicy: IfNotPresent
        name: pod-nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-wljl2
          readOnly: true
      dnsPolicy: ClusterFirst
      enableServiceLinks: true
      nodeName: minikube
      preemptionPolicy: PreemptLowerPriority
      priority: 0
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: default
      serviceAccountName: default
      terminationGracePeriodSeconds: 30
      tolerations:
      - effect: NoExecute
        key: node.kubernetes.io/not-ready
        operator: Exists
        tolerationSeconds: 300
      - effect: NoExecute
        key: node.kubernetes.io/unreachable
        operator: Exists
        tolerationSeconds: 300
      volumes:
      - name: kube-api-access-wljl2
        projected:
          defaultMode: 420
          sources:
          - serviceAccountToken:
              expirationSeconds: 3607
              path: token
          - configMap:
              items:
              - key: ca.crt
                path: ca.crt
              name: kube-root-ca.crt
          - downwardAPI:
              items:
              - fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
                path: namespace
    status:
      conditions:
      - lastProbeTime: null
        lastTransitionTime: "2023-01-06T04:29:39Z"
        status: "True"
        type: Initialized
      - lastProbeTime: null
        lastTransitionTime: "2023-01-06T04:29:53Z"
        status: "True"
        type: Ready
      - lastProbeTime: null
        lastTransitionTime: "2023-01-06T04:29:53Z"
        status: "True"
        type: ContainersReady
      - lastProbeTime: null
        lastTransitionTime: "2023-01-06T04:29:39Z"
        status: "True"
        type: PodScheduled
      containerStatuses:
      - containerID: docker://4cf374fe28c7cfea72e9c9e099de1fcafdf9929e9376d10f90e0a1464d5f654d
        image: butch/sdc-nginx:1.0.0
        imageID: docker-pullable://butch/sdc-nginx@sha256:6550554427248228c13cf8f8a95dac922c142ddb947db938add3d4637a6be9cd
        lastState: {}
        name: pod-nginx
        ready: true
        restartCount: 0
        started: true
        state:
          running:
            startedAt: "2023-01-06T04:29:52Z"
      hostIP: 192.168.49.2
      phase: Running
      podIP: 172.17.0.17
      podIPs:
      - ip: 172.17.0.17
      qosClass: BestEffort
      startTime: "2023-01-06T04:29:39Z"
  ```
  - Service: svc-nginx
    ```bash
    # Get Service Definition
    kubectl get service svc-nginx -o yaml > svc-nginx.yaml
    
    # Check Service: svc-nginx
    cat  svc-nginx.yaml
    ```
    ```bash
    # Output
      apiVersion: v1
      kind: Service
      metadata:
        creationTimestamp: "2023-01-06T04:37:33Z"
        labels:
          run: pod-nginx
        name: svc-nginx
        namespace: default
        resourceVersion: "222919"
        uid: 007ebfe3-6617-4369-ae33-cd362ce270d4
      spec:
        clusterIP: 10.110.22.228
        clusterIPs:
        - 10.110.22.228
        externalTrafficPolicy: Cluster
        internalTrafficPolicy: Cluster
        ipFamilies:
        - IPv4
        ipFamilyPolicy: SingleStack
        ports:
        - nodePort: 30821
          port: 80
          protocol: TCP
          targetPort: 80
        selector:
          run: pod-nginx
        sessionAffinity: None
        type: NodePort
      status:
        loadBalancer: {}
    ```
    
 ### Clean Up
 ```bash
 kubectl delete pod pod-nginx
 # Output: pod "pod-nginx" deleted
 kubectl delete service svc-nginx
 # Output: service "svc-nginx" deleted
 ```
