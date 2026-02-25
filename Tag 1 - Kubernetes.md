
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

#### Control-Plane: 

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



