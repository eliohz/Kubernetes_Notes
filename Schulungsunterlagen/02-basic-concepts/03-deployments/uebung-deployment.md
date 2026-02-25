[← Zurück zur Übersicht](../../../README.md)

# Übung Deployments

In dieser Übung beschäftigen wir uns mit dem ausrollen von Deployments in Kubernetes. Wir setzten uns hierbei mit den neuen Eigenschaften die ein solches Deployment mit sich bringt. Zudem kann man das Verhalten mit einem neuen Kommando von _kubectl_ beeinflussen.


## Übersicht: Was ein Deployment genau ist

Deployments führt eine höherer Abstraktionsebene zu ReplicaSets ein und ermöglicht dadurch weitere Funktionen.
Kubernetes verwaltet Deployments und __leitet__ davon ReplicaSets ab.
Änderungen am Deployment führt zu einer neuen Version des ReplicaSets, __Rollbacks__ ermöglichen zu einer früheren Änderung zurückzuspringen.
Es werden __Upgrades mit wenig Aufwand__ durchgeführt.

> __Ein Deployment sorgt für den gewünschten Zustand des ReplicaSets und aktualisiert den Pod.__

| __Funktion__                      | _Beispiel_                                                    |
| :---                              | :---                                                          |
| Pods prüfen                       | _kubectl get pods_                                            |
| Rollouts durchsetzen              | _kubectl apply -f deployment.yaml_                            |
| Rollbacks durchsetzen             | _kubectl rollout undo deployment/my-simple-deployment_        |
| Status des Deployments anzeigen   | _kubectl rollout status deployment/my-simple-deployment_      |
| Skalierung des Deployments        | _kubectl scale deployment my-simple-deployment --replicas=5_  |
| Den Status des Services prüfen	| _kubectl get services_                                        |

## Deployment-Definition (deployment.yaml)

Deployments sehen beim ersten Blick ziemlich Ähnlich aus wie ReplicaSets.
Jedoch kann man über die Einstellung `spec.strategy` festlegen, wie ein Update stattfinden soll. (``kubectl explain ...``)

```yaml
apiVersion: apps/v1               # apps/v1 → Standard für Deployments
kind: Deployment                  # Deployment → Wir erstellen ein Deployment
metadata:                         # ...
  name: my-simple-deployment      # Der Name des Deployments
spec:                             # ...
  replicas: 3                     # Die Anzahl an anzufertigenden Kopien
  selector:                       # ...
    matchLabels:                  # ...
      app: my-simple-app          # ...
  template:                       # ...
    metadata:                     # ...
      labels:                     # ...
        app: my-simple-app        # ...
    spec:                         # ...
      containers:                 # Die Pods enthalten einen Container mit dem Image nginx:latest
      - name: my-container        # ...
        image: nginx:latest       # ...
        ports:                    # ...
        - containerPort: 80       # ...
```

Ausrollen des Deployment

```bash
kubectl apply -f yaml/deployment.yaml
```


## Deployments verwalten

Erstellen Sie eine Kopie des aktuellen Deployment-Manifests und machen Sie die folgenden Änderungen.

```yaml
# NOTE: incomplete YAML
metadata:
  annotations:
    kubernetes.io/change-cause: "set image to httpd"
...
spec:
  template:
    spec:
      containers:
      - name: my-container
        image: httpd:latest
```

Überwachen Sie in einem zweiten Fenster die Pods (``watch kubectl ...``) und rollen Sie danach die Änderungen des neues Manifests aus (``kubectl apply -f ...``)
Kubernetes wird automatisch einen Rolling Update durchführen, sodass deine Pods nach und nach aktualisiert werden.

## Rollouts

Mit dem Befehl ``kubectl rollout ...`` können Sie Einfluss auf das Rollout eines Deployments oder anderer Ressourcen nehmen.
Es ermöglicht den Status von einem Rollout zu einem Deployment anzusehen.
Außerdem ermöglicht die Rollout Historie anzusehen und das Deployment auf eine frühere Version zurücksetzten.


Status des Deployments anzeigen:

```bash
kubectl rollout status deployment/my-simple-deployment
```


Historie des Deployments anzeigen:

```bash
kubectl rollout history deployment/my-simple-deployment
```


Rollback durchführen und Deployment auf das nginx-image zurücksetzten:

```bash
kubectl rollout undo deployment/my-simple-deployment
# und gleich den status verfolgen ...
kubectl rollout status deployment/my-simple-deployment
```

Die Historie sollte jetzt eine neue Revision anzeigen und die Pods wieder mit nginx laufen.


## Optional

Das Hinzufügen von Annotationen an ein Objekt ist auch manuell mit kubectl annotate möglich:

```bash
kubectl annotate deployment my-simple-deployment kubernetes.io/change-cause="image=nginx"
```

Wie auch beim ReplicaSet, ist es auch beim Deployment möglich die Anzahl der laufenden Pods festzulegen

```bash
kubectl scale deployment my-simple-deployment --replicas=5
```


## Cleanup

```bash
kubectl delete -f yaml/deployment.yaml
```
