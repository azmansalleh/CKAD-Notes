# CKAD Notes

## Exam Version
Version 1.21

## Exam Details
- [Technical Instructions](https://www.cncf.io/certification/ckad/)
- [Exam Cluster Environment](https://www.cncf.io/certification/ckad/)

## Current Progress
The list below is based on the curriculum v1.0. Once you have mastered a section, check it off and move on to the next. You need to understand them ALL very well. The Core Concepts piece is kind of vague, but the others are defined well enough that it is easy to prepare for with a hands-on work through the tasks offered at kubernetes.io. The rest of this document follows this same outline of curriculum.

- [ ] __Core Concepts - 13%__
  - [ ] API Primitives
  - [ ] Create and Configure Basic Pods
- [ ] __Configuration - 18%__
  - [ ] Understand ConfigMaps
  - [ ] Understand SecurityContexts
  - [ ] Define App Resource Requirements
  - [ ] Create and Consume Secrets
  - [ ] Understand Service Accounts
- [ ] __Multi-Container Pods - 10%__
  - [ ] Design Patterns: Ambassador, Adapter, Sidecar
    - [ ] - Sidecar Pattern
    - [ ] - Init Containers
- [ ] __Pod Design - 20%__
  - [ ] Using Labels, Selectors, and Annotations
  - [ ] Understand Deployments and Rolling Updates
  - [ ] Understand Deployment Rollbacks
  - [ ] Understand Jobs and CronJobs
- [ ] - __State Persistence - 8%__
  - [ ] - Understand PVCs for Storage
- [ ] __Observability - 18%__
  - [ ] Liveness and Readiness Probes
  - [ ] Understand Container Logging
  - [ ] Understand Monitoring Application in Kubernetes
  - [ ] Understand Debugging in Kubernetes
- [ ] __Services and Networking - 13%__
  - [ ] Understand Services
  - [ ] Basic Network Policies

## 1. Core Concepts

The core concepts section covers the core K8s API and its primitives and resources. It also covers the important concept of a POD. This is the basic unit of deployment for app developers and so this 'POD' concept is important to understand as well as how they are managed with kubectl. To me, this is embodied in the kubectl RUN command.

### 1.1. Pods

A Pod (as in a pod of whales or pea pod) is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. 

|     Kind      |    Version    |
| ------------- | ------------- |
| Pod           | v1            |
| Service       | v1            |
| ReplicaSet    | apps/v1       |
| Deployment    | apps/v1       |

#### 1.1.1 Pod Definition

A sample `.yaml` file of pod definition file. The fields required are apiVersion, kind, metadata and spec. Refer to [pod_definition.yaml]("1.%20Core-Concepts/pod_definition.yaml")

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
    - name: nginx-container
      image: nginx
```

#### 1.1.2 Pod Information and Commands

Below are simple commands to retrieve pod information. We can either retrieve all of the pod information or a specific detailed pod information

```
$ kubectl get pods

NAME                            READY   STATUS    RESTARTS   AGE
myapp-pod-798bbc95c7-62cc7      1/1     Running   0          15d
mydb-pod-987478c7-2lpzq         1/1     Running   0          5d1h
```
```
$ kubectl describe pod myapp-pod

Name:         myapp-pod
Namespace:    default
Priority:     0
Node:         ip-192-168-123-123.ap-southeast-1.compute.internal/192.168.123.123
Start Time:   Mon, 01 Jun 2021 23:59:59 +0800

.....
```

If you are not given a pod definition file, you may extract the definition to a file using the below command:
```
$ kubectl get pod myapp-pod -o yaml > pod-definition-sample.yaml
```

If you wish to edit the pod directly to change the configuration, you may use the following command
```
kubectl edit pod <pod-name>
```

### 1.2. ReplicaSet

A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

#### 1.2.1  ReplicaController Definition

A sample `.yaml` file of a replicaSet definition file. The fields required are apiVersion, kind, metadata, spec and replicas Refer to [replicacontroller_definition.yaml]("1.%20Core-Concepts/replicacontroller_definition.yaml")

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: frontend
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: frontend
      spec:
        containers:
        - name: nginx-container
          image: nginx
  replicas: 3

```

Basic Commands to retrieve information on replica controller
```
$ kubectl create -f rc-definition.yaml
$ kubectl get replicacontroller
```

#### 1.2.2  ReplicaSet Definition

A sample `.yaml` file of a replicaSet definition file. The fields required are apiVersion, kind, metadata, spec, selector and replicas. Refer to [replicaset_definition.yaml]("1.%20Core-Concepts/replicaset_definition.yaml")

```
apiVersion: v1
kind: ReplicationSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: frontend
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: frontend
      spec:
        containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector: 
    matchLabels:
      type: frontend
```
<br>
<p align="center">
  <img src="images/replicaset.png" width="50%">
</p>
<br>

Basic Commands to retrieve information on replica controller
```
$ kubectl create -f replicaset-definition.yaml
$ kubectl delete replicaset myapp-replicaset
$ kubectl get replicaset
```

We can also update the amount of replicas by scaling in or out with the following command
```
$ kubectl replace -f replicaset-definition.yaml

OR

$ kubectl scale --replicas=6 -f replicaset myapp-replicaset
```