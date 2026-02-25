[← Zurück zur Übersicht](../../../README.md)

# Pods


In dieser Übung erstellen Sie einen ersten einfachen Pod.
Dazu werden wir das nginx image benutzen, das automatisch von dem docker hub heruntergeladen wird.
Als erstes erstelle ein Verzeichnis, in dem wir unsere Objekte ablegen. Dies dient zur besseren Übersicht / Struktur.


## Vorbereitung

Falls noch nicht geschehen, machen Sie sich mit den folgenden Kommandos einen Überblick über ihre Umgebung.

- ssh zu den verschiedenen Knoten
- uptime
- top/htop/ps
- memory
- disks (blkid/df/lsblk)
- ip
- Linux version (cat /etc/*release - uname -a)
- [Zusätzliche Kommandos](https://www.ionos.de/digitalguide/server/konfiguration/linux-befehle-terminal-kommandos-im-ueberblick/)


Ein Kubernetes Cluster sollte jetzt bereit stehen, mit dem wir interagieren können.

Wenn man wissen möchte, mit welcher Version man aktuell Arbeitet kann man kubectl verwenden:
````sh
kubectl version
kubectl cluster-info
````
Als nächstes prüfen wir zuerst, ob der Cluster soweit läuft bzw. die Knoten alle auf "ready" stehen.
````sh
kubectl get nodes
````
Um zu erfahren, welche Pods eventuell schon im Cluster / Default namespace laufen:
````sh
kubectl get pods
kubectl get pods -A #zeigt alle Pods an, die im Cluster aktuell laufen.
````

## Einfacher Pod

Erstelle nun mit folgender Datei den ersten Pod:
````sh
vi erster-pod.yaml
````

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstpod
  labels:
    app: web
spec:
  containers:
    - name: myfirstcontainer
      image: nginx
      ports:
        - containerPort: 80
```

Hinweis: Beachte die richtige Einrückung (Indentation), da dies bei YAML-Dateien sehr wichtig ist!
Nun wird diese Datei an die Kubernetes API übergeben und der Pod wird erstellt.
Kubernetes sorgt dafür, dass das Image gezogen und auf einem der Knoten der Pod gestartet wird.
````sh
kubectl apply -f erster-pod.yaml
````

Danach check:
````sh
kubectl get pods
kubectl describe pod myfirstpod
````
Jetzt überprüft wir, ob der Pod bzw. die Applikation, die der Container enthält läuft.
````sh
kubectl exec myfirstpod -- service nginx status
````
Wenn alles richtig ist, kommt eine Meldung: "nginx is running"

Wenn der Pod dann nicht mehr benötigt wird, kann man diesen wie folgt löschen:

```sh
kubectl delete pod myfirstpod
# oder
kubectl delete -f erster-pod.yaml
```

## kubectl und Optionen

Anstelle der YAML-Datei kann man auch recht einfach mit ``kubectl`` ein Pod starten:

```bash
kubectl run myfirstpod --image nginx
```

``kubectl`` besitzt eine umfassende Liste an Optionen die wir testen möchten.

```sh
kubectl get pod -o wide
```

- Was wird zusätzlich angezeigt?
- Auf welchem Node läuft die Anwendung?

```sh
kubectl get pods --all-namespaces
```

- In welchem Namespace läuft unser Pod?
- Welche Pods aus welchen Namespaces werden noch angezeigt?

```sh
kubectl get namespaces
```

- gibt Liste aus der vorhandenen Namespaces

```sh
kubectl get pod myfirstpod -o yaml
```

Hinweis zu image – herkunft, name, state, ip…


> **Hinweis:** Auch für troubleshooting ist folgender Befehl ganz gut um Infos über ein Pod zu bekommen.
> ``kubectl describe pod $POD_NAME`` ( –n "namespace" nur, wenn der pod nicht im aktuellen default läuft)


Erklärung der möglichen Einstellungen beim Pod:

```sh
kubectl explain pod
kubectl explain pod.metadata
# komplette Objektdefinition ausgeben
kubectl explain pod --recursive
```

Für Fehlersuche immer gut:
zusätzliches Terminalfenster erstellen und folgenden Befehl:

```sh
kubectl get pod --watch
# kubectl get pod -w
```

Damit kann 'live' beobachtet werden, wenn man zB ein Pod verändert (erneutes apply)

Eine Liste der Pods erstellen, sortiert nach Name:
```sh
kubectl get pods --sort-by=metadata.name
```

### Pod Zugriff

Wir haben in diesem Beispiel eine NGINX-Anwendung im Pod am laufen und können diese ansprechen.
Der Schwerpunkt liegt darauf, den Pod direkt über seine **Pod-IP** sowie über **Port-Forwarding** mit `curl` zu erreichen.
Diese Zugriffsarten ermöglichen im Development eine schnelle und einfache Möglichkeit eine Anwendung zu testen.
Service-Ressource stellen eine weitere Möglichkeit dar, einen Zugriff auf die Anwendung zu

```sh
# get POD_IP
# from cluster-node (controlplane or worker)
curl $POD_IP
```

```sh
# from admin (kubectl installed and proper kubeconfig)
kubectl port-forwarding myfirstpod 8080:80
# new terminal
curl localhost:8080
```

### Pod Logs

Log eines Pods anschauen:
```sh
kubectl logs myfirstpod
```

## Cleanup

```bash
kubectl delete pod myfirstpod
```


## Pod Vorlagen/Beispiele

Hier noch die Vorlage eines busybox pods mit Argumenten zur Ausführung des Containers.

````yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
````
