# EKS Cluster

### Prerequisite
 - [aws-cli](https://aws.amazon.com/cli/) - command line tool to help managing your AWS services. 
 - [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html) -  command line tool for creating and managing Kubernetes clusters on Amazon EKS.
 - [kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html) - command line tool that you use to communicate with the Kubernetes API server.



#### Step 1: Creating Cluster
```bash
# Template
$ eksctl create cluster \
  --name <cluster-name> \
  --region <target-aws-region> \
  --version <k8s-version> \
  --without-nodegroup

# Example: 
$ eksctl create cluster \
    --name app-cluster \
    --region ap-southeast-1 \
    --version 1.24 \
    --without-nodegroup \
    --tags "Project=Cap Build,Topic=EKS"

    
# Cluster provisioning takes several minutes. While the cluster is being created, several lines of output appear.
# The last line of output is similar to the following example line.    
# [✓]  EKS cluster "app-cluster" in "ap-southeast-1" region is ready 
 
 
# To verify, get cluster list.
$ eksctl get cluster
---
NAME		REGION		EKSCTL CREATED
app-cluster	ap-southeast-1	True
```

### Step 2: Creating an IAM OIDC provider for your cluster.
> To use some Amazon EKS add-ons, or to enable individual Kubernetes workloads to have specific AWS Identity and Access Management (IAM) permissions
```bash
# Template
$ eksctl utils associate-iam-oidc-provider --cluster <cluster-name> --approve

# Example:
$  eksctl utils associate-iam-oidc-provider --cluster app-cluster --approve
-----
# [✔]  created IAM Open ID Connect provider for cluster "app-cluster" in "ap-southeast-1"
```

### Step 3: Create EC2 Keypair
> This will use to EKS Nodes as SSH-Key.

```bash
# Template
$ aws ec2 create-key-pair \
    --key-name <key-pair-name> \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > <key-pair-name>.pem
    
# Example
$ aws ec2 create-key-pair \
    --key-name eks \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > eks.pem
    
    
# Change Permission 
$ chmod 400 eks.pem
```

### Step 4: Create Node Group with additional Add-Ons in Public Subnets
> Automate the provisioning and lifecycle management of nodes (Amazon EC2 instances) for Amazon EKS Kubernetes clusters.

**Create Public Node Group**
```bash
# Template
$ eksctl create nodegroup \
    --cluster=<cluster-name>  \
    --region=<aws-region> \
    --name=<nodegroup-name> \
    --spot \
    --instance-types=<spot-instance-types>\
    --nodes=<no-of-nodes> \
    --nodes-min=<min-no-of-nodes> \
    --nodes-max=<max-no-of-nodes> \
    --node-volume-size=<node-volume-size> \
    --ssh-access \
    --ssh-public-key=<ssh-key-name> \
    --managed \
    --asg-access \ 
    --external-dns-access \
    --full-ecr-access \
    --appmesh-access \
    --alb-ingress-access 
    
# Example    
$  eksctl create nodegroup \
    --cluster=app-cluster \
    --region=ap-southeast-1 \
    --name=app-cluster-ng-public1 \
    --spot \
    --instance-types=t3.medium \
    --nodes=1 \
    --nodes-min=1 \
    --nodes-max=4 \
    --node-volume-size=20 \
    --ssh-access \
    --ssh-public-key=eks \
    --managed \
    --asg-access \
    --external-dns-access \
    --full-ecr-access \
    --appmesh-access \
    --alb-ingress-access \
    --tags "Project=Cap Build,Topic=EKS"
    
# To Verify, get list NodeGroups in a cluster.
$ eksctl get nodegroups --cluster=app-cluster
---
CLUSTER		NODEGROUP		STATUS	CREATED			MIN SIZE	MAX SIZE	DESIRED CAPACITY	INSTANCE TYPE	IMAGE ID	ASG NAME							TYPE
app-cluster	app-cluster-ng-public1	ACTIVE	2023-01-06T03:49:32Z	1		4		1			t3.medium	AL2_x86_64	eks-app-cluster-ng-public1-bac2c299-55c5-9493-afb3-76824d7b9de8	managed

# List Nodes in current kubernetes cluster
$ kubectl get nodes -o wide
---
NAME                                                STATUS   ROLES    AGE   VERSION               INTERNAL-IP      EXTERNAL-IP     OS-IMAGE         KERNEL-VERSION                 CONTAINER-RUNTIME
ip-192-168-32-189.ap-southeast-1.compute.internal   Ready    <none>   18m   v1.24.7-eks-fb459a0   192.168.32.189   54.179.130.81   Amazon Linux 2   5.4.226-129.415.amzn2.x86_64   containerd://1.6.6
```

### Step 5: Create Node Group in Private Subnets

Key option for the command is `--node-private-networking`
```bash
 eksctl create nodegroup \
    --cluster=app-cluster \
    --region=ap-southeast-1 \
    --name=app-cluster-ng-private1 \
    --spot \
    --instance-types=t3.medium \
    --nodes=1 \
    --nodes-min=1 \
    --nodes-max=2 \
    --node-volume-size=20 \
    --ssh-access \
    --ssh-public-key=eks \
    --managed \
    --asg-access \
    --external-dns-access \
    --full-ecr-access \
    --appmesh-access \
    --alb-ingress-access \
    --node-private-networking \
    --tags "Project=Cap Build,Topic=EKS"
```


### Step 6: Terminate NodeGroup & Cluster
```bash
# Get available clusters
$ eksctl get clusters

# Get avaiable nodegroups within cluster
$ eksctl get nodegroup --cluster=<cluster-name>

# Delete Nodegroup
$ eksctl delete nodegroup --cluster=<cluster-name> --name=<nodegroup-name>

# Once all nodegroup are deleted, delete cluster
$ eksctl delete cluster <cluster-name>
