#### K8s Object
 > The term **object** is a reference to a **thing** that exists inside K8s cluster.

*Object serve different purposes such as:*
   - running containers
   - monitoring containers
   - setting up networking
   - etc.
---

**Object Types**
 - StatefulSet
 - ReplicaController
 - Pod - runs one or more very closely related containers.(*Use Deployment Instead of Pod*)
 - Deployment - Maintains a set of identical pods, ensuring that they have the correct config and the right number exists.

 - Service - Sets up networking in a K8s cluster.
   - ClusterIP - Exposes a set of pods to other objects in the cluster.
   - NodePort - Exposes a container to the outside world **(Only good for dev purposes!!!)**
   - LoadBalancer
   - Ingress 


##### Pods vs Deployments
 - Pods
   - Runs a single set of containers
   - Good for one-off dev purposes
   - Rarely used directly in production
 - Deployment
   - Runs a set of identical pods (1 or more)
   - Monitors the state of each pod, updating as necessary.
   - Good for dev & production
  
---

#### API VERSION
Scopes or limit types of objects that we can specify that we want to create with any configuration file.

**Each API version defines a different set of 'object' we can use.**

 - apiVersion: v1
   - componentStatus
   - configMap
   - Endpoints
   - Event
   - Namespace
   - Pod

 - apiVersion: apps/v1
   - ControllerRevision
   - StatefulSet
   - Deployment
