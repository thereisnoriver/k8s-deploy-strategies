# Kubernetes Deployment Strategies — Study Notes

**Portfolio category:** Kubernetes learning notes, practical setup walkthroughs, and proposed deployment experiments. **Not a validated end-to-end deployment benchmark.**

This repository connects eight [original Korean technical blog posts](https://tyge-ottesen-brahe.tistory.com/category/Kubernetes) with an English study guide. The posts document conceptual study, Google Cloud / kubeadm practice-oriented walkthroughs, and rolling-update experiment planning. They **do not provide a complete, reproducible record of successful rolling, blue-green, or canary deployment tests**.

## Read the notes

| Topic | English note | Evidence level |
| --- | --- | --- |
| Containers and Kubernetes fundamentals | [Container orchestration and components](docs/01_CONTAINER_ORCHESTRATION.md) | Conceptual explanation |
| Google Cloud and Kubernetes basics | [Cluster setup and basic workloads](docs/02_CLUSTER_SETUP.md) | Practical walkthrough notes; no preserved execution transcript |
| Rolling, blue-green, and canary releases | [Deployment strategies](docs/03_DEPLOYMENT_STRATEGIES.md) | Conceptual comparison |
| Failure and recovery testing | [Rolling-update experiment design](docs/04_ROLLING_UPDATE_EXPERIMENT_PLAN.md) | Proposed experiments / partial-progress notes; no recorded outcomes |
| Learning schedule | [12-week roadmap](docs/ROADMAP.md) | Plan, not completion evidence |
| Original repository checklist | [14-day plan (Korean)](docs/ORIGINAL_STUDY_PLAN.md) | Historical plan |

All eight source posts are linked individually in the respective notes.

## Deployment strategies at a glance

| Approach | How traffic changes | Main risk / trade-off |
| --- | --- | --- |
| **Rolling update** | A Deployment gradually replaces Pods; readiness and `maxSurge`/`maxUnavailable` constrain the process. | Availability is not automatic if health checks, capacity, or application compatibility are wrong. |
| **Blue-green** | Deploy a distinct candidate environment, validate, then change which version receives traffic. | Requires sufficient capacity and carefully scoped traffic selectors; rollback depends on retaining the old environment. |
| **Canary** | Introduce a new version to a limited share before expanding. | Replica counts alone do not guarantee exact request percentages or user cohorts. |

## Repository contents and limitations

```text
docs/                  English notes, source links, and historical plans
manifests/rolling/     Historical Deployment and Service example
manifests/bluegreen/   Historical versioned examples
manifests/kanary/      Historical canary example (original directory spelling)
spring-app/app-v1/     Gitlink in the original source tree
spring-app/app-v2/     Small Spring Boot /version endpoint
```

The source-code and manifest directories are **incomplete learning scaffolding**, not runnable reference architecture. Known issues include:

- Example Services target **8090**, while the corresponding container declarations show **8080**; verify the *actual application listening port* before changing either.
- One blue-green Service uses invalid `apiVersion: v2` instead of `v1`; the Services also have conflicting names / selectors and do not demonstrate controlled blue-green switching.
- The canary examples use equal replica counts and do not establish limited or weighted rollout traffic.
- No repeatable cluster configuration, successful rollout output, failure-injection logs, latency data, or production validation has been committed.

These limitations are intentional disclosures, **not claims that the proposed experiments were completed**.

## How I would validate the examples next

1. Build two distinguishable images and fix Service ports, labels, selectors, and manifest API versions.
2. Reproduce a disposable cluster environment, documenting versions and commands.
3. Add suitable readiness probes and verify endpoint behavior while changing `maxSurge` and `maxUnavailable`.
4. Record requests, errors, Pod transitions, and rollback behavior; compare against expectations.
5. Add blue-green traffic switching and a clearly defined canary traffic-routing method only after the rolling baseline works.

## References

- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- [Deployments and rolling updates](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes canary tutorial](https://kubernetes.io/docs/tutorials/stateless-application/canary-deployment/)

**Scope:** Independent study and documentation. Neither this README nor the blog series should be interpreted as production deployment experience.
