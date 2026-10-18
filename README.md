# Kubernetes Deployment Strategies — Study Notes

**Status: conceptual study and unfinished example scaffolding — not an executed deployment experiment.**

This repository collects notes on deployment and rollout strategies in Kubernetes. It was created to organize the ideas behind **rolling updates, blue-green deployment, and canary releases** and to plan possible failure and recovery experiments.

There are some illustrative YAML manifests and a small Spring Boot `/version` endpoint, but **no verified Kubernetes cluster runs, rollout logs, fault-injection results, or measured performance data** are included. The original 14-day exercise schedule is preserved in [docs/ORIGINAL_STUDY_PLAN.md](docs/ORIGINAL_STUDY_PLAN.md) as a **plan, not a completion log**.

## Strategy comparison

| Strategy | Concept | Main trade-off | Validation question |
| --- | --- | --- | --- |
| **Rolling update** | Gradually replace old Pods with new Pods in a Deployment. | Trades rollout speed, temporary capacity, and availability; behavior depends on readiness and `maxSurge`/`maxUnavailable`. | When is an updated Pod actually ready to serve requests? |
| **Blue-green** | Prepare a separate new version and switch the Service's target traffic after validation. | Fast switching/rollback is possible, but running both environments costs capacity and requires careful selector management. | Does the switch isolate users from an unhealthy release? |
| **Canary** | Run a small new-version subset alongside the stable version and gradually expand exposure. | Reduces initial exposure but needs monitoring; basic Service/replica proportions are not precise per-user traffic guarantees. | What signals should block canary expansion or trigger rollback? |

These are design concepts, **not evidence that the repository implements all three strategies correctly**.

## Important Kubernetes concepts

### Rolling-update controls

A Deployment can use `strategy.type: RollingUpdate`. Its `maxUnavailable` field limits how many desired replicas can be unavailable during the rollout; `maxSurge` controls how many extra Pods can be created above the desired replica count. Readiness is crucial because a running process is not necessarily ready for traffic.

### Services, labels, and version routing

A Kubernetes Service targets Pods through its selector. For blue-green deployment, a release switch can be modeled by selecting only the intended version. For a simple replica-based canary, a Service can select a common application label across both stable and canary Pods; this does not guarantee a fixed traffic percentage for every client/session.

### Verification before rollback decisions

A useful experiment would distinguish:
- **Process failure:** crash and restart behavior.
- **Readiness failure:** a Pod exists but should not receive Service traffic.
- **Application failure:** health checks pass while the user-facing endpoint fails.
- **Capacity regression:** requests slow down under CPU or replica constraints.

All four are **proposed questions**, not findings from this repository.

## What is in the repository?

```text
manifests/
  rolling/        Deployment and Service sketch
  bluegreen/      Versioned Deployment/Service sketches
  kanary/         Stable/new-version sketch (historical spelling)
spring-app/
  app-v1/         Gitlink/submodule-style reference in the source tree
  app-v2/         Minimal Spring Boot version endpoint and Dockerfile
docs/
  ORIGINAL_STUDY_PLAN.md   Original Korean two-week experiment plan
```

### Known gaps in current scaffolding

Do **not** apply the manifests as a working tutorial without revision:

1. Some Services forward to port **8090** while the container declaration exposes **8080**; actual app ports must be aligned.
2. `manifests/bluegreen/service-v2.yaml` declares Service `apiVersion: v2`, which is not a valid core Service API version. Blue-green traffic selection is not properly expressed in the supplied Service definitions.
3. The blue-green Services use the same name and an overly broad selector, so the files do not perform a verified version-specific traffic switch.
4. The canary manifests use equal replicas for both versions; they do not demonstrate a deliberately small initial canary exposure.
5. Image builds, registry access, probes, test workloads, rollback procedures, and a reproducible cluster environment are not documented or verified.

## Proposed validation steps (not performed)

1. Build and publish two distinguishable application images.
2. Fix the image references, labels, Service selectors, and port mappings.
3. Run each rollout variant on a disposable Kubernetes cluster.
4. Generate concurrent requests and record response versions, errors, and latencies throughout rollout and rollback.
5. Introduce readiness and application-level faults, then compare the observed impact.
6. Save commands, events, logs, manifest revisions, and results before claiming any result.

## References

- [Kubernetes: Deployments and rolling updates](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes: Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes: Canary deployment tutorial](https://kubernetes.io/docs/tutorials/stateless-application/canary-deployment/)

These official docs are useful for understanding the concepts. Any future implementation should follow the API and version guidance appropriate to the chosen cluster.

## Portfolio note

This is a **learning note**, not a production engineering project or a benchmark. The motivation is understanding how deployment strategies handle availability, failure, and safe change. The code and YAML are historical, incomplete drafts.
