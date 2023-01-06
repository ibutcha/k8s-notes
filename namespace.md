Namespace
---
**Provides a mechanism for isolating groups of resources within a single cluster**. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).


#### Setting the namespace preference
```bash
$ kubectl config set-context --current --namespace=<insert-namespace-name-here>
# Validate it
$ kubectl config view --minify | grep namespace:
```


#### Namespaces and DNS
When you create a [Service](https://kubernetes.io/docs/concepts/services-networking/service/), it creates a corresponding [DNS entry](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/). This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`, which means that if a container only uses <service-name>, it will resolve to the service which is local to a namespace. This is useful for using the same configuration across multiple namespaces such as Development, Staging and Production. If you want to reach across namespaces, you need to use the fully qualified domain name (FQDN).
