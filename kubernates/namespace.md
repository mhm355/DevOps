# Namespaces

* is a virtual cluster backed by the same physical cluster.

* it provides mechanism for isolating groups of resources within the same cluster.

* name of resources should be unique within the namespace.

* namespaces cannot be nested inside one another.

* each Kubernetes resource can only be in one namespace.

> when use it : Namespaces are intended for use in environments with many users spread across multiple teams, or projects.



* K8S starts with four initial namespaces.

    * default : can use cluster without creating a new namespace
    
    * kube-node-lease : This namespace holds `Lease objects` associated with each node. Node leases allow the `kubelet` to send `heartbeats` so that the `control plane` can detect node failure. 

        > Leases

        are the primary mechanism for `kubelets` to signal their health and availability to the Kubernetes `control plane`.

        provide a lightweight and efficient way for distributed components within Kubernetes.

        reduce the overhead on the API server and etcd.

    * kube-public : it is readable by all clients, reserved for cluster usage, sensitive data should not be placed in this namespace. such as `cluster-info ConfigMap`

    * kube-system : for objects created by the Kubernetes system, includes core components like the API server, controller manager, scheduler, and essential addons like `kube-dns, kube-proxy`. it is not for general use.


## Working with namespaces.

1. `kubectl create namespace <namespace-name>` : creat new namespace.

2. `kubectl get namespaces` : show all namespaces in the cluster.

3. `kubectl get <resource-type> <resource-name> --namespace <or -n> <namespace-name>` 

    that show the resouce in the namespace ex: `kubectl get pods -n default`

4. `kubectl config set-context --current --namespace=<namespace-name>` : set namespace for the current context. When executing subsequent `kubectl` commands without explicitly specifying a namespace. we can verify it using `kubectl config view --minify | grep namespace:`








