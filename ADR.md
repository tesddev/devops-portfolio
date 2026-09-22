# Architecture Decision Records

## ADR-01: Cloud Provider — AWS

**Context:** Needed a cloud provider with mature managed Kubernetes, a real IAM/OIDC model for keyless CI/CD, and a free tier generous enough to run a full stack (VPC, EC2, EKS) without an immediate bill.

**Options considered:**
- **AWS** — most mature ecosystem, most job-market relevance, steepest IAM learning curve
- **DigitalOcean** — simpler pricing and UX, weaker IAM/OIDC story, smaller managed K8s ecosystem
- **GCP** — strong GKE, but less overlap with the IAM/security patterns used in the target job market

**Decision:** AWS, using EC2 for the early server-hardening work and EKS once the platform moved to Kubernetes.

**Consequences:** IAM complexity (OIDC trust policies, IRSA, security group layering with UFW) added real setup time that a simpler provider wouldn't have required — but that complexity is also exactly what's tested in real DevOps roles, so it was time well spent rather than overhead to minimize. Free-tier limits (particularly EKS node pricing, which isn't in the free tier) became the actual constraint that forced later scope decisions — see ADR-06.

## ADR-02: Kubernetes Distribution — kind (local) → EKS (cloud)

**Context:** Needed to learn Kubernetes mechanics before paying for managed infrastructure, then needed a production-realistic managed cluster once the platform work required IAM integration, load balancer provisioning, and autoscaling.

**Options considered:**
- **kind for everything** — free, fast iteration, but no IAM/ALB/EBS integration, so it can't teach the parts of K8s that are actually cloud-specific
- **EKS from day one** — production-realistic, but expensive to iterate on while still learning basic kubectl fluency
- **k3s on a single EC2 box** — cheaper than EKS, but a single-node "cluster" doesn't exercise scheduling, node autoscaling, or multi-node failure modes — which matters directly for the chaos engineering work later

**Decision:** kind locally for fundamentals (manifests, HPA, StatefulSets, kubectl fluency), then EKS once the work required IAM roles for service accounts, an ALB controller, and genuine multi-node behavior.

**Consequences:** Two different environments meant occasionally relearning small differences between them (ingress controller tolerations behaved differently between kind and EKS, for one). Worth it — EKS costs money per hour whether or not you're using it, so front-loading the cheap iteration onto kind kept the AWS bill down during the phase with the most trial and error.

## ADR-03: GitOps Tool — ArgoCD

**Context:** CI/CD was pushing changes to the cluster directly from GitHub Actions, which meant GitHub held live credentials to the cluster — a real security surface, and also meant "what's running" could silently drift from "what's in Git" if anyone ran a manual `kubectl apply`.

**Options considered:**
- **ArgoCD** — pull-based, has a UI, widely used, strong self-heal/drift-correction behavior
- **Flux** — also pull-based and GitOps-native, more CLI-centric, lighter-weight but less approachable for demoing to non-CLI reviewers
- **Keep push-based CD** — simplest to keep, but leaves long-lived cluster credentials in CI and no drift protection

**Decision:** ArgoCD, with `syncPolicy.automated` (prune + self-heal) enabled.

**Consequences:** Gained real drift protection — deleting a resource on the cluster gets it recreated automatically, which was demonstrated directly rather than just claimed. Downside: it also means the cluster can no longer be hand-patched during an incident without that fix immediately getting reverted by the next sync — the fix has to go through Git, which adds a step during a live incident. That trade-off is deliberate (it's the same reason production teams use GitOps at all) but it's a real cost, not a free win.

## ADR-04: Secrets Management — HashiCorp Vault (partially implemented)

**Context:** Kubernetes Secrets are base64-encoded, not encrypted — anyone with namespace read access can decode them with one command. Wanted encrypted-at-rest storage and dynamic secret injection without putting credentials in the Deployment spec.

**Options considered:**
- **Kubernetes Secrets as-is** — zero setup cost, but not actually secure against anyone with cluster read access
- **AWS Secrets Manager** — good AWS-native option, but ties secret retrieval to AWS APIs specifically rather than a portable pattern
- **HashiCorp Vault** — industry-standard, supports dynamic secrets and rotation, but has real setup cost (Kubernetes auth method, injector webhook, policies)

**Decision:** Vault, via the Vault Agent Injector sidecar pattern.

**Consequences:** Got as far as storing secrets in Vault's KV engine and configuring the Kubernetes auth method before the work was paused to move into the DevSecOps time budget elsewhere — the injector-based delivery to `/vault/secrets/config.env` and the full replacement of Kubernetes Secret objects wasn't finished. Documenting this as incomplete rather than as done is the honest state of it; the design decision (Vault over Secrets Manager, for portability) still stands as the intended direction.

## ADR-05: Service Mesh — Linkerd

**Context:** Needed encrypted (mTLS) service-to-service traffic and golden-signal metrics (request rate, error rate, latency) without instrumenting each service's code individually.

**Options considered:**
- **Linkerd** — lightweight, Rust-based data plane, simple installation, strong out-of-the-box metrics via `linkerd viz`
- **Istio** — more feature-rich (traffic mirroring, more elaborate routing), but heavier resource footprint and steeper operational complexity — a real concern on `t3.small` nodes already tight on pod density
- **No mesh, manual mTLS** — avoids the extra moving part entirely, but pushes certificate rotation and metrics instrumentation into every service by hand

**Decision:** Linkerd.

**Consequences:** mTLS and per-service metrics came essentially for free once namespaces were mesh-injected — confirmed directly via `linkerd viz tap` rather than assumed. The resource footprint concern was real, though: Linkerd's own control-plane pods contributed to hitting the per-node pod-density ceiling (EKS caps pods per node by ENI/IP allocation, not CPU/memory) once Chaos Mesh was added alongside it — a genuine capacity trade-off, not a hypothetical one.
