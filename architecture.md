# Architecture

```mermaid
flowchart TB
    User["User Browser"]

    subgraph Edge["Edge"]
        DNS["DuckDNS / Domain"]
        ALB["Nginx / AWS ALB<br/>TLS Termination"]
    end

    subgraph EKS["EKS Cluster — dev-eks"]
        subgraph Mesh["Linkerd Service Mesh (mTLS)"]
            GW["api-gateway"]
            AUTH["auth-service"]
            NOTIF["notification-service"]
        end

        subgraph Obs["Observability — monitoring namespace"]
            PROM["Prometheus (linkerd-viz)"]
            GRAF["Grafana"]
            TEMPO["Tempo — traces"]
            ALERT["Alertmanager"]
        end

        ARGO["ArgoCD"]
    end

    GIT["Git — platform-services repo<br/>(Helm charts + Application manifests)"]
    SLACK["Slack — alert notifications"]

    User --> DNS --> ALB --> GW
    GW <-->|mTLS| AUTH
    GW <-->|mTLS| NOTIF

    GW -.->|metrics/traces| PROM
    AUTH -.->|metrics/traces| PROM
    NOTIF -.->|metrics/traces| PROM
    PROM --> GRAF
    GW -.->|spans| TEMPO
    AUTH -.->|spans| TEMPO
    PROM --> ALERT --> SLACK

    GIT -.->|watches & syncs| ARGO
    ARGO -.->|reconciles| GW
    ARGO -.->|reconciles| AUTH
    ARGO -.->|reconciles| NOTIF
```

**Legend:** solid arrows = live request traffic. Dashed arrows = the GitOps reconciliation loop (ArgoCD watching Git and applying changes) and the telemetry loop (services emitting metrics/traces to the observability stack) — neither is triggered by a user request.
