# Cold Case CC-03: The Intermittent 502

**Domain**: Kubernetes / Ingress / Application Lifecycle  
**Platform Service**: `kolo-ledger` (Core Financial Ledger Service)  

---

## Scenario Briefing

Following Tuesday's scheduled deployment of `kolo-ledger`, the SRE on-call was paged for elevated 5xx error rates. Out of approximately 15,000 requests processed during the rollout, around 50 requests failed with `HTTP 502 Bad Gateway` (~0.3% - 0.4% error rate).

The on-call engineer inspected the Kubernetes dashboard and observed:
- All `kolo-ledger` pods showed `Running` status with `0` restarts.
- The 502 error spikes aligned temporally with the timestamps of deployment rollouts (`kubectl rollout restart`).
- The engineer concluded that one or more pods must have crashed during startup or shutdown, but failed to find any `OOMKilled` or `CrashLoopBackOff` events.

You have been provided with authentic forensic artifacts captured directly from the ingress controller and cluster control plane during the incident window.

---

## Forensic Artifacts (`learner/`)

- `deployment.yaml`: Kubernetes Deployment manifest for `kolo-ledger`.
- `ingress-access.log`: NGINX ingress access logs containing upstream connection and response statuses.
- `endpointslice.yaml`: Snapshot of the Kubernetes `EndpointSlice` captured during the rollout transition.
- `rollout-events.log`: Output of cluster events recorded during the pod replacement timeline.

---

## Your Deliverables

1. **Root Cause Analysis**: Explain the exact race condition causing the 502 errors. Why did the errors only occur during the rolling update despite the pods being healthy?
2. **Decoy Analysis**: Explain why the alignment with pod lifecycle events misled the on-call engineer into suspecting application crashes.
3. **Mechanical Chain**: Detail the asynchronous sequence between Kubelet `SIGTERM`, process termination, and EndpointSlice/iptables deregistration across the cluster.
4.                                      **Remediation**: Provide the exact Kubernetes deployment patch to eliminate the 502 errors and guarantee zero dropped in-flight requests.
