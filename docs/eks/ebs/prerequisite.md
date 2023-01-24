## Amazon EBS CSI Driver Set-Up

> Container Storage Interface (CSI) driver allows Amazon Elastic Kubernetes Service (Amazon EKS) clusters to manage the lifecycle of Amazon EBS volumes for persistent volumes.


<br />

### Step 1: Create EBS CSI DRIVER IAM Policy
 - Go to Services => IAM => Policies
 - Create Policy and paste the permission below.
    ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                "Effect": "Allow",
                "Action": [
                    "ec2:AttachVolume",
                    "ec2:CreateSnapshot",
                    "ec2:CreateTags",
                    "ec2:CreateVolume",
                    "ec2:DeleteSnapshot",
                    "ec2:DeleteTags",
                    "ec2:DeleteVolume",
                    "ec2:DescribeInstances",
                    "ec2:DescribeSnapshots",
                    "ec2:DescribeTags",
                    "ec2:DescribeVolumes",
                    "ec2:DetachVolume"
                ],
                "Resource": "*"
                }
            ]
        }
    ```
 - Name the policy as `k8s-amazon_ebs_csi_driver`
 - Add descrition: Policy for EC2 Instances to access Elastic Block Store.
 - Create Policy.


### Step 2:  Associate it with Node IAM Role
 - Get Worker node IAM Role ARN
   ```bash
   kubectl -n kube-system describe configmap aws-auth
   ```
   ```bash
   Name:         aws-auth
    Namespace:    kube-system
    Labels:       <none>
    Annotations:  <none>

    Data
    ====
    mapRoles:
    ----
    - groups:
    - system:bootstrappers
    - system:nodes
    rolearn: arn:aws:iam::<AWS-ACCOUNT_ID>:role/eksctl-app-cluster-nodegroup-app-NodeInstanceRole-8RS8LUB7M4GX
    username: system:node:{{EC2PrivateDNSName}}


    BinaryData
    ====

    Events:  <none>
   ```
 - Search for it to Services => IAM => Roles
 - Attach `k8s-amazon_ebs_csi_driver` Policy and Save.


### Step 3: Deploy Amazon EBS CSI Driver
*You can install it using [AWS Manage Add-On](https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html) directly to EKS cluster Managment console or manually create it and follow steps below*

 - Deploy EBS CSI Driver
   ```bash
   kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"
   ```
   ```bash
    serviceaccount/ebs-csi-controller-sa created
    serviceaccount/ebs-csi-node-sa created
    clusterrole.rbac.authorization.k8s.io/ebs-csi-node-role created
    clusterrole.rbac.authorization.k8s.io/ebs-external-attacher-role created
    clusterrole.rbac.authorization.k8s.io/ebs-external-provisioner-role created
    clusterrole.rbac.authorization.k8s.io/ebs-external-resizer-role created
    clusterrole.rbac.authorization.k8s.io/ebs-external-snapshotter-role created
    clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-attacher-binding created
    clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-node-getter-binding created
    clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-provisioner-binding created
    clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-resizer-binding created
    clusterrolebinding.rbac.authorization.k8s.io/ebs-csi-snapshotter-binding created
    deployment.apps/ebs-csi-controller created
    poddisruptionbudget.policy/ebs-csi-controller created
    daemonset.apps/ebs-csi-node created
    csidriver.storage.k8s.io/ebs.csi.aws.com created
   ```
 - Verify ebs-csi pods running
   ```bash
   kubectl get pods -n kube-system
   ```
   ```bash
   NAME                                  READY   STATUS    RESTARTS   AGE
    aws-node-5lxx4                        1/1     Running   0          34m
    coredns-6c97f4f789-24j9d              1/1     Running   0          45m
    coredns-6c97f4f789-cpk4m              1/1     Running   0          45m
    ebs-csi-controller-5c67d6d86b-fb7fl   6/6     Running   0          2m34s
    ebs-csi-controller-5c67d6d86b-zbj2l   6/6     Running   0          2m34s
    ebs-csi-node-zql8v                    3/3     Running   0          2m34s
    kube-proxy-qczzj                      1/1     Running   0          34m

   ```
