| Kubernetes Object  | YAML File |
| ------------- | ------------- |
| Storage Class  | 01-storage-class.yml |
| Persistent Volume Claim | 02-persistent-volume-claim.yml   |
| Config Map  | 03-drop-db-config-map.yml  |
| Deployment, Environment Variables, Volumes, VolumeMounts  | 04-deployment.yml  |
| ClusterIP Service  | 05-service.yml  |


### Commands
 1. To provision the ff k8s objects above.
    ```bash
    kubectl apply -f src/mysql
    ```
 2. To verify the k8s objects creation.
    - Get Storage Classes 
      ```bash
      kubectl get sc
      ```
      ```bash
      # Output
      NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
      ebs-sc          ebs.csi.aws.com         Delete          WaitForFirstConsumer   false                  17m
      ```
    - Get Persistent Volume Claims
      ```bash
      kubectl get pvc
      ```
      ```bash
      # Output
      NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
      ebs-mysql-pv-claim   Bound    pvc-2168b867-d7c2-4ee1-99aa-1109d90f1292   4Gi        RWO            ebs-sc         18m
      ```
    - Get Persistent Volume
      ```bash
      kubectl get pv
      ```
      ```bash
      # Output
      NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                        STORAGECLASS   REASON   AGE
      pvc-2168b867-d7c2-4ee1-99aa-1109d90f1292   4Gi        RWO            Delete           Bound    default/ebs-mysql-pv-claim   ebs-sc                  17m
      ```
    - Get MySql Pod
      ```bash
      kubectl get pods 
      ```
      ```bash
      # Output
      NAME                     READY   STATUS    RESTARTS   AGE
      mysql-58c65d6ff6-9g6m5   1/1     Running   0          19m
      ```
 3. To connect to database.
    ```bash
    kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword
    ```
    ```bash
    # Output
    If you don't see a command prompt, try pressing enter.

    mysql> 
    ```
 4. Show database databases.
    ```mysql
    show databases;
    ```
    ```mysql
    mysql> show databases;
    +---------------------+
    | Database            |
    +---------------------+
    | information_schema  |
    | #mysql50#lost+found |
    | mysql               |
    | performance_schema  |
    | usermgmt            |
    +---------------------+
    5 rows in set (0.00 sec)
    mysql> 
    ```

