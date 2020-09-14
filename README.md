# k8s-notes
 - **Nodes(Minions)** - is a machine (physical or virtual) on which k8s is installed. Also, it's a worker machine wherein k8s container will launch/reside.
 - **Cluster** is a set of nodes that group and connect together.

 - **kubectl**  is the Kubernetes `command-line tool` that allows you to run commands against `Kubernetes clusters`. You can use kubectl to `deploy applications`, `inspect`, `manage cluster resource`s, and `view logs`.
 
   Common commands:
    - kubectl version --client
    - kubectl get nodes   // to view the OS-IMAGE column, add `-o wide`
    - kubectl create deployment
    - kubectl expose deployment
    - kubectl get pod
    - kubectl delete services <service-name>
    - kubectl delete deployment <deployment-name>
    
  
 - **Minikube** is a `tool` that makes it easy to run Kubernetes locally. Minikube runs a **single-node Kubernetes cluster** inside a Virtual Machine (VM), docker, etc. on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

  Common commands:
   - minikube service <service-name> --url - Get the URL of the exposed `service`, to view the `service` details, remove --url.
   - minukube start - start minikube and create cluster.
   - minikube stop - Stop the local Minikube cluster.
   - minikube delete - Delete the local Minikube cluster.
