
[← Zurück zur Übersicht](./README.md)
### Cloud Native
-> Entwicklungsansatz
-> Von Anfang an alles in der Cloud
### Cloud Angostisch
-> Teil Cloud Lösung (Hybrid wie z.B. Azure mit AD sync)

## Begriffe

- **Container** → isolierte Laufzeitumgebung für eine Anwendung + ihre Abhängigkeiten (z. B. Docker-Container).
- **Pod** → kleinste deploybare Einheit in Kubernetes; kann einen oder mehrere Container enthalten, die Ressourcen teilen (Netzwerk, Volumes).
- **Service** → abstrakte Schnittstelle, die Pods über stabile IP/Domain erreichbar macht; ermöglicht Lastverteilung und Service Discovery innerhalb des Clusters.

### Kubernetes Aufbau
![[Pasted image 20260225134123.png]]

## Komponenten

**Control-Plane**: Zentrale administrative Service-Komponente auf dem Master


