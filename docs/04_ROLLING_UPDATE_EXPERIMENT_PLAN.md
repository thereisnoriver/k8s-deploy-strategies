# Rolling-Update Failure and Recovery: Experiment Design

**Status:** Experimental outline / working notes, **not a completed or validated test report**.

**Adapted from:** [Rolling update test (/42)](https://tyge-ottesen-brahe.tistory.com/42), published August 2025.

Although the original title includes "test," the article mainly develops scenarios and preparation steps. It mentions partial early work (multiple replicas and an initial rollout without a readiness probe), but includes no preserved logs, request traces, or measured outcomes sufficient to substantiate test completion.

## Research question

How do `replicas`, `maxSurge`, `maxUnavailable`, and readiness configuration affect service continuity when a new version becomes unhealthy?

## Experiment matrix — proposed, not executed here

| Scenario | Manipulation | What to record | Expected issue to investigate |
| --- | --- | --- | --- |
| Baseline | v1 → v2 with stable endpoint | Pod transitions, response versions, HTTP errors | Whether connections continue during replacement |
| Replica/rollout settings | ≥4 replicas; vary `maxSurge` and `maxUnavailable` | Ready/available counts, update time | Capacity versus availability |
| No readiness probe | Let app start before serving reliably | Responses and Service endpoints | Premature traffic to unready app |
| Application exception | Introduce error in candidate endpoint | HTTP errors, Pod readiness, rollout state | Healthy container does not imply healthy user function |
| Process crash | Make candidate process exit | Restarts, events, availability | Controller replacement is not instant |
| Rollback | Revert to earlier revision | New/old responses, recovery time, rollout history | Whether the stable revision and compatible state remain available |

## Preparation checklist

- [ ] Specify Kubernetes distribution and exact version (Minikube, kubeadm VM cluster, or managed service).
- [ ] Confirm container listening port, image tags, Deployment labels, and Service selector.
- [ ] Build and make both v1 and v2 images accessible to the chosen runtime. A local Minikube image workflow is **not interchangeable** with publishing images to a GKE/EKS-accessible registry.
- [ ] Add a deliberately observable `/version` endpoint and, separately, meaningful readiness checks.
- [ ] Capture the **before** state: Pods, ReplicaSets, Deployment revisions, and manifest checksums.
- [ ] Collect timestamped request outcomes throughout rollout and recovery.
- [ ] Capture `kubectl rollout status`, `kubectl describe deployment`, events, and Pod logs.
- [ ] Report observed results and limitations; only then check off a scenario as completed.

## Example commands for a future session

```bash
kubectl get deploy,rs,pods -o wide
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>
kubectl describe deployment <deployment-name>
kubectl rollout undo deployment/<deployment-name>
```

Commands are illustrative. `rollout undo` requires an available revision; it does not revert database schemas or external side effects. Replace the placeholders with your deployment name.

## Evidence standard

A credible result should include image digests, manifest revisions, cluster context, exact injected fault, timestamped requests and error counts, and observed restoration times. **No such measured findings are currently claimed by this repository.**

## Further reading

- [Update a Deployment Without Downtime](https://kubernetes.io/docs/tasks/run-application/update-deployment-rolling/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Readiness and liveness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
