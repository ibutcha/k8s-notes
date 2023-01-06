## Deployment

### Rollback Deployment
 - Check rollback history
    ```bash
    kubectl rollout history deployment/nginx-deployment
    ```
    ```bash
    # Output
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>
    ```
 - Verify Changes
    - Revision 1
        ```bash
        kubectl rollout history deployment/nginx-deployment --revision=1
        ```
        ```yaml
        # Ouput
        Pod Template:
        Labels:	app=nginx-deployment
            pod-template-hash=7d6c9564c
        Containers:
        sdc-nginx:
            Image:	butch/sdc-nginx:1.0.0
            Port:	<none>
            Host Port:	<none>
            Environment:	<none>
            Mounts:	<none>
        Volumes:	<none>
        ```
    - Revision 2
        ```bash
        kubectl rollout history deployment/nginx-deployment --revision=2
        ```
        ```yaml
        # Output
        Pod Template:
        Labels:	app=nginx-deployment
            pod-template-hash=66d5c8bfc5
        Containers:
        sdc-nginx:
            Image:	nginx:alpine
            Port:	<none>
            Host Port:	<none>
            Environment:	<none>
            Mounts:	<none>
        Volumes:	<none>
        ```
 - Rollback to previous version
    ```bash
    kubectl rollout undo deployment/nginx-deployment --to-revision=1
    # Output: deployment.apps/nginx-deployment rolled back
    ```
 - Verify Deployment
    ```bash
    kubectl describe deployment nginx-deployment
    ```
    ```bash
    # Output
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Fri, 06 Jan 2023 14:41:20 +0800
    Labels:                 app=nginx-deployment
    Annotations:            deployment.kubernetes.io/revision: 3
    Selector:               app=nginx-deployment
    Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
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
    NewReplicaSet:   nginx-deployment-7d6c9564c (5/5 replicas created)
    Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  29m   deployment-controller  Scaled up replica set nginx-deployment-7d6c9564c to 1
    Normal  ScalingReplicaSet  29m   deployment-controller  Scaled up replica set nginx-deployment-7d6c9564c to 5 from 1
    Normal  ScalingReplicaSet  19m   deployment-controller  Scaled up replica set nginx-deployment-66d5c8bfc5 to 2
    Normal  ScalingReplicaSet  19m   deployment-controller  Scaled down replica set nginx-deployment-7d6c9564c to 4 from 5
    Normal  ScalingReplicaSet  19m   deployment-controller  Scaled up replica set nginx-deployment-66d5c8bfc5 to 3 from 2
    Normal  ScalingReplicaSet  19m   deployment-controller  Scaled down replica set nginx-deployment-7d6c9564c to 3 from 4
    Normal  ScalingReplicaSet  19m   deployment-controller  Scaled up replica set nginx-deployment-66d5c8bfc5 to 4 from 3
    Normal  ScalingReplicaSet  19m   deployment-controller  Scaled down replica set nginx-deployment-7d6c9564c to 2 from 3
    Normal  ScalingReplicaSet  19m   deployment-controller  Scaled up replica set nginx-deployment-66d5c8bfc5 to 5 from 4
    Normal  ScalingReplicaSet  19m   deployment-controller  Scaled down replica set nginx-deployment-7d6c9564c to 1 from 2
    Normal  ScalingReplicaSet  18m   deployment-controller  Scaled down replica set nginx-deployment-7d6c9564c to 0 from 1
    Normal  ScalingReplicaSet  81s   deployment-controller  Scaled up replica set nginx-deployment-7d6c9564c to 2 from 0
    Normal  ScalingReplicaSet  81s   deployment-controller  Scaled down replica set nginx-deployment-66d5c8bfc5 to 4 from 5
    Normal  ScalingReplicaSet  81s   deployment-controller  Scaled up replica set nginx-deployment-7d6c9564c to 3 from 2
    Normal  ScalingReplicaSet  79s   deployment-controller  Scaled down replica set nginx-deployment-66d5c8bfc5 to 3 from 4
    Normal  ScalingReplicaSet  79s   deployment-controller  Scaled up replica set nginx-deployment-7d6c9564c to 4 from 3
    Normal  ScalingReplicaSet  79s   deployment-controller  Scaled down replica set nginx-deployment-66d5c8bfc5 to 2 from 3
    Normal  ScalingReplicaSet  79s   deployment-controller  Scaled up replica set nginx-deployment-7d6c9564c to 5 from 4
    Normal  ScalingReplicaSet  78s   deployment-controller  Scaled down replica set nginx-deployment-66d5c8bfc5 to 1 from 2
    Normal  ScalingReplicaSet  76s   deployment-controller  Scaled down replica set nginx-deployment-66d5c8bfc5 to 0 from 1

    ```       
