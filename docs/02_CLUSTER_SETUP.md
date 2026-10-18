# Google Cloud Kubernetes Practice Notes

**Source type:** Practice-oriented walkthroughs recorded in 2025. The articles describe commands and expected checks, but this repository does not contain a full terminal transcript, cluster snapshot, or repeatable test output. **Do not treat the steps below as an up-to-date, copy-paste installation guide.**

**Adapted from:**
- [Google Cloud Kubernetes practice, part 1 (/15)](https://tyge-ottesen-brahe.tistory.com/15)
- [Google Cloud Kubernetes practice, part 2 (/16)](https://tyge-ottesen-brahe.tistory.com/16)
- [Google Cloud Kubernetes practice, part 3 (/18)](https://tyge-ottesen-brahe.tistory.com/18)

## 1. Infrastructure outline

The first article organizes a small learning cluster around one `k8s-master` VM and two worker VMs using Google Cloud Shell and Compute Engine with Ubuntu. It covers VM creation, SSH access, hostname/IP discovery, name resolution through `/etc/hosts`, and basic connectivity checks.

This is a **self-managed kubeadm cluster on cloud VMs**, not a claim that Google Kubernetes Engine (GKE) was provisioned.

## 2. Node preparation and cluster bootstrap

The notes walk through these ideas:

1. Prepare each Linux node and disable swap where required by the chosen kubelet configuration.
2. Install and configure a compatible container runtime, such as containerd.
3. Install matching `kubeadm`, `kubelet`, and `kubectl` components using distribution-appropriate package sources.
4. Initialize the control plane with `kubeadm init`.
5. Configure `kubectl` access and install a compatible Container Network Interface (CNI) plugin, with Flannel and Calico discussed as alternatives.
6. Join the workers to the control plane with a **newly generated** `kubeadm join` command.
7. Inspect the result with `kubectl get nodes` and check that expected nodes are `Ready`.

Any bootstrap tokens or certificate hashes copied in old tutorial text are historical examples and must never be reused. Package repositories, versions, networking CIDRs, firewall rules, and runtime settings need to be checked against the current Kubernetes documentation.

### What would count as verification?

A reproducible practice report would retain the Kubernetes and OS versions, sanitized init/join commands, `kubectl get nodes -o wide` output, CNI state, relevant events, and a description of network access restrictions. Those artifacts are **not present in this GitHub repository**.

## 3. Workload objects explored

The third article moves from cluster operations into workloads:

- **Pod:** The smallest schedulable group of one or more containers.
- **Deployment and ReplicaSet:** Reconcile the desired number of application Pods and manage template changes.
- **Scaling:** `kubectl scale deployment/<name> --replicas=3` changes the desired replica count.
- **Resource requests/limits:** Influence scheduling and runtime constraints; they do not guarantee performance.
- **Maintenance:** `kubectl cordon` prevents new scheduling, while `kubectl drain` attempts to evict workloads safely under applicable policies.
- **Updates:** Replacing Pod templates through Deployments can support rolling updates, provided health checks and workload assumptions are suitable.

### Illustrative inspection commands

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get deployments
kubectl describe deployment <deployment-name>
kubectl rollout status deployment/<deployment-name>
kubectl get events -A --sort-by=.lastTimestamp
```

The angle-bracket placeholders above must be replaced; these are **reference commands**, not a captured execution log.

## 4. What this study did not establish

The blog contains operational steps and some narrative progress, but it does not preserve enough evidence here to conclude that a production-ready cluster, resilience guarantee, zero-downtime update, or automated rollback was successfully tested.

## References

- [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Safely Drain a Node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
