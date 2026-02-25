[← Zurück zur Übersicht](../../../README.md)

# Übung Requests & Limits


## Ziel der Übung

Du lernst:

* Wie Requests und Limits das Scheduling beeinflussen
* Was passiert, wenn Container mehr Ressourcen verbrauchen
* Wie sich unterschiedliche Einstellungen auf die Performance und Stabilität auswirken
* Wie Kubernetes Pods bei Ressourcenknappheit behandelt

## Vorbereitung

Kubernetes bezieht sich bei einigen Funktionen auf die aktuellen Metriken der einzelnen Nodes im Cluster, den Verbrauch von CPU oder RAM.
Damit diese gesammelt und entsprechend bereitstehen, ist eine zusätzliche Komponente Voraussetzung, welche nicht im Standard installiert ist.
Der *Metics-Server* ist diese Komponente und wird nachfolgend dem Cluster hinzugefügt.

### Metric Server

Was ist der Kubernetes Metrics Server?

Der Kubernetes Metrics Server ist ein schlanker, clusterweiter Aggregator für Ressourcenverbrauchsdaten.
Er sammelt Echtzeit-CPU- und Speicher-Metriken von allen Knoten und Pods in einem Cluster und stellt sie über die Metrics-API bereit.
Ein installierter Metrics-Server erlaubt es, den ``kubectl top`` Befehl auf Pods und Knoten auszuführen und entsprechend deren Auslastung anzuzeigen.
In erster Linie dient der Server jedoch für Autoscaling-Funktionen mit *Horizontal-* oder *Vertical-Pod-Autoscaler* als Schnittstelle.

> **Hinweis:** Der Metrics-Server ist **nicht** dafür geeignet als Schnittstelle für Monitoring-Lösungen zu dienen.
> Für dieses Fall sollte direkt die Metriken von den Kubelets und dessen Endpunkt ``/metrics/resource`` gesammelt werden.

Die grundlegenden Ressourcen des Metrics-Servers werden wie gezeigt installiert:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Da wir im Seminar mit self-signed Zertifikaten arbeiten, ist bei dem Deployment des Metric-Servers eine Anpassung nötig.

> **Hinweis:** Dies bitte **NICHT**  unbedacht im produktiven Umfeld machen.

```bash
kubectl -n kube-system patch deployment metrics-server --type='json' \
  -p='[
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/args/-",
      "value": "--kubelet-insecure-tls"
    }
  ]'
```


---

## Erstelle ein Deployment mit definierten Requests und Limits

Es ist folgendes Deployment ``burstable-deployment`` mit Inhalt gegeben: ``yaml/deployment.yaml`` machen Sie sich damit vertraut:
- Welche Anforderungen hat der Pod an CPU und Arbeitsspeicher hinterlegt?
- Mit welchem Label steuern wir, wo der Pod landet?
- Was wird im Container ausgeführt und kann man sagen, wie der Pod ausgelastet sein wird?


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: burstable-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: burstable-deployment
  template:
    metadata:
      labels:
        app: burstable-deployment
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: type
                operator: In
                values:
                - request
      containers:
      - name: stress-container
        image: polinux/stress
        command: ["stress"]
        args: ["--cpu", "2"]
        resources:
          requests:
            cpu: "250m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
```

Füge ein Label zu einem node hinzu:
``` bash
kubectl get nodes --show-labels
```

Wähle dann zB den Node w-01 aus:
```bash
kubectl label nodes w-01 type=request
```

Überprüfen, ob das Label korrekt hinzugefügt wurde:
```bash
kubectl get node w-01 --show-labels
```

Wenden Sie danach das Deployment im Cluster an und prüfe, ob der Pod läuft und welche Ressourcen er verbraucht.
- Mit welchen Sub-Befehlen und Ressourcen kann ich mit ``kubectl`` eine Übersicht über die Ressourcenauslastung erhalten?
- Wie stark sind unsere burstable-deployment Pods ausgelastet in %?


Beobachte, dass der Scheduler die Pods nur auf Nodes setzt, die mindestens **250m CPU und 128Mi RAM** frei haben.

---

## Überlaste den Arbeitsspeicher des Containers

Nun wirst du untersuchen, wie Kubernetes reagiert, wenn der Container versucht, über seine Limits vom Arbeitsspeicher hinauszugehen.

Verbinde dich mit ``kube exec ...`` mit einem der burstable-deployment Pods.

Starte eine RAM-intensive Aufgabe, z. B.:
```bash
stress --vm 1 --vm-bytes 250M --timeout 20
```
In einem zweiten Terminal kann man kann mit ``watch -n1 kubectl top pods`` den Speicherverbrauch überwachen.
Die Aktualisierung von ``kubectl top`` ist jedoch nicht ganz so schnell bis man etwas sieht.

> **Hinweis:** Der ``stress``-Prozess ist etwas seltsam.
> Wenn man den zweiten Prozess (``kubectl exec ...``und und ``stress`` ) mit ``CTRL-c`` beendet, tritt ein Fehler auf was den gesamten Container beendet.
> Daher vermeiden den Prozess mit ``CRTL-c`` zu beenden und füge ein Timeout dem Kommando hinzu (``--timeout 20``).

Die Anzahl der Bytes des ``stress``-Befehls dann entsprechend auf das Limit des Containers erhöhen.
- Was passiert und was war der Status des Pods?


---

## Quality of Service Klassen (QoS)

### CPU

Erstelle zwei weitere Deployment-Manifest und passe die **Namen** sowie **Requests** und **Limits** entsprechend den angaben an:

| Deployment       | Requests            | Limits              | Erwartetes Verhalten                                        |
| ---------------- | ------------------- | ------------------- | ----------------------------------------------------------- |
| **best-effort**  | Keine               | Keine               | Pods können verdrängt werden, wenn Ressourcen knapp sind    |
| **burstable**    | 250m CPU, 128Mi RAM | 500m CPU, 256Mi RAM | moderate Stabilität, CPU-Drosselung möglich                 |
| **guaranteed**   | 250m CPU, 256Mi RAM | 250m CPU, 256Mi RAM | höchste Priorität, garantiert Ressourcen                    |

Überprüfe mit kubectl die QoS-Klasse der Pods, verwende dafür die Ausgabe ``-o custom-columns``.
Die Spalte der Klasse bekommt man durch die Angabe von ``QoS:status.qosClass``

Führe alle drei Deployments gleichzeitig aus, um somit auf dem Knoten ein Last zu simuliere.
Überprüfe das den aktuellen Verbrauch und den Status der Pods:
````bash
kubectl top pods
kubectl get pods -o wide
````

Skaliere danach ein paar der Pods, um den Knoten weiter künstlich zu überlasten und beobachte:
- Wann werden keine Pods mehr gescheduled?
- Werden Pods durch das Skalieren beendet?
  - wenn nein warum nicht?
- Wie variiert die Auslastung von CPU der Pods?

## Reflexionsfragen

1. Was passiert, wenn Requests zu hoch gesetzt werden?
2. Wie beeinflussen Limits die Performance deiner Anwendung?
3. Wann würdest du gleiche Werte für Request und Limit wählen (Guaranteed QoS)?
4. Warum ist es riskant, keine Limits zu setzen?
5. Wie würdest du Requests und Limits für eine Web-App, eine Batch-Job-App und eine Datenbank-App unterschiedlich festlegen?

## CleanUp

```bash
kubectl delete deployment best-effort-deployment burstable-deployment guaranteed-deployment
```
