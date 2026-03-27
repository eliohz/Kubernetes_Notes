
### RBAC (Role-Based Access Control)

RBAC regelt, wer was im Cluster darf. Die wichtigsten Objekte sind `Role` / `ClusterRole` (Was darf man?) und `RoleBinding` / `ClusterRoleBinding` (Wer bekommt diese Rolle?). Roles gelten namespace-scoped, ClusterRoles clusterwide. Subjects können `User`, `Group` oder `ServiceAccount` sein.
![[Pasted image 20260327104753.png]]

### Autoscaling Basics

Kubernetes kennt drei Arten von Autoscaling:

- **HPA (Horizontal Pod Autoscaler)** – skaliert die Anzahl Pods basierend auf Metriken (z.B. CPU, Memory)
- **VPA (Vertical Pod Autoscaler)** – passt Resource Requests/Limits eines Pods an
- **Cluster Autoscaler** – skaliert Nodes im Cluster hoch/runter je nach Bedarf

### Controller & Operator

Ein **Controller** ist eine Kontrollschleife, die den Ist-Zustand kontinuierlich mit dem Soll-Zustand (aus etcd) vergleicht und angleicht. Ein **Operator** erweitert dieses Prinzip mit Custom Resource Definitions (CRDs) und domänenspezifischer Logik – z.B. für Datenbanken oder komplexe Stateful Apps.

### Paket Manager & Registry

**Helm** ist der De-facto-Paketmanager für Kubernetes. Charts bündeln alle benötigten Manifeste als Template. Eine **Registry** (z.B. Docker Hub, Harbor, GHCR) speichert Container-Images. Im Cluster-Kontext wichtig: Image Pull Secrets für private Registries.

### TLS Basics

TLS sichert die Kommunikation im Cluster. Kubernetes hat eine eingebaute CA (`kubernetes-ca`). Zertifikate werden über `CertificateSigningRequest` (CSR) Objekte beantragt. Tools wie **cert-manager** automatisieren die Ausstellung und Erneuerung von TLS-Zertifikaten (z.B. via Let's Encrypt oder eigener CA).

### DNS

CoreDNS läuft als Deployment im Cluster und übernimmt die interne Namensauflösung. Jeder Service bekommt automatisch einen DNS-Eintrag nach dem Schema:

```
<service>.<namespace>.svc.cluster.local
```

Pods können ebenfalls per DNS erreichbar sein. DNS ist der primäre Weg, wie Services miteinander kommunizieren.