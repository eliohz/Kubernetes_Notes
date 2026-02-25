[← Zurück zur Übersicht](../../../README.md)

# Übung ReplicaSet

In dieser Übung erstellen Sie ein einfaches ReplicaSet und sehen die Unterschiede zu einem einzelnen Pod.
Ebenfalls werden Sie einfache Befehle zur Erstellung und Verwaltung von ReplicaSets lernen.


## Übersicht: Was ein ReplicaSet genau ist

Ein Pod ist einzelne Instanz.

Ein ReplicaSet __stellt sicher, dass eine festgelegte Anzahl eines Pods__ existieren.

| __Funktion__              | _Beispiel_                                  |
| :---                      | :---                                        |
| ReplicaSet erstellen      | _kubectl apply -f ReplicaSet.yaml_          |
| ReplicaSet prüfen         | _kubectl get rs_                            |
| ReplicaSet skalieren      | _kubectl scale rs replicaset  --replicas n_ |



```yaml
# Erklärungen

apiVersion: apps/v1             # apps/v1 → Standard für ReplicaSets
kind: ReplicaSet                # ReplicaSet → Wir erstellen ein ReplicaSet
metadata:                       # ...
  name: my-simple-replicaset    # Der Name des ReplicaSets
spec:                           # ...
  replicas: 3                   # Die Anzahl an anzufertigenden Kopien
  selector:                     # ...
    matchLabels:                # ...
      app: my-simple-app        # Label bestimmt, welches Pods von ReplicaSet verwaltet werden, muss mit Pod-Label übereinstimmen.
  template:                     # Ab hier kommt die Pod Spezifikation
    metadata:                   # ...
      labels:                   # ...
        app: my-simple-app      # Pod-Label
    spec:                       # ...
      containers:               # Die Pods enthalten einen Container mit dem Image nginx:latest
      - name: my-container      # ...
        image: nginx:latest     # ...
        ports:                  # ...
        - containerPort: 80     # ...
```

Das bereitgestellte ReplicaSet deployen.

```bash
kubectl apply -f yaml/replicaset.yaml
```

Prüfen Sie ob die gewünschten Objekte (ReplicaSet, Pods) erstellt wurden. (``kubectl get ...``, ``kubectl describe ...``)
- Was ist bei einem Pod in der Ausgabe von kubectl describe bei "_Controlled By_" gesetzt?

Verändern sie die Anzahl der Replicas, die verwaltet werden mittels ``kubectl scale ...``.
- Was ist das Maximum an Pods was unser Cluster verträgt?
- Was ist das Minimum auf das man skalieren kann?

Als nächstes möchten wir uns ansehen, was das Labeling für Seiteneffekte verursachen kann.
- Was passiert wenn man ein Pod, mit dem gleichen Label deployt?

Beobachten Sie die Pods in einem zweiten Ausgabefenster (Option `-w`).
```bash
kubectl apply -f yaml/pod.yaml
```
- Was passiert wenn man zuerst den Pod deployed und dann das ReplicaSet?
- Wie verändert sich das "_Controlled By_" Feld vor und nach dem Ausrollen des ReplicaSets?


## Cleanup

```bash
kubectl delete -f yaml/pod.yaml
kubectl delete -f yaml/replicaset.yaml
```
