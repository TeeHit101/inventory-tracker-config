# Inventory Tracker - GitOps Application Configuration

Detta arkiv innehåller Kubernetes-manifest och Kustomize-konfigurationer för att driftsätta applikationen **Inventory Tracker** (inklusive dess databas- och cache-infrastruktur). 

Arkivet är utformat för att köras som en isolerad tenant-applikation i en multi-tenant Kubernetes-miljö (t.ex. K3s med Flux CD).

---

## 🛠️ Arkitektur & Komponenter

Applikationen är uppdelad i tre delar under mappen `apps/`:

* **`inventory-app/`**: Flask-baserad webbapplikation som ansluter till Postgres och Redis.
  * Exponeras via en `Service` av typen `LoadBalancer` (tilldelas IP-adress lokalt av t.ex. MetalLB).
* **`postgres/`**: PostgreSQL-databas för persistent lagring.
  * Använder en `PersistentVolumeClaim` (PVC) kopplad till **Longhorn** (`storageClassName: longhorn`) för hög tillgänglighet och replikering.
* **`redis/`**: Redis-cache.
  * Använder också en **Longhorn**-backed PVC för att spara cache-tillstånd.

---

## 🔐 Säkerhet & Secrets (Viktigt)

Av säkerhetsskäl sparas **inga lösenord eller känsliga uppgifter i detta Git-arkiv**. 

Applikationen förväntar sig en befintlig Kubernetes Secret i samma namnutrymme som heter **`inventory-secrets`**. Denna måste skapas manuellt i klustret innan synkningen kan slutföras.

### Skapa Secret manuellt:
Kör följande kommando på din Kubernetes-server (ersätt `tenant-a` med namnet på ditt namnutrymme):

```bash
kubectl create secret generic inventory-secrets \
  --namespace tenant-a \
  --from-literal=username="inventory_user" \
  --from-literal=password="DITT_SÄKRA_LÖSENORD"
```

---

## 🚀 Driftsättning via GitOps

Denna applikation styrs och synkroniseras via ditt centrala infrastrukturarkiv (`k8s-infrastructure`). 

Genom att lägga till en `GitRepository`- och en `Kustomization`-resurs i din tenant-konfiguration pekar Flux på detta arkivs `apps/`-mapp och rullar ut resurserna säkert under tenantens begränsade ServiceAccount:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: application-config
  namespace: tenant-a
spec:
  interval: 10m0s
  path: ./apps
  prune: true
  sourceRef:
    kind: GitRepository
    name: application-config
  serviceAccountName: tenant-a-deployer
  targetNamespace: tenant-a
```