Deploy AWS ALB Ingress controller
---
Let’s deploy the AWS ALB Ingress controller into our EKS cluster using the steps below.


### Create Service Account with IAM Role

 - Deploy the relevant RBAC roles and role bindings as required by the AWS ALB Ingress controller.
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
    ```

 - Create an IAM policy named `AWSLoadBalancerControllerIAMPolicy` to allow the ALB Ingress controller to make AWS API calls on your behalf
    ```bash
    aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

    # If you want specific version: https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/install/iam_policy.json
    ```
    ```json
    // Output
    {
        "Policy": {
            "PolicyName": "AWSLoadBalancerControllerIAMPolicy", 
            "PermissionsBoundaryUsageCount": 0, 
            "CreateDate": "2023-01-13T07:40:01Z", 
            "AttachmentCount": 0, 
            "IsAttachable": true, 
            "PolicyId": "ANPAXE26SPP7QVXKG6HOK", 
            "DefaultVersionId": "v1", 
            "Path": "/", 
            "Arn": "arn:aws:iam::491435228159:policy/AWSLoadBalancerControllerIAMPolicy", 
            "UpdateDate": "2023-01-13T07:40:01Z"
        }
    }
    ```
    Record the `Policy.Arn` in the command output, you will need it in the next step 

 - Create a `Kubernetes service account` and an `IAM role` (for the pod running the AWS ALB Ingress controller) by substituting `$PolicyARN` with the recorded value from the previous step:
    ```bash
    eksctl create iamserviceaccount \
       --cluster=app-cluster \
       --namespace=kube-system \
       --name=aws-load-balancer-controller-sa \
       --attach-policy-arn=$PolicyARN \
       --override-existing-serviceaccounts \
       --approve
    ```
    ```bash
    # Output
    2023-01-13 15:42:14 [ℹ]  1 iamserviceaccount (kube-system/aws-load-balancer-controller-sa) was included (based on the include/exclude rules)
    2023-01-13 15:42:14 [!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
    2023-01-13 15:42:14 [ℹ]  1 task: { 
        2 sequential sub-tasks: { 
            create IAM role for serviceaccount "kube-system/aws-load-balancer-controller-sa",
            create serviceaccount "kube-system/aws-load-balancer-controller-sa",
        } }2023-01-13 15:42:14 [ℹ]  building iamserviceaccount stack "eksctl-app-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller-sa"
    2023-01-13 15:42:14 [ℹ]  deploying stack "eksctl-app-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller-sa"
    2023-01-13 15:42:14 [ℹ]  waiting for CloudFormation stack "eksctl-app-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller-sa"
    2023-01-13 15:42:45 [ℹ]  waiting for CloudFormation stack "eksctl-app-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller-sa"
    2023-01-13 15:43:43 [ℹ]  waiting for CloudFormation stack "eksctl-app-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller-sa"
    2023-01-13 15:43:44 [ℹ]  created serviceaccount "kube-system/aws-load-balancer-controller-sa"
    ```

    Verify k8s Service Account using kubectl.
    ```bash
    kubectl describe sa aws-load-balancer-controller-sa \
        -n kube-system 
    ```
    ```bash
    # Output
    Name:                aws-load-balancer-controller-sa
    Namespace:           kube-system
    Labels:              app.kubernetes.io/managed-by=eksctl
    Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::491435228159:role/eksctl-app-cluster-addon-iamserviceaccount-k-Role1-172W78J7YHH9S
    Image pull secrets:  <none>
    Mountable secrets:   <none>
    Tokens:              <none>
    Events:              <none>
    ```

### Install AWS Load Balancer Controller via HELM
 - Install aws-load-balancer-controller
    ```bash
    # Add the eks-charts repository.
    helm repo add eks https://aws.github.io/eks-charts

    # Update your local repo to make sure that you have the most recent charts.
    helm repo Update

     helm install aws-load-balancer-controller \
        eks/aws-load-balancer-controller \
        -n kube-system \
        --set clusterName=app-cluster \
        --set serviceAccount.create=false \
        --set serviceAccount.name=aws-load-balancer-controller-sa \
        --set region=ap-southeast-1 \
        --set vpcId=vpc-06fc0020e9f69b4fd \
        --set image.repository=602401143452.dkr.ecr.ap-southeast-1.amazonaws.com/amazon/aws-load-balancer-controller
    ```
    For region and code of image, please refer on this [link](https://docs.aws.amazon.com/eks/latest/userguide/add-ons-images.html)

    ```bash
    # Output
    NAME: aws-load-balancer-controller
    LAST DEPLOYED: Fri Jan 13 15:58:12 2023
    NAMESPACE: kube-system
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    AWS Load Balancer controller installed!
    ```
 -  Verify that the controller is installed:
    ```bash
    kubectl -n kube-system get deployment aws-load-balancer-controller
    ```
    ```bash
    # Output
    Name:                   aws-load-balancer-controller
    Namespace:              kube-system
    CreationTimestamp:      Fri, 13 Jan 2023 15:58:14 +0800
    Labels:                 app.kubernetes.io/instance=aws-load-balancer-controller
                            app.kubernetes.io/managed-by=Helm
                            app.kubernetes.io/name=aws-load-balancer-controller
                            app.kubernetes.io/version=v2.4.6
                            helm.sh/chart=aws-load-balancer-controller-1.4.7
    Annotations:            deployment.kubernetes.io/revision: 1
                            meta.helm.sh/release-name: aws-load-balancer-controller
                            meta.helm.sh/release-namespace: kube-system
    Selector:               app.kubernetes.io/instance=aws-load-balancer-controller,app.kubernetes.io/name=aws-load-balancer-controller
    Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
    Labels:           app.kubernetes.io/instance=aws-load-balancer-controller
                        app.kubernetes.io/name=aws-load-balancer-controller
    Annotations:      prometheus.io/port: 8080
                        prometheus.io/scrape: true
    Service Account:  aws-load-balancer-controller-sa
    Containers:
    aws-load-balancer-controller:
        Image:       public.ecr.aws/eks/aws-load-balancer-controller:v2.4.6
        Ports:       9443/TCP, 8080/TCP
        Host Ports:  0/TCP, 0/TCP
        Command:
        /controller
        Args:
        --cluster-name=app-cluster
        --ingress-class=alb
        --aws-region=ap-southeast-1
        --aws-vpc-id=vpc-06fc0020e9f69b4fd
        Liveness:     http-get http://:61779/healthz delay=30s timeout=10s period=10s #success=1 #failure=2
        Environment:  <none>
        Mounts:
        /tmp/k8s-webhook-server/serving-certs from cert (ro)
    Volumes:
    cert:
        Type:               Secret (a volume populated by a Secret)
        SecretName:         aws-load-balancer-tls
        Optional:           false
    Priority Class Name:  system-cluster-critical
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   aws-load-balancer-controller-75b5fc9cf8 (2/2 replicas created)
    Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  99s   deployment-controller  Scaled up replica set aws-load-balancer-controller-75b5fc9cf8 to 2

    ```
 - Verify AWS Load Balancer Controller Webhook service created
    ```bash
    kubectl -n kube-system get svc aws-load-balancer-webhook-service
    ```
    ```bash
    # Output
    NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    aws-load-balancer-webhook-service   ClusterIP   10.100.38.250   <none>        443/TCP   3m27s
    ```

 - Verify AWS Load Balancer Controller Logs
    ```bash
    # List Pods
    kubectl get pods -n kube-system | grep aws-load-balancer-controller

    # Output
    aws-load-balancer-controller-75b5fc9cf8-rntcc   1/1     Running   0          7m18s
    aws-load-balancer-controller-75b5fc9cf8-tsp4x   1/1     Running   0          7m18s
    ``` 

    ```bash
     kubectl -n kube-system logs -f aws-load-balancer-controller-75b5fc9cf8-rntcc
    ```
    ```bash
    # Output
    {"level":"info","ts":1673596695.9520242,"msg":"version","GitVersion":"v2.4.6","GitCommit":"a92e689dfe464f5b24784f398947e0fef31dc470","BuildDate":"2023-01-12T06:29:16+0000"}
    {"level":"info","ts":1673596695.9800384,"logger":"controller-runtime.metrics","msg":"metrics server is starting to listen","addr":":8080"}
    {"level":"info","ts":1673596695.984325,"logger":"setup","msg":"adding health check for controller"}
    {"level":"info","ts":1673596695.9844737,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/mutate-v1-pod"}
    {"level":"info","ts":1673596695.9845953,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/mutate-elbv2-k8s-aws-v1beta1-targetgroupbinding"}
    {"level":"info","ts":1673596695.9847312,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/validate-elbv2-k8s-aws-v1beta1-targetgroupbinding"}
    {"level":"info","ts":1673596695.9850075,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/validate-networking-v1-ingress"}
    {"level":"info","ts":1673596695.9851177,"logger":"setup","msg":"starting podInfo repo"}
    I0113 07:58:17.985462       1 leaderelection.go:243] attempting to acquire leader lease kube-system/aws-load-balancer-controller-leader...
    {"level":"info","ts":1673596697.9858274,"msg":"starting metrics server","path":"/metrics"}
    {"level":"info","ts":1673596697.9858983,"logger":"controller-runtime.webhook.webhooks","msg":"starting webhook server"}
    {"level":"info","ts":1673596697.9862094,"logger":"controller-runtime.certwatcher","msg":"Updated current TLS certificate"}
    {"level":"info","ts":1673596697.9862907,"logger":"controller-runtime.webhook","msg":"serving webhook server","host":"","port":9443}
    {"level":"info","ts":1673596697.986575,"logger":"controller-runtime.certwatcher","msg":"Starting certificate watcher"}
    ```

### UNINSTALL AWS Load Balancer Controller using Helm
```bash
# Uninstall AWS Load Balancer Controller & This step should not be implemented.
helm uninstall aws-load-balancer-controller -n kube-system 
```