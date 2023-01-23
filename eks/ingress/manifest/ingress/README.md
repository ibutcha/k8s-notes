Ingress
---

### High-Level Diagram
![aws-alb-ingress](./../../../../images/k8s-aws-alb-ingress.png)


#### Routes
 - `/app1/*` - will route to nginx-app1
 - `/app2/*` - will route to nginx-app2
 - `/*` - will route to nginx-app3

Respective annotation alb.ingress.`kubernetes.io/healthcheck-path:` will be moved to respective application `NodePort Service`.

##### References
 - [Ingress Annotations](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/ingress/annotations/)