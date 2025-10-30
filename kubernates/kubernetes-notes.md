
# Kubernetes

* is an open source system for automating deployment, scaling, and management of containerized applications.

* Container Orchestration tool

* known as K8S ( 8 is meaning 8 letters bettwen k and s)

## K8S Cluster Components

![Cluster](./screenshots/kubernetes-cluster-architecture.svg)

1. Control/Master Node

* kube-apiserver

    * the front end for the Kubernetes control plane.

    * designed to scale horizontally

    * it exposes the Kubernetes API, allowing users, external tools, and other cluster components to communicate with the cluster, and manage its resources

    * handles authentication of API requests and performs authorization checks to ensure that users and service accounts have the necessary permissions to perform requested actions.

    * validates and configures data for API objects


* etcd 

    * key value database stores cluster information (configuration, state, metadata)

* kube-controller-manager

    * monitors the cluster’s state through the Kubernetes API server

    * types

        * Replication Controller : Ensures the desired number of Pod replicas are running

        * Namespace Controller: Creates and manages Kubernetes namespaces

        * Endpoints Controller: Maintains an Endpoints object for each Service

        * Service Account Controller: Creates and manages Service Accounts used for Pod authentication and authorization.

        * Node Controller: Tracks the health and availability of Nodes in the cluster.

        * Token Controller: Responsible for issuing authentication tokens for service accounts.

        * Lease Controller: Enforces leasing mechanisms for certain resources to prevent conflicts and maintain coordination.

* kube-scheduler

    * assign pods to suitable nodes based on various factors

* cloud-controller-manager

    *  link the cluster into cloud provider's API

    * types

        * Node Controller: responsible for checking if a particular node is present in the cloud or not and updating the node labels with appropriate labels and annotations

        * Route Controller: responsible for configuring routes in the cloud environment 

        * Service Controller: esponsible for creating a load balancer in the cloud environment.


2. Worker Node

* kubelet

    * agent that runs on each node in the cluster

    * makes sure that containers are running in a Pod.

    * doesn't manage containers which were not created by Kubernetes

    * acts as the bridge between the Kubernetes control plane and the node, and interacting with the container runtime to execute and manage containers

    * monitors the health of pods and their containers using `liveness and readiness probes`.

    * Manages resources on the node and can perform evictions if thresholds are met (terminate)

    * it sends gRPC requests through the CRI to the container runtime

* kube-proxy

    * network proxy that runs on each node in your cluster

    * maintains network rules on nodes

    * no need to kube-proxy, if network plugins used which forwared packets to services


* container runtime

    * responsible for running containers

    * it links to a third party project or product that is not part of Kubernetes itself

    * installed on each node in the cluster

    * k8s interacts with container runtimes through the `CRI` (container runtime interface : standard API. It acts as a "plugin" interface)

    * types

        * low-level : `runc`
        
        * high-level

            * containerd

            * OCR-I


> Low-Level Runtime (runc): This is the engine. It just does one job: it takes instructions and runs the car. It doesn't know about steering, gas, or where you're going.

> High-Level Runtime (containerd, CRI-O): This is the car's internal system (the driver's controls, electronics, and gas tank). It manages the engine (runc), pulls the fuel (images), and handles the steering and brakes (lifecycle, networking, and storage).

> Container Engine (Docker, Podman): This is the complete car experience. It includes the user-friendly CLI (the steering wheel and dashboard), the high-level runtime, and the low-level runtime, all bundled together into one product.


3. addons : use Kubernetes resources (DaemonSet, Deployment, etc) to implement cluster features

    * DNS

    * Web UI (Dashboard)

    * Network plugins



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



## Namespaces

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


### Working with namespaces.

1. `kubectl create namespace <namespace-name>` : creat new namespace.

2. `kubectl get namespaces` : show all namespaces in the cluster.

3. `kubectl get <resource-type> <resource-name> --namespace <or -n> <namespace-name>` 

    that show the resouce in the namespace ex: `kubectl get pods -n default`

4. `kubectl config set-context --current --namespace=<namespace-name>` : set namespace for the current context. When executing subsequent `kubectl` commands without explicitly specifying a namespace. we can verify it using `kubectl config view --minify | grep namespace:`


## Role Based Access Control (RBAC)

