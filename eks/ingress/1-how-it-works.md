How Kubernetes Ingress works with [aws-alb-ingress-controller](https://github.com/kubernetes-sigs/aws-load-balancer-controller)
---

 ![k8S components](./../../images/aws-alb-ingress-controller.png)

 The following diagram details the AWS components that the `aws-alb-ingress-controller` creates whenever an Ingress resource is defined by the user. The Ingress resource routes ingress traffic from the ALB to the Kubernetes cluster.


#### Ingress Creation Process

Following the steps in the numbered blue circles in the above diagram:

 1. The controller watches for [Ingress events](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-controllers) from the API server. When it finds Ingress resources that satisfy its requirements, it starts the creation of AWS resources.
An ALB is created for the Ingress resource.

 2. An ALB is created for the Ingress resource.

 3. [TargetGroups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) are created for each backend specified in the Ingress resource.

 4. [Listeners](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html) are created for every port specified as Ingress resource annotation. If no port is specified, sensible defaults (80 or 443) are used.

 5. [Rules](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-update-rules.html) are created for each path specified in your Ingress resource. This ensures that traffic to a specific path is routed to the correct `TargetGroup` created.

### Ingress Traffic
AWS ALB Ingress controller supports two traffic modes: instance mode and ip mode. Users can explicitly specify these traffic modes by declaring the alb.ingress.kubernetes.io/target-type annotation on the Ingress and the service definitions. 

 - `instance mode`: Ingress traffic starts from the ALB and reaches the NodePort opened for your service. Traffic is then routed to the pods within the cluster.

 - `ip mode`: Ingress traffic starts from the ALB and reaches the pods within the cluster directly. To use this mode, the networking plugin for the Kubernetes cluster must use a secondary IP address on ENI as pod IP, also known as the [AWS CNI plugin for Kubernetes](https://github.com/aws/amazon-vpc-cni-k8s).
   > Note: If you are using the ALB ingress with EKS on Fargate you want to use ip mode.

<br/>

#### References:
 - [Kubernetes Ingress with AWS ALB Ingress Controller](https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/#:~:text=Ingress%20Creation&text=The%20controller%20watches%20for%20Ingress,specified%20in%20the%20Ingress%20resource.)

