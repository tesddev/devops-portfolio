# Build Log

## Linux Server Hardening & Web Serving
Provisioned an EC2 Ubuntu server, hardened SSH (key-only, custom port, non-root), configured UFW, and served a page over Nginx with Let's Encrypt TLS — all wrapped in an idempotent setup script. **Learned:** Ubuntu 24.04's `ssh.socket` silently overrides `sshd_config`'s `Port` directive unless you disable socket activation — cost real debugging time. **Would do differently:** write the idempotency checks before the actions, not after — retrofitting "check if already done" onto a script written procedurally is more work than designing for it from the start.
Repo: https://github.com/tesddev/server-bootstrap

## API Development, Deployment, Docker & CI/CD
Built a FastAPI service, ran it under systemd, put Nginx in front as a reverse proxy, then containerized it with a multi-stage Dockerfile and stood up CI (lint/test/build) and CD (build → push → SSH deploy → smoke test) in GitHub Actions.\n 
**Learned:** GHCR's first push needs explicit package-to-repo linkage under "Manage Actions access" — `GITHUB_TOKEN` alone silently isn't enough, and the failure mode doesn't say why. **Would do differently:** set up Buildx multi-platform builds from day one instead of hitting an arm64/amd64 crash-loop on the EC2 host after developing on Apple Silicon.
Repo: https://github.com/tesddev/fastapi-nginx-service

## Infrastructure as Code — Terraform & Ansible
Wrote a modular Terraform layout (`modules/networking`, `modules/compute`, per-environment `envs/`) with S3+DynamoDB remote state, and Ansible roles (`common`, `docker`, `app`) with vault-encrypted secrets to configure what Terraform provisioned. **Learned:** on macOS, Ansible's `become_user` tasks need to clone as root and fix ownership as a separate step — the privilege model doesn't translate cleanly cross-platform. **Would do differently:** run `aws sts get-caller-identity` as a hard pre-flight check before every apply — nearly targeted the wrong AWS account more than once.
Repo: https://github.com/tesddev/terraform-infra

## Kubernetes Fundamentals
Ran a local `kind` cluster, wrote full manifest sets (Deployment/Service/Ingress/ConfigMap/Secret) for the API, configured HPA and watched it scale under generated load, and ran PostgreSQL as a StatefulSet to prove data survives pod restarts. **Learned:** `ingress-nginx` needs `operator: Exists` (not `Equal`) to tolerate valueless taints on newer kind versions. **Would do differently:** set resource requests from the start — HPA is silently non-functional without them, and that's an easy thing to discover only once you're already debugging why scaling isn't happening.
Repo: https://github.com/tesddev/k8s-manifests

## Observability — Monitoring, Logging & Alerting
Deployed Prometheus, Node Exporter, Grafana, and Loki/Promtail; wrote PromQL and LogQL queries; wired Alertmanager to Slack with real alert rules (instance down, high CPU, low disk). **Learned:** on macOS `localhost` resolves to IPv6 by default, which breaks assumptions that hold fine on Linux — use `127.0.0.1` explicitly. **Would do differently:** size the stack for `t3.micro` constraints up front rather than discovering resource pressure once everything's running simultaneously.
Repo: https://github.com/tesddev/monitoring

## Advanced Kubernetes — Helm, ArgoCD & Service Mesh
Provisioned EKS via Terraform, packaged the app as a Helm chart, set up ArgoCD for GitOps (auto-sync + self-heal demonstrated), and installed Linkerd for mTLS between services with a traffic-split canary demo. **Learned:** ArgoCD's reconciliation loop overrides local `kubectl delete` — any fix has to go through Git, not the cluster directly, or it just gets reverted on the next sync. **Would do differently:** provision the IAM OIDC trust and IRSA roles before touching workloads — retrofitting them onto already-running pods meant extra restarts.
Repo: https://github.com/tesddev/helm-charts

## Production-Grade SRE Microservices Platform
Built a 3-service microservices platform (api-gateway, auth-service, notification-service) fully managed by ArgoCD, meshed with Linkerd (mTLS verified via `linkerd viz tap`), with a golden-metrics Grafana dashboard, distributed tracing via Tempo, an SLO with a multi-window burn-rate alert, four incident runbooks tested via a live drill with a blameless postmortem, and three chaos engineering experiments (pod kill, network delay, node drain) each run against a stated hypothesis. **Learned:** EKS pod density is capped by ENI/IP allocation per node (`kubelet --max-pods`), not CPU/memory — hit a 33-pod ceiling installing Chaos Mesh alongside the existing stack, which CPU/memory graphs gave zero warning about. **Would do differently:** enable VPC CNI prefix delegation from the start rather than discovering the pod-density ceiling mid-experiment.
Repo: https://github.com/tesddev/platform-services