* [doc-link](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

* uses the `rbac.authorization.k8s.io` API group 

*  security mechanism that regulates access to Kubernetes API resources

### types of RBAC objects

#### Roles

* define a set of permissions within a specific namespace.

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

#### ClusterRoles 

* define the permissions accross the cluster

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

#### RoleBindings 

* associates a Role with a user, group, or service account within a specific namespace

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

#### ClusterRoleBinding

* associates a ClusterRole with a user, group, or service account across the cluster

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

![rbac](./screenshots/rbac.png)


## Application Configuration

* k8s allow pass dynamic configuration values at the application runtime which control its workflow


### ConfiMap

* used to assign non-sensitive configuration

* key value format

* data stored in a ConfigMap cannot exceed 1 MiB

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:

  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true

immutable: true  # set the configmap immutable
```


* ways to use ConfigMap in the pod containers

    * Inside a container command and args
    * Environment variables for a container
    * Add a file in read-only volume, for the application to read
    * Write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # Define the environment variable
        - name: PLAYER_INITIAL_LIVES # Notice that the case is different here
                                     # from the key name in the ConfigMap.
          valueFrom:
            configMapKeyRef:
              name: game-demo           # name of the  ConfigMap 
              key: player_initial_lives # The key to fetch.
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name

      # or use this 

      #envFrom:
      #  - configMapRef:
      #      name: myconfigmap    # name of the configmap


      volumeMounts:
      - name: config
        mountPath: "/config"  # inside the container
        readOnly: true
        # defaultMode: 0444 # Read-only permissions for owner, group, and others
  volumes:
  # You set volumes at the Pod level, then mount them into containers inside that Pod
  - name: config
    configMap:
      # Provide the name of the ConfigMap you want to mount.
      name: game-demo
      # An array of keys from the ConfigMap to create as files
      items:
      - key: "game.properties"
        path: "game.properties"
      - key: "user-interface.properties"
        path: "user-interface.properties"
        
```

### Secret

* used to configure the sensitive data

* key value format

* command to encrypt a text `echo -n "some-text" | base64 `

* [types of secrets](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)

  * Opaque  : default Secret type

    we must use `kubectl create secret generic <...>`



```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
type: Opaque

---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  volumes:
    - name: secret-volume
      secret:
        secretName: dotfile-secret
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      command:
        - ls
        - "-l"
        - "/etc/secret-volume"
      volumeMounts:
        - name: secret-volume
          readOnly: true
          mountPath: "/etc/secret-volume"
```
## Storage

### Volume

  * provide a way for containers in a pod to access and share data via the filesystem

  * important for data persistence and shared storage

* types

1. ConfigMap

  * inject configuration data into pods

  * data stored in a ConfigMap can be referenced using this volume type

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox:1.28
      command: ['sh', '-c', 'echo "The app is running!" && tail -f /dev/null']
      volumeMounts:
        - name: config-vol   # name of the volume
          mountPath: /etc/config  # path in the container
  volumes:
    - name: config-vol   # name of the volume
      configMap:          # type of the volume  always mounted as readOnly
        name: log-config   # name of the configMap
        items:              # may share some items from the configMap 
          - key: log_level
            path: log_level.conf
```

2. emptyDir

  * created when the Pod is assigned to a node

  * all containers in the Pod can read and write the same files in the emptyDir volume

  * can be mounted at the same or different paths in each container

  * if pod removed the volume removed but when container removed has no affect on the volume

  * by default are stored on whatever medium that backs the node such as disk, SSD, or network storage, depending on your environment
  * used for Sharing Data Between Containers, Temporary File Storage, High-Speed In-Memory Storage when assign `medium: "Memory"`

```yaml

spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
      medium: Memory   # memory storage

```

3. hostPath

  * mounts a file or directory from the host node's filesystem into the Pod

  * presents many security risks. insted it use `local persistence volume`

  * uses 

    * running a container that needs access to node-level system components such as reading logs 

    * making a configuration file stored on the host system available read-only to a static pod as it can not access configMap


```yaml
 volumeMounts:
    - mountPath: /foo
      name: example-volume
      readOnly: true        # restict the access that is for more security
  volumes:
  - name: example-volume
    # mount /data/foo, but only if that directory already exists
    hostPath:
      path: /data/foo # directory location on host
      type: Directory # this field is optional default is "" means no checks will be performed before mounting the hostPath volume
```

  * hostPath type

    1. `""` it is the default value which means no checks will be performed befor mounting the hostPath volume

    2. `DirectoryOrCreate` if the path not exist then it will be created with permissions `0755`

    3. `Directory` directory must exist at the given path

    4. `FileOrCreate` If nothing exists at the path, an empty file is created with permissions 0644`, If the parent directory of the mounted file does not exist, the pod fails to start

    5. `File` A file must already exist at the given path.

    6. `Socket` A UNIX socket must already exist at the given path.

    7. `CharDevice` A character device must exist at the path (Linux nodes only).

    8. `BlockDevice`  A block device must exist at the path (Linux nodes only).


4. other types

  * `gitRepo` mounts a git repo as a volume

  * `image` mounts an OCI object (a container image or artifact) that is available on the kubelet's host machine

  * `nfs` mounts an existing NFS (Network File System) share. Data is preserved, and it can be mounted by multiple writers simultaneously.

  * `local` used when creating `PresistenceVolume`, It requires nodeAffinity to be set on the PersistentVolume so the scheduler can place the Pod on the correct node. 

  * `persistentVolumeClaim`  Used to mount a `PersistentVolume` into a Pod, allowing users to claim durable storage without knowing the details of the particular cloud environment.

  * `secret` used to pass sensitive information, such as passwords, to Pods. mounted as `readOnly`

```yaml

volumes:
  - name: git-volume
    gitRepo:
      repository: "git@somewhere:me/my-git-repository.git"
      revision: "22f1d8406d464b0c0874075539c1f2e96c253775"
---

volumes:
  - name: volume
    image:
      reference: quay.io/crio/artifact:v2  # Artifact reference
      pullPolicy: IfNotPresent
---
volumes:
  - name: test-volume
    nfs:
      server: my-nfs-server.example.com
      path: /my-nfs-volume
      readOnly: true
```

### PersistenceVolume (PV)

  * provides details of how storage is provided from how it is consume

  * piece of storage in the cluster that has been `provisioned` by an `administrator` or `dynamically` provisioned using `Storage Classes`


#### PV provisioning

1. Static

  * cluster administrator creates a number of PVs

  * PVs are available in the API for users to claim.

2. Dynamic

  * using `StorageClass`

#### PV Binding

  * control loop in the control plane watches for new PVCs, finds a matching PV, and binds them together. if `PV` dynamicaly provisioned, loop always bind that PV to the PVC

#### Storage Object in use Protection

  * prevents data loss by stopping the deletion of PVCs or PVs that are actively in use.

  * protection is managed using `finalizers` (e.g., kubernetes.io/pvc-protection). ensures that PVs with a `Delete` policy are only removed after the backing storage has been successfully deleted.

  
#### Reclaiming

  * define the affect of PV after deleting the PVC

  1. `Retain` 

  * PV and its data remains after deleting PVC

  * Volume is `released` and require manual cleanup


  2. `Delete`

  * default for dynamic provisioning

  * deletes both the PV object and the associated storage asset in the external infrastructure.

  3. `Recycle (deprecated)`

  * performs a basic scrub (rm -rf /thevolume/*) on the volume. and makes it available again for a new claim.

#### PV Types

[Types of Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)

#### Access Modes

1. `ReadWriteOnce` volume can be mounted as read-write by a single node, allow multiple pods in the same node to access the volume at the same time

2. `ReadOnlyMany` volume can be mounted as read-only by many nodes.

3. `ReadWriteMany` volume can be mounted as read-write by many nodes.

4. `ReadWriteOncePod`  volume can be mounted as read-write by a single Pod


```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv-volume
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem  # by default is Filesystem, mounted into Pods into a directory 
                         # another mode is  `Block` volume is presented into a Pod as a block device without any filesystem on it
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  # The actual storage details vary based on the storage type (e.g., nfs, awsEbs, azureDisk)
  nfs:
    path: /mnt/nfs/data
    server: 192.168.1.100
```


#### PV Phase

1. `Available` : The volume is free and not currently bound to any claim.

2. `Bound` : The volume is actively bound to a PersistentVolumeClaim (PVC).

3. `Released` : The associated PVC has been deleted, but the cluster has not yet reclaimed the storage resource.

4. `Failed` : The volume's automated reclamation process has failed.


### PersistentVolumeClaim (PVC) 

  * it is a request for storage by a user.

  * Pods consume node resources and PVCs consume PV resources

  * `Pods` can request specific levels of resources (CPU and Memory). `Claims` can request specific `size` and `access modes`

  * Pods access storage by mounting the PVC as a volume. The Pod and the PVC must exist in the same namespace.

  * only expand a `PVC` if its `storage class`'s `allowVolumeExpansion` field is set to `true`.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow    # Bound with pv if has the same storage class name, PVCs don't necessarily have to request a class, set it ""  bound only to PVs of that default storageClass
  selector:       # volumes whose labels match the selector can be bound to the claim
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}

---

# PVC from volume snapshot
spec:
  storageClassName: csi-hostpath-sc
  dataSource:
    name: new-snapshot-test
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---
# Clone a PVC

spec:
  storageClassName: my-csi-plugin
  dataSource:
    name: existing-src-pvc-name
    kind: PersistentVolumeClaim
```

### Storage Class

* define the type of the storage service

* primary mechanism for `dynamic provisioning`, which automatically creates a PersistentVolume (PV) when a PersistentVolumeClaim (PVC) requests a specific class

* `provisioner` that determines what `volume plugin` is used for provisioning PVs

* `parameters` describe the storage (e.g., disk type, filesystem, or encryption settings)



```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: low-latency
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: csi-driver.example-vendor.example
reclaimPolicy: Retain # default value is Delete
allowVolumeExpansion: true
mountOptions:
  - discard # this might enable UNMAP / TRIM at the block storage layer
volumeBindingMode: WaitForFirstConsumer             # `Immediate` mode indicates that volume binding and dynamic provisioning occurs once the PersistentVolumeClaim is created
parameters:                                   # describe the storage (e.g., disk type, filesystem, or encryption settings)
  guaranteedReadWriteLatency: "true" # provider-specific

allowedTopologies:      # Used with `WaitForFirstConsumer` to restrict the provisioning of volumes to specific topologies (e.g., only in us-central-1a or us-central-1b).
- matchLabelExpressions:
  - key: topology.kubernetes.io/zone
    values:
    - us-central-1a
    - us-central-1b
```

#### Provisioner

* `Internal Provisioners`: Legacy in-tree provisioners (e.g., kubernetes.io/vsphere-volume). Many of these, including those for AWS EBS and Azure Disk, are deprecated.

* `External Provisioners`: The recommended approach. These are independent programs that follow the `Container Storage Interface (CSI)` specification (e.g., ebs.csi.aws.com, efs.csi.aws.com).

* `Local Storage`: A special case that uses `kubernetes.io/no-provisioner`. It doesn't dynamically provision new storage but uses `WaitForFirstConsumer` to delay binding until a Pod is scheduled, allowing the Pod to be matched with a pre-existing local PV on a specific node.



## Pods

* set of containers with shared namespaces and shared filesystem volumes.

### Containers Resources (cpu & memory)

* Resource Request

  * request amount of that system resource specifically for that container to use

  * `kube-scheduler` uses this information to decide which node to place the Pod

* Resource Limt

  * `kubelet` enforces those limits so that the running container is not allowed to use more of that resource than the limit you set

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
# we can assign resources for the pod and its container
  resources:
    limits:
      cpu: "1"
      memory: "200Mi"
    requests:
      cpu: "1"
      memory: "100Mi"

  containers:
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"   # m is millicpu or millicores
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Container HealthCheck

1. Liveness probes 

* determine when to restart a container

* if a container fails its liveness probe repeatedly, the kubelet restarts the container.

* do not wait for readiness probes to succeed, we can use startup probe in this case


2. Readiness probes

* determine when a container is ready to accept traffic, end-end healthcheck lifecycle

* used when the app establishing network connections, loading files, and warming caches

* run on the container during its whole lifecycle

3. startup probe

* verifies whether the application within a container is started.

* if configured, it disables liveness and readiness checks until it succeeds.


![healthcheck](./screenshots/healthcheck.png)


> we can define the probes using

  * command in the container 

  * HTTP request

  * [TCP socket](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-tcp-liveness-probe)

  * [gRPC](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-grpc-liveness-probe)


4. _examples_

* 
```yaml
#....

spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox:1.27.2
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5   # tells kubelet that it should wait 5 seconds before performing the first probe
      periodSeconds: 5         # kubelet should perform a liveness probe every 5 seconds

---

#....

spec:
  containers:
  - name: liveness
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - liveness
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3

---
#....
ports:
- name: liveness-port
  containerPort: 8080

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30    # the application will have a maximum of 5 minutes (30 * 10 = 300s) to finish its startup, If the startup probe never succeeds, the container is killed after 300s, then restart the pod.
  periodSeconds: 10

---

readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```


* configure the probes doc

> [doc](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#configure-probes)



### Restart Policies

* k8s has the abaility to restart the container when it fails


1. Always

  * default policy

  * even if run successfully

2. OnFailuer

  * only when container process exit with error code

  * when liveness probe detect that the container is unhealthy

3. Never

  * conatiner never restart even liveness probe detect that the container unhealthy 


```yaml

#...
spec:
  restartPolicy: OnFailure   # restart policy for the pod

  containers:

  - name: try-once-container    # This container will run only once because the restartPolicy is Never.
    image: docker.io/library/busybox:1.28
    command: ['sh', '-c', 'echo "Only running once" && sleep 10 && exit 1']
    restartPolicy: Never     # restart policy for the container

  - name: on-failure-container  # This container will be restarted on failure.
    image: docker.io/library/busybox:1.28
    command: ['sh', '-c', 'echo "Keep restarting" && sleep 1800 && exit 1']
```


### SideCar Container

* are the secondary containers that run along with the main application container within the same Pod

* remain running after Pod startup

* have their own independent lifecycles

* support probes to control their lifecycle.

* Shares the same network and storage as the main app.

*  used to enhance or to extend the functionality of the primary app container by providing additional services, or functionality such as logging, monitoring, security, or data synchronization, without directly altering the primary application code.


```yaml
spec:
      containers:
        - name: myapp
          image: alpine:latest
          command: ['sh', '-c', 'while true; do echo "logging" >> /opt/logs.txt; sleep 1; done']
          volumeMounts:
            - name: data
              mountPath: /opt
      initContainers:
        - name: logshipper
          image: alpine:latest
          restartPolicy: Always
          command: ['sh', '-c', 'tail -F /opt/logs.txt']
          volumeMounts:
            - name: data
              mountPath: /opt
      volumes:
        - name: data
          emptyDir: {}
```


### init Container

* run before the app containers are started.

* always run to completion

* Pod cannot be Ready until all init containers have succeeded

* Shares the same network and storage as the main app.

* Do not support lifecycle, livenessProbe, readinessProbe, or startupProbe.


```yaml
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

### Scheduling Pods



1. using `nodeSelector`and `nodeName`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:       #best practice
    disktype: ssd

  #nodeName: foo-node

```

>  * add label to node   

  `kubectl label nodes <your-node-name> disktype=ssd`

  `kubectl get nodes --show-labels`

2. using Affinity and anti-affinity

* Node affinity functions like the nodeSelector field but is more expressive and allows you to specify `soft rules`

  * two types of node affinity:

    * `requiredDuringSchedulingIgnoredDuringExecution` : Pod won't be scheduled unless the rule is met

    * `preferredDuringSchedulingIgnoredDuringExecution` : scheduler tries to meet the rule but will schedule the Pod elsewhere if no matching Node is found. Preferred rules can be given a `weight (1-100)`


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:           # required during scheduling pod must pass the conditions
                                                                #  ignore during execution pod still run if label change 
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:        # preferred during scheduling the pod wil be scheduled if rules not satisfied
                                                                #  ignore during execution pod still run if label change  
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:3.8
```

* Node anti-Affinity

  * pods run on a Node that DOES NOT HAVE this label

  * only change the operatore with add `NotIN` or `DoesNotExist` also `Gt` and `Lt` 
```yaml
affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: NotIn
            values:
            - antarctica-east1
```

> Operatores

| Operator | Behavior |
|---|---|
| In | The label value is present in the supplied set of strings |
|NotIn|The label value is not contained in the supplied set of strings|
|Exists|A label with this key exists on the object|
|DoesNotExist|No label with this key exists on the object|
|Gt|The field value will be parsed as an integer, and that integer is less than the integer that results from parsing the value of a label named by this selector|
|Lt|The field value will be parsed as an integer, and that integer is greater than the integer that results from parsing the value of a label named by this selector|


## DaemonSets

![daemonset](./screenshots/daemonset.png)

* Run a Copy of the Pod on each cluster node

* best pratctice to use in `monitoring, log collection, proxy configuration`

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v5.0.1
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      # it may be desirable to set a high priority class to ensure that a DaemonSet Pod
      # preempts running Pods
      # priorityClassName: important
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

### Static pod

* managed by `kubelet`, API server not required

* `kubelet` automatically tries to create a `mirror Pod` on the Kubernetes API server for each static Pod

* `mirror pod` it is replica of staic pod allow API monitore the static pod

* to create it 

    go to the node where you want to runt the pod

    `/etc/kubernetes/manifests`

    create the pod yaml file

    then reload the `kubelet`  : `systemctl restart kubelet`

    `crictl ps` used to view the running containers

    `crictl stop <container-id>`  stop the running container


```bash
# Run this command on the node where kubelet is running
mkdir -p /etc/kubernetes/manifests/
cat <<EOF >/etc/kubernetes/manifests/static-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
EOF
```

## App Scaling

* k8s support automatic scaling

1. Vertical Scaling : scaling up

    * adding more resources (memory,cpu)

2. Horizontal Scaling : scaling out

    * adding more pods,nodes


## Stateless ans Statful

![stateless&stateful](./screenshots/stateless&stateful.png)

* Stateless app does not save the client data , scalled horizonatly, pods can be replaced or scaled up/down without affecting the application's functionality. managed by k8s `Deployments`

* Stateful app stores the client data, scaled verticaly only because we cant split the file system such as database apps, managed by k8s `StatefulSet`

* StatefulSet requires a Headless Service to manage network identities assign `clusterIP: None` in `service` Makes it a Headless Service


```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx" # Name of the Headless Service
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:   # Each pod gets its own PVC based on this template
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```


## ReplicationControlller and ReplicaSet

![replica](./screenshots/replicaset.png)

* ReplicationController : Ensures a specified number of pod replicas are running.

* ReplicaSet : next-generation ReplicationController

* "bare pod" or "standalone pod" refers to a Pod that is created directly without being managed by a higher-level controller like a Deployment, ReplicaSet, Job, or DaemonSet.

```yaml

apiVersion: v1
kind: ReplicationController
metadata:
  name: my-replication-controller
spec:
  replicas: 3
  selector:
    app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: my-image
---

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchExpressions:               # use enchanced selector than rc
      - {key: app, operator: In, values: [my-app]}  # NotIn, values....
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: my-image
```

## Deployment

![deployment](./screenshots/deployment.png)

* Deployment provides declarative updates for Pods and ReplicaSets.

* define the desired state in the Deployment specification, and the Deployment Controller automatically changes the actual state to match the desired state at a controlled rate

* uses

    * Rollout a new ReplicaSet to create Pods.

    * Declare a new state (e.g., updating the Pod image) to trigger a gradual scale-up of a new ReplicaSet and scale-down of the old ReplicaSet.

    * Rollback to a previous stable revision.

    * Scale up or down to manage load.

    * Pause and resume rollouts to batch multiple updates.


```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx    # Deployment's label selector is immutable after it gets created.
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

* work with deployment   

    * `kubectl set image deployment.v1.apps/nginx-deployment <container-name=new-image>` 

    * `kubectl set image deployment/nginx-deployment <container-name=image-name>`

    * `kubectl edit deployment/nginx-deployment`

    * `kubectl rollout status deployment/nginx-deployment`  : to see the satatus

    * `kubectl rollout history deployment/nginx-deployment`  : check the history

    * `kubectl rollout history deployment/nginx-deployment --revision=2` : see the previous revision number 2

    * `kubectl rollout undo deployment/nginx-deployment`  : rollback to previous revision

    * `kubectl rollout undo deployment/nginx-deployment --to-revision=2`  : rollback to previous revision

    * `kubectl scale deployment/nginx-deployment --replicas=10` scale the deployment

    * `kubectl autoscale deployment/nginx-deployment --min=10 --max=15 --cpu-percent=80`  scale the deployment if `HPA` is enabled

    * `kubectl rollout pause deployment/nginx-deployment`  pause the update

    * `kubectl rollout resume deployment/nginx-deployment`  resume the update

    * `kubectl set resources deployment/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi`  update the resources


* updating the deployment process

    * create new replicaset
    * scale up the new replicaset
    * scale down the old replicaset

* failed dwployment 

    * insufficient quota
    * Readiness probe failures
    * Image pull errors
    * Insufficient permissions
    * Limit ranges
    * Application runtime misconfiguration

### Deployment Strategy

![deployment-strategy](./screenshots/deployment-strategy.png)

1. Recreate Deployment

    * down all pods and recreate a new version of them again

    * good at testing and development environment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-recreate
spec:
  replicas: 3
  strategy:
    type: Recreate # strategy type
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-app:v2 # Change to :v3 for the update
```

2. Rolling Update

    * default strategy

    * update pod one by one

    * low downtime, easy to roll back to the previous version.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-rolling
spec:
  replicas: 4
  strategy:
    type: RollingUpdate # This is the default, can be omitted
    rollingUpdate:
      maxUnavailable: 25% # Max 1 pod unavailable at any time (1/4 of 4 replicas) , can be number or percentage
      maxSurge: 1 # Max 1 extra pod allowed during the update (total pods: 4+1=5), Pods that can be created over the desired number of replicas
  template:
    # ... Pod template definition with image: my-app:v2 ...
```

3. Blue/Green 

    * two environments side-by-side 

    * blue it is current version

    * green it is the new version

    * cons : Double the resource consumption

    * used when new version may have a risk and rapid rollback is required 

```yaml
# Deployment 'blue-app-v1'
spec:
  selector:
    matchLabels:
      app: my-app
      version: v1 # Blue version label
# ... container image: my-app:v1

---

# Deployment 'green-app-v2'
spec:
  selector:
    matchLabels:
      app: my-app
      version: v2 # Green version label
# ... container image: my-app:v2

---
# to change the traffic from blue to green change the version
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: v1 # <--- Currently routes to Blue (v1)
  ports:
  - port: 80
# ...
```

4. Canary

    * partial update process

    * allow test the new version on real users without commitment the full rollout

    * Highly risk-mitigating as issues only affect a small user group

5. A/B Testing

    * Multiple versions are concurrently tested on different users to compare performance or user experience.



## K8s Networking

* The network plugin is configured to assign IP addresses to Pods.
* The kube-apiserver is configured to assign IP addresses to Services.
* The kubelet or the cloud-controller-manager is configured to assign IP addresses to Nodes.

### Container Network Interface (CNI) 

* provide network connectivity to Kubernetes Pods, enabling communication both within a node and across nodes in the cluster.

* When a Pod is created, the CNI plugin creates and configures virtual network interfaces within the Pod's network namespace

* CNI plugins assign unique IP addresses to Pods from a configured IP address pool, ensuring that each Pod has a routable address.

* configure routing rules, firewall rules, and other network settings to facilitate communication between Pods, and between Pods and external services.

> [__plugins examples__](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy)

* Flannel: A simple and lightweight CNI plugin often used for basic overlay networking.

* Calico: Offers advanced network policy enforcement and supports various networking modes.

* Cilium: Leverages eBPF for high-performance networking, security, and observability.

* Weave Net: Provides a simple and resilient overlay network for containers.


### DNS

* k8s creates DNS records for Services and Pods

* DNS queries may be expanded using the Pod's `/etc/resolv.conf`. kubelet configures this file for each Pod

### DNS Records

1. Services

* Normal Services (with a Cluster IP)

    * `A` record (or `AAAA` for IPv6) that points to the Service's stable `Cluster IP`.

    * The Name Format: `my-svc.my-namespace.svc.cluster-domain.example` such as `web-app.production.svc.cluster.local`


* Headless Services (without a Cluster IP)

    * `A` record (or `AAAA`) that resolves to the IP addresses of `ALL the Pods` that the Service selects

    * name format : `my-svc.my-namespace.svc.cluster-domain.example`  resolve to the IP of every Pod running the application

* SRV Records (Service Records)

    * provide information about the port and protocol of a named service port.

    * format : `_port-name._port-protocol.my-svc.my-namespace.svc.cluster-domain.example`

2. Pods

* Default Pod Naming (Older Format)

    * `<pod-IPv4-address-with-dashes>.<namespace>.pod.<cluster-domain>`

* Custom Hostname and Subdomain (For specific Naming)

    * give it a special subdomain to create a specific, resolvable Fully Qualified Domain Name (FQDN). commonly used for stateful applications

    * `spec.hostname` 

    * `spec.subdomain` work with Headless Service

    * `<hostname>.<subdomain>.<namespace>.svc.cluster-domain.example`

    * `setHostnameAsFQDN: true` : maked the full FQDN (the long name) be returned as the hostname inside the Pod, instead of just the short hostname.


#### Pod DNS policy  (`dnsPolicy`)

* `ClusterFirst` (Default)	Tries to resolve names inside the cluster first (like web-app.production), then forwards external queries (like www.google.com) to the node's upstream DNS.	Used by default if no policy is set.

* `Default`	The Pod inherits the DNS settings directly from the node it's running on.	When you need the Pod to use the host's DNS setup.

* `ClusterFirstWithHostNet`	Same as `ClusterFirst`, but used specifically for Pods with `hostNetwork: true`.

* `None` Completely `ignores` the `Kubernetes-generated DNS settings`. Requires you to `manually specify all settings` using `dnsConfig`. When you need complete, custom control over the Pod's DNS.

#### DNS Config (`dnsConfig`)

* allows fine-grained customization of DNS settings, especially when using `dnsPolicy: "None"`.

    * `nameservers`: A list of specific IP addresses for the DNS servers to use (max 3).

    * `searches`: A list of search domains to try when resolving a short name.

        * Example: If you query for `web-app` and have `production.svc.cluster.local` in your searches, the DNS will try to resolve `web-app.production.svc.cluster.local`.

    * `options`: Special options for the resolver, like ndots (how many dots a name must have before it's considered fully qualified).

        * Example: Setting `ndots: 2` means a `name with 2` or `more dots` (like `foo.bar`) is tried as a `FQDN` before using search paths.


### Network Policy

* k8s object

* control traffic flow at the IP address or port level (OSI layer 3 or 4)

* cluster must use a network plugin that supports NetworkPolicy enforcement. such as `calico`

* entities that a Pod can communicate with three identifiers

    * other pods that are allowed (exception: a pod cannot block access to itself)

    * namespaces that are allowed

    * ip blocks (exception: traffic to and from the node where a Pod is running is always allowed)

* ingress : incoming traffic to the pod

* egress : outcoming traffic from the pod, By default, a pod is `non-isolated` for egress


```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:   # select the pods based on the labels
    matchLabels:
      role: db
  policyTypes:    # ingress and egress
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24  # deny
    - namespaceSelector:            # selects particular namespaces for which all Pods should be allowed
        matchLabels:
          project: myproject
    - podSelector:                  # selects particular Pods in the same namespace as the NetworkPolicy
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 32000
      endPort: 32768    # target range of ports 
```

> `note`

```yaml

ingress:
  - from:
    - namespaceSelector:                    # This policy contains a single from element allowing connections from Pods with the label role=client in namespaces with the label user=alice. But the following policy is different:
        matchLabels:                      
          user: alice
      podSelector:
        matchLabels:
          role: client

---

ingress:                                          # It contains two elements in the from array, and allows connections from Pods in the local Namespace with the label role=client, or from any Pod in any namespace with the label user=alice.
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client
---

spec:
  podSelector: {}           # deny ingress traffic  same for egress
  policyTypes:
  - Ingress

---
spec:
  podSelector: {}           # allow all ingress traffic     same for egress
  ingress:
  - {}
  policyTypes:
  - Ingress
```

### Service

*  method for exposing a network application that is running as one or more Pods in your cluster.

* Pods are ephemeral resources 

* Pod gets its own IP address

* Pods targeted by a Service is usually determined by a `selector` 

* default protocol for Services is TCP

* service without selector we can use `EndpointSlice` which represents a subset (a slice) of the backing network endpoints for a Service.

```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: my-service-1 # by convention, use the name of the Service
                     # as a prefix for the name of the EndpointSlice
  labels:
    # You should set the "kubernetes.io/service-name" label.
    # Set its value to match the name of the Service
    kubernetes.io/service-name: my-service
addressType: IPv4
ports:
  - name: http # should match with the name of the service port defined above
    appProtocol: http
    protocol: TCP
    port: 9376
endpoints:
  - addresses:
      - "10.4.5.6"
  - addresses:
      - "10.1.2.3"
```

#### Service Types

1. ClusterIp

* Exposes the Service on a cluster-internal IP, Service only can be reachable within the cluster

* default type

2. NodePort

* Exposes the Service on each Node's IP at a static port `30000 - 32767`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort  
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
  #externalIPs:
   # - 198.51.100.32
```

3. LoadBalancer

* Exposes the Service externally using an external load balancer. k8s must integrated with cloud provider

```yaml

spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
```

4. ExternalName

* Maps the Service to the contents of the `externalName` field

```yaml
spec:
  type: ExternalName
  externalName: my.database.example.com
```

> headless Services, a cluster IP is not allocated, kube-proxy does not handle these Services, and there is no load balancing or proxying done by the platform for them. allows a client to connect to whichever Pod it prefers


### Ingress

* manages external access to the services in a cluster

* provide load balancing, SSL termination and name-based virtual hosting.

* must have an [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) to satisfy an Ingress

*  uses `annotations` to configure some options depending on the Ingress controller


#### ingress rules

  contains

  * `optional host`  (if no host applied the rules will be applied to all inbound traffic)
  
  * `paths`  such as (/testpath) , host and path must match the incoming request befor loadbalancer direct the traffic
  
  * `backend` : contains the `service`
  
> A `defaultBackend` is often configured in an Ingress controller to service any requests that do not match a path in the spec, If no `.spec.rules` are specified, `.spec.defaultBackend` must be specified
  
> Resource backend is an ObjectRef to another Kubernetes resource within the same namespace as the Ingress object. A Resource is a mutually exclusive setting with Service, and will fail validation if both are specified. A common usage for a Resource backend is to ingress data to an object storage backend with static assets.


```yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-backend
spec:
  defaultBackend:
    resource:
      apiGroup: k8s.example.com
      kind: StorageBucket
      name: static-assets
  rules:
    - http:
        paths:
          - path: /icons
            pathType: ImplementationSpecific
            backend:
              resource:
                apiGroup: k8s.example.com
                kind: StorageBucket
                name: icon-assets
```
  
  
#### Path Types

* decide which backend service a request should be routed to


1. `ImplementationSpecific` 
  
  * matches is entirely up to the Ingress controller
  
  * could behave like Exact, Prefix

2. `Exact` : request path must exactly match the path defined in the Ingress rule, including being case-sensitive.

3. `Prefix` 

  * Matches based on a path element by element prefix.
  
  *  It treats the URL path as segments separated by `/`.
  
  * A match occurs if every segment in the Ingress path is an exact match for the corresponding segment in the request path

  * / matches all paths (it's the prefix of everything)


> multiple matches : precedence will be given `first` to the `longest matching path`. If two paths are still `equally matched`, precedence will be given to paths with an `exact path` type `over prefix path` type.
  
  
```yaml

spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: service1
            port:
              number: 80
```

#### Ingress Types

1. backed by a single Service 
  
  *  expose a single Service, inested it we can use service type NodePort or LoadBalancer
  
  
2. Simple fanout

  * routes traffic from a single IP address to more than one Service
  
![simple-fanout](./screenshots/ingressFanOut.svg)  

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-fanout-example
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 4200
      - path: /bar
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 8080
```

3. Name based virtual hosting

* routing HTTP traffic to multiple host names at the same IP address.

![ingressNameBased](./screenshots/ingressNameBased.svg)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: bar.foo.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service2
            port:
              number: 80
```

> TLS
  
  * keeps the connection between a user's web browser and your application private and secure (encrypted).
  
  * Ingress resource only supports a single TLS port, 443
  
  * `secrets` must contain `key` and `certificate`
  
  * SNI	Server Name Indication allows a single Ingress gateway to securely handle multiple different domain names through same port
  
```yaml
spec:
  tls:
  - hosts:                # 1. Host to secure
      - https-example.foo.com
    secretName: testsecret-tls # 2. Name of the Secret holding the cert/key
  rules:
---
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```
### Ingress Class

* Ingress Class defines which `Ingress controller` (e.g., NGINX controller, GCE controller) should `handle a specific Ingress resource`, and `provides a way to supply additional configuration to that controller`.

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: external-lb
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true" # set it default `IngressClass`
spec:
  controller: example.com/ingress-controller  # k8s.io/ingress-nginx
  parameters:
    apiGroup: k8s.example.com
    kind: IngressParameters
    name: external-lb
```



