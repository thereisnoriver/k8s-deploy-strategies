# Rolling Updates, Blue-Green, and Canary Deployments

**Source type:** Conceptual comparison, not implementation results.

**Adapted from:** [Zero-downtime deployment methods (conceptual) (/40)](https://tyge-ottesen-brahe.tistory.com/40)

## Motivation: availability during change

Deploying an updated application can briefly break requests if old instances stop before new ones become usable. Deployment strategies manage the transition between versions, but *zero downtime is a goal to verify*, not an intrinsic promise made by a strategy name.

## Rolling update

A Kubernetes Deployment with `strategy.type: RollingUpdate` progressively creates new Pods and scales down old ones.

- `maxSurge`: permitted temporary replicas above the desired count.
- `maxUnavailable`: permitted unavailable replicas relative to the desired count.
- **Readiness probes:** Help keep endpoints out of Service traffic until they can serve requests.
- **Progress and rollback:** A stalled rollout can be detected; an operator or automation may initiate a rollback to a retained revision. Kubernetes should not be described as universally performing automatic rollback of a bad release.

**Trade-off:** Lower duplicate-environment overhead than a full blue-green setup, but old and new versions may coexist, so APIs, schemas, and state changes need compatibility planning.

## Blue-green deployment

Maintain an existing **blue** version while preparing a candidate **green** version. After validating green, switch traffic using an appropriate router, Service selector, Gateway, or load-balancer configuration. Retaining a healthy blue environment can make switching back faster.

**Trade-off:** Running two complete environments *can* demand substantial extra capacity, but the exact cost depends on architecture and scheduling. Once the old environment is removed, a fast switch back is no longer guaranteed. Data migrations and persistent state can complicate any rollback.

## Canary deployment

Run a small candidate release next to the stable release. Route a limited share of traffic to it, observe user-facing health and error signals, then expand or revert.

**Trade-off:** Risk can be limited to a subset of traffic, but controlling actual exposure requires a concrete routing scheme. A 1:9 ratio of Pods selected by an ordinary Service is **not a guaranteed 10% of requests or users**. Weighted or identity-aware traffic routing requires suitable infrastructure.

## Comparison

| Concern | Rolling | Blue-green | Canary |
| --- | --- | --- | --- |
| Main action | Incrementally replace replicas | Switch traffic between environments | Gradually expand candidate exposure |
| Concurrent versions | During rollout | Usually two environments | Stable + candidate |
| Useful early check | Readiness and capacity | Candidate validation before cutover | Error and latency signals in a limited cohort |
| Rollback condition | Prior revision still available | Prior environment and compatible state retained | Traffic route and stable capacity still available |

## Questions for a future experiment

1. Can requests continue while version responses change?
2. What happens when a new Pod passes a shallow health check but its business endpoint fails?
3. How do rollout settings behave under limited cluster capacity?
4. How quickly can traffic return to the stable version, and does data remain compatible?
5. Can the chosen canary router measure and enforce an intended traffic share?

**These are proposed evaluation questions; no results are recorded here.**

## References

- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Canary tutorial](https://kubernetes.io/docs/tutorials/stateless-application/canary-deployment/)
