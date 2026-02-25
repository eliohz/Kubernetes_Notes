
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

# Komponenten

### Control-Plane: 
Zentrale Steuerungseinheit des Kubernetes-Clusters
![[Pasted image 20260225140659.png]]

**API-Server (kube-apiserver)**
- Das “Gehirn” des Clusters.
- Nimmt alle Befehle an (z. B. Pods starten, löschen).
- Speichert den gewünschten Zustand aller Ressourcen in **etcd**.

**etcd**
- Einfaches, sicheres Schlüssel-Wert-Speicher-System.
- Speichert den **Soll-Zustand** des Clusters.

**Controller Manager**
- Läuft im Hintergrund und kümmert sich darum, dass alles so läuft wie geplant.
- Prüft, ob der aktuelle Zustand (**IST**) dem gewünschten Zustand (**SOLL**) entspricht.
- Schickt Anweisungen, z. B. an den Scheduler.

**Scheduler (kube-scheduler)**
- Verteilt neue Pods auf die richtigen Nodes.
- Schaut, welche Nodes frei sind und was die Ressourcen brauchen.
- Arbeitet eng mit API-Server und Controller Manager zusammen, damit alles wie gewünscht läuft.

### Worker Node
(Dort laufen die Pods / Anwendungen)
![[Pasted image 20260225140716.png]]

**Kubelet**
- Agent auf jedem Worker Node.
- Spricht mit dem API-Server.
- Startet und überwacht Pods/Container.
- Sorgt dafür, dass der **IST-Zustand** dem **SOLL-Zustand** entspricht.

**Kube-Proxy**
- Kümmert sich um die Netzwerk-Regeln.
- Sorgt dafür, dass Pods miteinander kommunizieren können.
- Leitet Traffic korrekt an die richtigen Pods weiter (Service → Pod).
  
**CRI (Container Runtime Interface)**
- Schnittstelle zwischen Kubelet und der Container Runtime.
- Ermöglicht das Starten und Stoppen von Containern.
- Macht Kubernetes unabhängig von einer bestimmten Container-Technologie.
- Beispiele für Container Runtimes: containerd oder CRI-O.


# Namespace 
## Was ist ein Namespace?
Ein Namespace ist eine logische Unterteilung innerhalb eines Kubernetes-Clusters.
Er dient zur Organisation, Isolation und Zugriffskontrolle von Ressourcen.

## Wofür braucht man Namespaces?

- Trennung von Umgebungen (dev, test, prod)
- Trennung von Teams
- RBAC-Zugriffssteuerung
- ResourceQuotas (CPU / RAM Limits pro Namespace)
- Gleiche Ressourcennamen in unterschiedlichen Namespaces möglich

## Standard Namespaces

- default
- kube-system
- kube-public
- kube-node-lease

## Wichtig

- Namespaces isolieren Ressourcen logisch
- Netzwerkverkehr wird NICHT automatisch getrennt
- Für Netzwerkisolation → NetworkPolicies
