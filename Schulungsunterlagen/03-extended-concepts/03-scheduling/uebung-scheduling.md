[← Zurück zur Übersicht](../../../README.md)

# Kubernetes Scheduling

In dem Bereich Scheduling wollen wir uns etwas genauer anschauen, wie Kubernetes mit den Pods umgeht und wie diese verteilt werden.

## 1. Manuelle Verteilung (hard coded)

Mit diesem einfachen Pod-Manifest möchten wir das manuelle Scheduling demonstrieren.

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodename-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: NODENAME
````

Erstellen Sie die YAML-Datei und passen Sie entsprechend dem Cluster die ``nodeName``-Einstellung an.
Senden Sie es mit ``kubectl apply -f ...`` an den API Server.

Durch die Angabe von ``nodeName`` im Manifest wird der Scheduling - Prozess sozusagen ausser Kraft gesetzt.
Der Pod wird nur auf den angegebenen Node verteilt.
Sollte da kein Platz mehr sein, wird der Pod als "pending" gesetzt.
- Was passiert wenn man anstelle der Worker-Nodes die ControlPlane angibt?


## 2. NodeSelector

Anstelle des ``nodeName`` Attributs wird das ``nodeSelector`` Attribut verwendet.

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodeselector-pod
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    size: small
````

Der Scheduler filtert anhand des Labels die passenden Knoten heraus, welche für den Pod infrage kommen.
Existiert kein Knoten mit dem Label, bleibt der Pod im *pending*-Status.
Mit ``kubectl lable ...``  können wir ein Label auf den Knoten anbringen.

````bash
kubectl label nodes $NODE_NAME size=small
````

prüfen, ob korrekt:
````bash
kubectl get nodes --show-labels
````

> **Hinweis:** Zum Löschen von Labels wird ein lediglich ein - (Hyphen) hinter den Label-Name gestellt. Für das gezeigte Beispiels sieht dies so aus ``kubectl label nodes $NODE_NAME size-``

Der Scheduler sucht entsprechend dem Label den passenden Knoten für den Pod.
- Was passiert wenn man nur die ControlPlane wieder labelt?

## 3. Taints & Tolerations

In Kubernetes dienen *Taints* und *Tolerations* dazu, die Platzierung von Pods auf Nodes gezielt zu steuern.
Mit *Taints* können bestimmte Nodes „markiert“ werden, sodass nur Pods mit passenden *Tolerations* darauf ausgeführt werden dürfen.
Dieses Konzept hilft, spezielle Ressourcen zu reservieren, sensible Workloads zu isolieren oder ungewollte Platzierungen zu vermeiden.

Erstelle ein einfaches nginx-Deployment mit vier Replicas ``kubectl create deployment nginx --image=nginx``.
- Was kann man feststellen?

Welches Taint ist auf *cp-01* gesetzt und was ist entsprechend in den Pods, Deployments angegeben damit diese auf cp-01 ausgeführt werden?

> **Hinweis:** \
> Es gibt eine Vielzahl an vordefinierten Labels,Taints und Annotationen ([Kubernetes.io - Well-Known Labels, Annotations and Taints][kubernetesio-ref-labels-annotations-taints])

Möchte man die Knoten seines Clusters entsprechend mit Taints gestalten, kann man dies ``kubectl taint ...`` durchführen.
Machen Sie den Knoten ``w-02`` nur für das ``team=dev:... `` planbar, bestehender workload sollte nicht beeinträchtigt werden (``NoSchedule``).
- Was ist nach dem Anbringen von dem taint bei den nginx-Pods zu beobachten
Skalieren Sie dann nochmals das nginx-Deployment erst auf 0 und dann nochmals auf 4.
- Was kann man feststellen?

Erstellen Sie dann im zweiten Schritt einen neues Deployment ``httpd`` mit dem image ``httpd`` und fügen sie im zweiten Schritt mit ``kubectl edit ... `` (oder gerne auch mit ``kubectl patch ...``) die folgenden Zeilen entsprechend in die Pod-Spec des Deployments ein.

```yaml
# ...
      tolerations:
      - key: "team"
        operator: "Equal"
        value: "dev"
        effect: "NoSchedule"
# ...
```

Erhöhe Schritt für Schritt die Anzahl der Replicas von dem *httpd*-Deployment.
- Was kann man beobachten?

Als letztes wollen wir noch den Knoten w-01 nur für Produktiv-Workload (``team=prod``)einrichten, bestehende Pods sollen dabei verdrängt werden (``NoExecute``).
- Wie ist vorzugehen bzw. welchen Effekt müssen wir setzen?
- Was passiert beim anwenden?
- Welche der Systemkomponente des Clusters könnte dieses Taint betreffen (kube-apiserver, kube-scheduler, kube-proxy, ...) und hatte es Auswirkungen?

### Cleanup

```bash
kubectl delete deployments httpd
kubectl delete deployments nginx
kubectl taint node w-01 team-
kubectl taint node w-02 team-
```

> **WICHTIG:** Nach dem Beenden der Übung die Taints auf den Knoten wieder entfernen mit ``kubectl taint node $NODE team-``.


## 4. Affinity

### 4.1 Node Affinity

Erstelle folgenden Pod als Beispiel für eine node affinity:

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: required-affinity-pod
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: region
            operator: In
            values:
            - german-south
````

Wie bereits zuvor mit dem NodeSelector Beispiel muss der Knoten gelabelt sein, damit das funktioniert.

````bash
kubectl label nodes $NODE_NAME region=german-south
````

Mit ``kubectl apply -f ...`` das erstellte Pod-Manifest an den API Server schicken.

### 4.2 nice to have node affinity

Der Scheduler wird versuchen, den Pod auf einem Knoten zu platzieren, der die Kriterien erfüllt.
Wenn jedoch keine solchen Knoten verfügbar sind, kann der Pod auch auf einem anderen Knoten eingeplant werden, der die Präferenz nicht erfüllt.

Erstelle folgendes Beispiel dafür:

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: preferred-affinity-pod
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: region
            operator: In
            values:
            - german-north
````

Mit ``kubectl apply -f ...`` an den API Server schicken.

In diesem Beispiel findet er keinen node mit dem label, aber der pod wird trotzdem verteilt.


### 4.3 Node Anti Affinity

Wie der Name schon sagt, ist das das Gegenteil von Node Affinity.
Sie stellt sicher, dass ein Pod nicht auf bestimmten Knoten basierend auf Knotenkennzeichnungen eingeplant wird.
Dies ist besonders nützlich, wenn Sie vermeiden möchten, dass bestimmte Arbeitslasten auf bestimmten Knoten platziert werden, z.B. um hochverfügbare Anwendungen von Knoten mit weniger zuverlässiger Hardware fernzuhalten.


### Cleanup

```bash
kubectl delete pod required-affinity-pod
kubectl delete pod preferred-affinity-pod
```

## Referenzen
- [kubernetes.io - Concept: Taint and Toleration ][kubernetesio-concepts-taint-and-toleration]
- [kubernetes.io - Well-Known Labels, Annotations and Taints ][kubernetesio-ref-labels-annotations-taints]


[kubernetesio-concepts-taint-and-toleration]: https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
[kubernetesio-ref-labels-annotations-taints]: https://kubernetes.io/docs/reference/labels-annotations-taints/
