
# Pods

Ein Pod ist eine Art „Kapsel“ für deine Anwendung. Er gruppiert einen oder mehrere Container (meistens Docker-Container) zusammen, damit sie als eine Einheit verwaltet werden können.
(Best Practise -> Ein Service pro Pod)
# Labels & Selectors

## Zweck

Labels verknüpfen Ressourcen miteinander. ohne feste Verdrahtung. Ein Service findet seine Pods nicht über Namen, sondern über Labels. Das macht alles flexibel: Pods kommen und gehen, der Service bleibt gleich.

---

## Wo werden Labels definiert?

Direkt im YAML der Ressource, unter `metadata.labels`:

```yaml
# z.B. in einem Pod oder Deployment
metadata:
  name: mein-pod
  labels:
    app: frontend      # selbst gewählter Key: Value
    env: production
```

> Labels kannst du auf **jeden** Kubernetes-Objekt setzen — Pod, Deployment, Service, Node…

---

## Eigenschaften

- Frei wählbar — kein fixes Schema
- Reine Metadaten, kein Einfluss auf das Verhalten
- Änderungen wirken sofort — verliert ein Pod ein Label, fällt er aus dem Service heraus

---

## Selectors

Ein Selector sagt: _"Gib mir alle Objekte mit diesen Labels."_

```yaml
# z.B. in einem Service
selector:
  app: frontend
  env: production
# → trifft nur Pods die BEIDE Labels haben
```

Mehrere Kriterien = **AND** — alle müssen passen.

---
# ReplicaSet

Stellt sicher, dass zu jedem Zeitpunkt eine **bestimmte Anzahl identischer Pods** läuft. 
Wenn ein Pod abstürzt, erstellt das ReplicaSet automatisch einen neuen. 
Wird heute meist nicht direkt verwendet, sondern indirekt über ein Deployment verwaltet.

---

# Deployment

Das **Standardmittel für zustandslose Applikationen**. 
Ein Deployment verwaltet ein ReplicaSet und ermöglicht zusätzlich kontrollierte **Updates und Rollbacks** 
z.B. Rolling Updates, bei denen Pods schrittweise ersetzt werden, ohne Downtime.

---

# DaemonSet

Sorgt dafür, dass **auf jedem Node genau ein Pod** läuft. 
Ideal für Systemdienste wie Log-Collector, Monitoring-Agents oder Netzwerk-Plugins, die auf jeder Maschine im Cluster vorhanden sein müssen. 
Neuer Node = automatisch neuer Pod.

---

# Job / CronJob

- **Job**: Führt eine Aufgabe **einmalig bis zur erfolgreichen Beendigung** aus – z.B. Datenbankmigrationen oder Batch-Verarbeitungen. Im Gegensatz zu Deployments soll der Pod am Ende _stoppen_.
- **CronJob**: Wie ein Job, aber **zeitgesteuert** (nach Linux-Cron-Syntax) – z.B. täglich um 02:00 Uhr ein Backup ausführen.

