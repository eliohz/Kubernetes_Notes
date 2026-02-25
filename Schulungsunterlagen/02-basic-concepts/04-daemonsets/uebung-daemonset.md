[← Zurück zur Übersicht](../../../README.md)

# Übung DaemonSet

In dieser Übung wollen wir uns mit einem DaemonSet und dessen Eigenschaften im Kubernetes-Cluster vertraut machen.

## Übersicht: Was ein DaemonSet genau ist

Ein DaemonSet __sorgt dafür, dass genau ein Pod pro Node läuft__.

Typische Einsatzszenarien eines Daemonsets sind:
- __Log-Collector__: sammeln von Logs auf jedem Node.
- __Monitoring Agenten__: bereitstellen von Metriken (CPU,Memory) auf einem Node.
- __Sicherheits-Scanner__: Node-relevante Sicherheits-Checks cluster-weit überprüfen.

| Funktion                         | Beispiel                                       |
| :---                             | :---                                           |
| Zeige alle Pods im DaemonSet     | $ kubectl get pods -l name=daemonset -o wide   |
| Zeigt Details eines DaemonSets   | $ kubectl describe daemonset $NAME             |
---


## Einfaches DaemonSet ausrollen (daemonset.yaml)

Das Manifest eines DaemonSets ist wieder sehr ähnlich zu dem von ReplicaSets bzw. Deployments, jedoch gibt es kleine Unterschiede.
Wie das Deployment hat auch das DaemonSet eine Einstellung um mit Updates des Manifests umzugehen.
- Wie heißt das Feld im Falle eines Daemonsets? (`kubectl explain ...`)

```yaml
apiVersion: apps/v1           # apps/v1 → Standard für DaemonSets
kind: DaemonSet               # DaemonSet → Wir erstellen ein DaemonSet
metadata:                     # ...
  name: daemonset             # ...
  labels:                     # ...
    app: daemonset            # ...
spec:                         # ...
  selector:                   # ...
    matchLabels:              # typischer matchLabels Ansatz, um Pods festzulegen.
      name: daemonset         # ...
  template:                   # ...
    metadata:                 # ...
      labels:                 # ...
        name: daemonset       # ...
    spec:                     # ...
      containers:             # Der Pod enthält einen Container mit dem Image nginx
        - name: app
          image: nginx:latest
```

Ausrollen eines einfachen DaemonSets

```bash
cd 04-daemonsets
kubectl apply -f yaml/daemonset.yaml
```

- Auf welchen Nodes werden Pods erstellt?
- Wie unterscheiden sich die Pods im Gegensatz zu Pods von Deployments?
- Durch welche Eigenschaft werden die Pods auf die jeweiligen Nodes zugewiesen? (Tipp: Nach Knotennamen suchen) (`kubectl get pod -o yaml ...`)


## Cleanup

```bash
kubectl delete -f yaml/daemonset.yaml
```
