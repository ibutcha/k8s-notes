External DNS
---

### For Ingress
  - Add External DNS Annotation
    ```yaml
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: k8s.solutions-chapters.click
    ```
  - Deploy your changes
    ```bash
    kubectl -f <ingress>.yaml
    ``` 

### For Service
 - Add External DNS Annotation
    ```yaml
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: k8s.solutions-chapters.click
    ```
 - Make sure the `Service Type` is `LoadBalancer`.
    ```yaml
    # Example
    apiVersion: v1
    kind: Service
    metadata:
    name: app1-nginx-loadbalancer-service
    labels:
        app: app1-nginx
    annotations:
        external-dns.alpha.kubernetes.io/hostname: k8s.solutions-chapters.click
    spec:
    type: LoadBalancer # <=
    selector:
        app: app1-nginx
    ports:
        - port: 80
        targetPort: 80  
    ```

### Verify DNS Log
```bash
kubectl logs -f $(kubectl get po | egrep -o 'external-dns[A-Za-z0-9-]+')
```


