# Container Orchestration and Kubernetes Architecture

**Source type:** Conceptual study notes. These are explanations, not records of an implemented cluster.

**Adapted from the author's Korean posts:**
- [Why Kubernetes? (/35)](https://tyge-ottesen-brahe.tistory.com/35)
- [Kubernetes concepts (/17)](https://tyge-ottesen-brahe.tistory.com/17)

## 1. Why container orchestration?

A virtual machine supplies its own guest operating system and kernel. A typical Linux container isolates processes using Linux primitives such as namespaces and cgroups while sharing the host kernel. Container images package the application's userspace dependencies, but compatibility still depends on operating-system, CPU architecture, kernel, and runtime constraints.

Containers make packaging and starting applications easier; they do not by themselves handle scheduling across machines, replacing failed instances, service discovery, or controlled multi-replica updates. Kubernetes adds APIs and controllers that continually work toward a declared desired state.

## 2. Images, containers, and runtimes

- **Image:** A read-only packaged artifact, typically built in layers, that describes an application and its dependencies.
- **Container:** A runtime instance created from an image, with an isolated process environment and writable layer.
- **Container runtime:** The software that starts and manages containers on a node. Kubernetes talks to compatible runtimes through the Container Runtime Interface (CRI), commonly using containerd or CRI-O.
- **Docker distinction:** Kubernetes removed its built-in *dockershim* integration in v1.24, **not support for Docker-built / OCI-compatible container images**.

An image does not guarantee identical behavior across different host kernels or CPU architectures.

## 3. Cluster components

| Component | Main responsibility |
| --- | --- |
| API server | Exposes the Kubernetes API and handles requests to the control plane |
| etcd | Stores Kubernetes cluster state |
| Scheduler | Assigns unscheduled Pods to suitable nodes |
| Controller manager | Runs reconciliation loops (including workload controllers) |
| kubelet | Agent on each node that works to keep assigned Pods running |
| Container runtime | Launches and manages containers on the node |
| Networking / CNI | Provides Pod networking, with behavior depending on the selected plugin |

A Deployment controller reconciles ReplicaSets; ReplicaSets reconcile Pod replicas. Controllers interact with Kubernetes objects through the API, not by directly operating containers on remote machines.

## 4. From desired state to a running workload

1. A user submits a Deployment object to the API server.
2. Controllers notice the discrepancy between desired and existing replicas.
3. Pod objects are created; the scheduler assigns unscheduled Pods to nodes.
4. Each node's kubelet asks its container runtime to start containers.
5. Readiness signals and Service endpoint membership help determine when Pods should receive traffic.

A Pod in `Running` phase is **not necessarily ready**, healthy from a user's perspective, or guaranteed to stay available. Deployments provide mechanisms for reconciliation; service-level availability also depends on design and configuration.

## 5. Services and networking

Pod IPs can change as Pods are recreated. A Service supplies a stable logical access point and selects matching Pods using labels; it typically sends traffic to *ready* endpoints. A Service is not, by itself, proof of a particular traffic percentage reaching a given release.

## Technical clarifications

The original posts were learning notes and use some simplifying language. This English edition distinguishes Docker images from dockershim, separates Pod phase from readiness, and describes controllers in terms of reconciliation rather than a direct one-step imperative sequence.

## Further reading

- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Dockershim Removal FAQ](https://kubernetes.io/blog/2022/02/17/dockershim-faq/)
