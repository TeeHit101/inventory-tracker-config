inventory-tracker-config/
├── clusters/
│   └── my-eks-cluster/
│       ├── apps.yaml                 # FluxCD-objekt: Säger åt Flux att applicera /apps
│       ├── infrastructure.yaml       # FluxCD-objekt: Säger åt Flux att applicera /infrastructure
│       └── kustomization.yaml        # Knyter ihop kluster-mappen
│
├── infrastructure/
│   ├── kustomization.yaml            # Läser in filerna nedan
│   ├── secrets-store.yaml            # Konfigurerar ESO mot AWS Secrets Manager
│   └── storage-class.yaml            # Aktiverar AWS EBS gp3-diskar för klustret
│
└── apps/
    ├── kustomization.yaml            # Huvudfilen som laddar namespace + alla appar
    ├── namespace.yaml                # Skapar 'inventory-tracker' namespacet
    │
    ├── postgres/
    │   ├── kustomization.yaml        # Laddar Postgres-filerna
    │   ├── deployment.yaml           # Databas-containern (postgres:15)
    │   ├── pvc.yaml                  # Begär 5Gi lagring via EBS
    │   └── service.yaml              # Intern nätverksadress (postgres-service)
    │
    ├── redis/
    │   ├── kustomization.yaml        # Laddar Redis-filerna
    │   ├── deployment.yaml           # Cache-containern med AOF (persistent data)
    │   ├── pvc.yaml                  # Begär 2Gi lagring via EBS
    │   └── service.yaml              # Intern nätverksadress (redis-service)
    │
    └── inventory-app/
        ├── kustomization.yaml        # Laddar App-filerna
        ├── deployment.yaml           # Din Python Flask-app (Taggen uppdateras av CI/GitHub Actions)
        ├── external-secret.yaml      # Hämtar DB-lösenordet från AWS via ESO
        └── service.yaml              # AWS Network Load Balancer (NLB) för    publik åtkomst