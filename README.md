# DevOps Platform — 12 Weeks, Zero to Production-Grade

## Overview
This is a production-grade DevOps/SRE platform built over 12 weeks, starting from a bare Ubuntu server and ending with a three-service microservices platform on EKS — managed end-to-end through GitOps, meshed for encrypted service-to-service traffic, observed with metrics, logs, and traces, alerted on an SLO error budget, and stress-tested with real chaos engineering experiments against stated hypotheses. Every piece here was actually deployed, broken, and fixed — not just described. Two originally planned areas (a broader internal-platform layer, and full secrets/RBAC/network-policy hardening) were consciously cut short partway through to protect AWS budget and put full depth into the final integrative platform work instead; that trade-off, and why it was made, is documented in [ADR.md](./ADR.md).

## Architecture
See [architecture.md](./architecture.md) for the full diagram — request path, service mesh, and the GitOps control loop that keeps the cluster reconciled.

## Platform Tour
No recorded walkthrough — see [platform-tour.md](./platform-tour.md) for a written tour covering the same ground: GitOps sync, the live dashboard, a chaos experiment recovering, and an alert firing end to end.

## What I Built
- [Linux Server Hardening & Web Serving](https://github.com/tesddev/server-bootstrap) — hardened Ubuntu EC2 server, SSH lockdown, UFW, Nginx + Let's Encrypt, idempotent setup script
- [API Development, Deployment, Docker & CI/CD](https://github.com/tesddev/fastapi-nginx-service) — FastAPI service under systemd behind Nginx, multi-stage Docker build, full CI (lint/test/build) and CD (build → push → deploy → smoke test) in GitHub Actions
- [Infrastructure as Code — Terraform & Ansible](https://github.com/tesddev/terraform-infra) — modular Terraform (networking/compute modules, per-environment layout, S3+DynamoDB remote state) and Ansible roles for configuration
- [Kubernetes Fundamentals](https://github.com/tesddev/k8s-manifests) — local `kind` cluster, full manifest sets, HPA under real load, PostgreSQL as a StatefulSet with verified data persistence
- [Observability — Monitoring, Logging & Alerting](https://github.com/tesddev/monitoring) — Prometheus, Node Exporter, Grafana, Loki/Promtail, Alertmanager routed to Slack
- [Advanced Kubernetes — Helm, ArgoCD & Service Mesh](https://github.com/tesddev/helm-charts) — EKS via Terraform, Helm-packaged app, ArgoCD GitOps with auto-sync and self-heal, Linkerd mTLS with a traffic-split canary
- [Production-Grade SRE Microservices Platform](https://github.com/tesddev/platform-services) — three meshed services managed by ArgoCD, distributed tracing, an SLO with burn-rate alerting, tested incident runbooks with a blameless postmortem, and three chaos engineering experiments

See [build-log.md](./build-log.md) for the full breakdown of each — what was built, what was learned, what I'd change.

## Architecture Decisions
See [ADR.md](./ADR.md) — six decisions: cloud provider, Kubernetes distribution, GitOps tool, secrets management, service mesh, and the call to stop before Platform Engineering.

## Technologies Used
**Compute & Orchestration**
- [Kubernetes](https://kubernetes.io/docs/) ([kind](https://kind.sigs.k8s.io/) for local, [EKS](https://docs.aws.amazon.com/eks/) for cloud)
- [kubectl](https://kubernetes.io/docs/reference/kubectl/) / [k9s](https://k9scli.io/)
- [Helm](https://helm.sh/docs/)
- [ArgoCD](https://argo-cd.readthedocs.io/)
- [Linkerd](https://linkerd.io/2/overview/)
- [ingress-nginx](https://kubernetes.github.io/ingress-nginx/) / [cert-manager](https://cert-manager.io/docs/)

**Infrastructure as Code**
- [Terraform](https://developer.hashicorp.com/terraform/docs)
- [Ansible](https://docs.ansible.com/)

**Containers & CI/CD**
- [Docker](https://docs.docker.com/) / [Docker Compose](https://docs.docker.com/compose/) / [Buildx](https://docs.docker.com/build/buildx/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [GHCR](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

**Observability**
- [Prometheus](https://prometheus.io/docs/introduction/overview/) / [Node Exporter](https://github.com/prometheus/node_exporter)
- [Grafana](https://grafana.com/docs/grafana/latest/)
- [Loki](https://grafana.com/docs/loki/latest/) / [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/)
- [Tempo](https://grafana.com/docs/tempo/latest/) (distributed tracing)
- [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/)

**Security**
- [Trivy](https://aquasecurity.github.io/trivy/) (image, config, and SBOM scanning)
- [HashiCorp Vault](https://developer.hashicorp.com/vault/docs) (secrets — partial)

**Chaos Engineering**
- [Chaos Mesh](https://chaos-mesh.org/docs/)

**Server & Networking**
- [systemd](https://www.freedesktop.org/software/systemd/man/systemd.html)
- [Nginx](https://nginx.org/en/docs/)
- [Certbot / Let's Encrypt](https://certbot.eff.org/)
- [UFW](https://help.ubuntu.com/community/UFW)
- [DuckDNS](https://www.duckdns.org/)
- [ssh-audit](https://github.com/jtesta/ssh-audit)

**Cloud**
- AWS: EC2, EKS, VPC, [IAM/OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect), S3, DynamoDB, ALB

## Blog Post
<!-- link once published -->
