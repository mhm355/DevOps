

## High Availability (HA) 

* refers to the design and implementation of a cluster to ensure continuous operation and minimize downtime even in the event of component failures.


### 1. Stacked etcd topology

![kubeadm-ha-topology-stacked-etcd](./screenshots/kubeadm-ha-topology-stacked-etcd.svg)

* distributed data storage cluster provided by etcd is stacked on top of the cluster formed by the nodes managed by kubeadm that run control plane components.

* each `control plane node` runs an instance of the `kube-apiserver`, `kube-scheduler`, and `kube-controller-manager`. The `kube-apiserver` is `exposed to worker nodes using a load balancer`.

* Each `control plane node` creates a `local etcd member` and this `etcd member` communicates `only` with the `kube-apiserver`

    > _If one node goes down, both an etcd member and a control plane instance are lost, and redundancy is compromised_ we can mitigate this risk by adding more control plane nodes.


### 2. External etcd topology

![kubeadm-ha-topology-external-etcd](./screenshots/kubeadm-ha-topology-external-etcd.svg)

* the distributed data storage cluster provided by etcd is external to the cluster formed by the nodes that run control plane components.

* each `control plane node` in an `external etcd topology` runs an instance of the `kube-apiserver`, `kube-scheduler`, and `kube-controller-manager`. And the `kube-apiserver` is `exposed to worker nodes` using a `load balancer`.

* each `etcd host` communicates with the `kube-apiserver` of each `control plane node`.

* losing a `control plane instance` or an `etcd member` has `less impact` and `does not affect the cluster redundancy` as much as the stacked HA topology.

* requires `twice the number of hosts` as the `stacked HA topology`



## Tools

1. kubectl : the offical cli tool 

2. minikube : used to create the k8s cluster in single node

3. kubadm : used to create the cluster , for production level

4. Helm : used to create k8s template, package mangement and convert k8s objects into usable templates

5. kompose : help to transfer docker-compose files into k8s objects 

6. kustomize : configuration management for k8s objects




## Manage Cluster

1. Drain Nodes

* Evicts all pods 'non-daemonset pods' from a node  and then marks the node as "unschedulable" This prevents new pods from being scheduled onto that node

* used to perform maintenance that requires the node to be completely free of workloads, such as OS upgrades, hardware replacement, or kernel patching.

* `kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data --force  --grace-period=0`

    `--ignore-daemonsets` : DaemonSet-managed pods are not evicted.
    `--delete-emptydir-data` : Deletes pods with emptyDir volumes (temporary storage).
    `--force` and `--grace-period=0` to forcibly evict pods.

2. Cordon Nodes

* Marks a node as "unschedulable" This prevents new pods from being scheduled onto that node.

* Existing pods remain unaffected.

* used to prevent new workloads from landing on a node that might be undergoing preparation for maintenance

* `kubectl cordon <node-name>`


3. Uncordon Node

* Allow schedualing to the node again, node will accept new pods.

* `kubectl uncordon <node-name>`


![drain-vs-cordon-node](./screenshots/drain-vs-cordon-node.png)


4. [Upgrade Cluster Instructions](./Instructions/k8s_cluster_upgrade.txt)

* Control Node :
    1. drain worker node
    2. upgrade kubeadm 
    3. Verify the upgrade plan
    4. Apply the Upgrade
    5. upgrade kubelet config
    6. restart kubelet
    7. uncordon the node

* Worker Node :
    1. drain worker node
    2. upgrade kubeadm 
    3. upgrade kubelet config
    4. restart kubelet
    5. uncordon the node





## Role Based Access Control (RBAC)

* [doc-link](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

* uses the `rbac.authorization.k8s.io` API group 

*  security mechanism that regulates access to Kubernetes API resources

* types of RBAC objects

    1. __Roles__ : define a set of permissions within a specific namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods", "pods/log"]   # for all resources we can use ["*"]
  verbs: ["get", "watch", "list"]   # ["*"]

```

2. __ClusterRoles__ : define the permissions accross the cluster

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]

```

3. __RoleBindings__ : associates a Role with a user, group, or service account within a specific namespace

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole, This kind of reference lets you define a set of common roles across your cluster, then reuse them within multiple namespaces.
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io

```

4. __ClusterRoleBinding__ : associates a ClusterRole with a user, group, or service account across the cluster

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole         
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io

```

* __`roleRef` is immutable__

    * After creating the binding, we cannot change the Role or ClusterRole that it refers to. 

    * To change the referenced role, you must delete the old binding and create a new one.

    * it prevents users from accidentally or maliciously changing a user's permissions.


### Aggregated ClusterRoles 

* merges permissions from multiple ClusterRoles into a single, larger role using labels

* avoid edit the main ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-monitoring: "true"
rules: [] # The control plane automatically fills in the rules

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: aggregate-cron-tabs-edit
  labels:
    # Add these permissions to the "admin" and "edit" default roles.
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
rules:
- apiGroups: ["stable.example.com"]
  resources: ["crontabs"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: aggregate-cron-tabs-view
  labels:
    # Add these permissions to the "view" default role.
    rbac.authorization.k8s.io/aggregate-to-view: "true"
rules:
- apiGroups: ["stable.example.com"]
  resources: ["crontabs"]
  verbs: ["get", "list", "watch"]
```

### ServiceAccount

* used by the container process to authenticate  with the K8S API 

* Kubernetes automatically assigns the ServiceAccount named default in default namespace.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false
```

