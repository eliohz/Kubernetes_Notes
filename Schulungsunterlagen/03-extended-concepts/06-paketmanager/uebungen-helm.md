[← Zurück zur Übersicht](../../../README.md)

# Übungen Helm

In diesem Tutorial richten wir Helm 3 ein und verwenden es zum Installieren, Neukonfigurieren, Zurücksetzen und Löschen einer Instanz [der Kubernetes Dashboard-Anwendung](https://github.com/kubernetes/dashboard).
Das Dashboard ist eine offizielle webbasierte Kubernetes-Oberfläche.


## Voraussetzungen

Für dieses Tutorial benötigen Sie Folgendes:

- Einen Kubernetes-Cluster mit aktivierter rollenbasierter Zugriffskontrolle (RBAC).
- Das Befehlszeilentool kubectl, für das eine Verbindung mit Ihrem Cluster konfiguriert ist.

Sie können Ihre Verbindung mit dem folgenden Befehl testen:

```sh
kubectl cluster-info
```

## 1. Installieren von Helm-CLI

Installieren Sie zunächst das helm-Befehlszeilendienstprogramm auf Ihrem lokalen Computer. Helm bietet ein Skript, das den Installationsprozess unter MacOS, Windows oder Linux verwaltet.

> **Hinweis:** Auf dem Admin-Knoten ist bereits helm installiert, dort kann man die Installation überspringen.
> Arbeiten Sie auf der ControlPlane können Sie wie gezeigt Helm einrichten.

Wechseln Sie in ein beschreibbares Verzeichnis und laden Sie das Skript aus dem GitHub-Repository von Helm herunter:

```sh
cd /tmp
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
```

Machen Sie das Skript mit ``chmod`` ausführbar:

```sh
chmod u+x get_helm.sh
```

Sie können Ihren bevorzugten Texteditor verwenden, um das Skript zu öffnen und zu überprüfen, ob es sicher ist.
Wenn Sie zufrieden damit sind, führen Sie es aus:

```sh
./get_helm.sh
```

Möglicherweise werden Sie zur Eingabe Ihres Passworts aufgefordert. Geben Sie es ein und drücken Sie ENTER, um fortzufahren.

Die Ausgabe sieht in etwa folgendermaßen aus:
```sh
Output
Downloading https://get.helm.sh/helm-v3.1.2-linux-amd64.tar.gz
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
```

Nachdem Helm nun installiert ist, können Sie Helm zur Installation Ihres ersten Chart verwenden.

Mit dem folgenden Befehl richten wir uns noch die Komfortfunktion der Autovervollständigung in der Bash ein:
```bash
helm completion bash | sudo tee /etc/bash_completion.d/helm
```
In einer neuen Shell sollte mit ``helm`` und *tab* mögliche Eingaben angezeigt und entsprechend vervollständigt werden.

## 2. Helm Charts

Helm-Softwarepakete werden als *Charts* bezeichnet.
Damit man Pakete in Helm installieren kann, muss das Repository bekanntgemacht werden.
Das Repo wird meistens bei der Installationsanleitung von Software mitangegeben, so auch für das [Kubernetes Dashboard](https://github.com/kubernetes/dashboard) welches wir in den nächsten Schritten installieren werden.

Fügen Sie das Repository stable hinzu, indem Sie Folgendes ausführen:

```sh
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
```

Der Output sieht wie folgt aus:

```sh
"kubernetes-dashboard" has been added to your repositories
```

**Repositories anzeigen:** \
Mit ``helm repo list`` erhalten Sie eine Übersicht der bekanntgemachten Repositories.

**Anwendungen suchen:** \
Möchten Sie nach einer Anwendung in einem Repo suchen, kann mit  ``helm search repo [keyword]`` danach gesucht werden.
Die Angabe von ``[keyword]`` ist dabei optional und zeigt Anwendungen aus allen bekannten Repos mit der aktuellsten Version.
Haben Sie eine bestimmte Anwendung gefunden, möchten jedoch eine bestimmte Version nutzten.
Mit der Option ``--versions`` ist es möglich, sich alle verfügbaren Möglichkeiten anzeigen zu lassen.
Mit ``--version string`` kann ein String mitgegeben werden, der die Version bzw. Version-Range bestimmt.

**Chart pullen:** \
Haben Sie die gesuchte Anwendung mit der gewünschten Version gefunden, können Sie dieses Chart mit einem  ``helm pull`` herunterladen und vor der Installation nochmals inspizieren.
Die Version geben Sie mit der ``--version`` Option an.
Helm lädt das Paket herunter und speichert das Archiv mit Anwendungsname und Version ab.

**Charts erstellen** \
Helm hilft auch bei der Erstellung und Verwaltung eigener Charts.
Mit ``helm create chart-name`` erstellt man sich die Grundlage eines Helm-Charts mit der notwendigen Struktur und ersten Dateien als Vorlage.
Viele der verfügbaren Kommandos, die Helm definiert, helfen bei der weiteren Verwaltung der Charts.


## 3. Installieren eines Helm Charts

Als Nächstes möchten wir die Dashboard-Anwendung aus dem zuvor hinzugefügten Repository installieren.
Dazu geben wir in unserem ``helm install`` einen *Name* an und spezifizieren entsprechend das Chart.

```sh
helm install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard
```

Die Ausgabe sieht ungefähr so aus:
```sh
NAME: dashboard-demo
LAST DEPLOYED: Tue Apr 20 15:04:19 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
...
```

Beachten Sie die Zeile ``NAME``, die in der obigen Beispielausgabe hervorgehoben ist.
In diesem Fall haben Sie den Namen ``dashboard-demo`` angegeben.
Das ist der *Release*-Name dieser installierten Anwendung.
Unter einem *Release*-Name wird die Anwendung eines Charts mit einer bestimmten Konfiguration bereitgestellt.
Eine Anwendung/Chart kann daher mehrmals durch unterschiedliche Namen und in unterschiedlichen Namespaces deployt und mit jeweils unterschiedlichen Einstellungen konfiguriert sein.
Beispielsweise eine Anwendung/Chart für unterschiedliche Teams (Frontend, Backend) oder für unterschiedliche Stages (Test, QA, Produktion), jedes mit seiner spezifischen Konfiguration.

Installierte Releases im Cluster kann man wie folgt anzeigen lassen:
```sh
helm list
```
HELM arbeitet ähnlich wie kubectl und man kann mit der Option ``-A`` alle Namespaces bzw. ``-n namespace`` spezifische Namespaces anzeigen lassen.

Sie können nun`` kubectl`` verwenden, um sich zu vergewissern, dass mehrere neue Dienste im Cluster bereitgestellt wurden:
````sh
kubectl get services
````

Die Ausgabe sieht in etwa folgendermaßen aus:
````
NAME                                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes                             ClusterIP   10.96.0.1       <none>        443/TCP    176d
kubernetes-dashboard-api               ClusterIP   10.108.61.189   <none>        8000/TCP   26m
kubernetes-dashboard-auth              ClusterIP   10.105.209.99   <none>        8000/TCP   26m
kubernetes-dashboard-kong-proxy        ClusterIP   10.99.202.140   <none>        443/TCP    26m
kubernetes-dashboard-metrics-scraper   ClusterIP   10.106.133.77   <none>        8000/TCP   26m
kubernetes-dashboard-web               ClusterIP   10.102.23.1     <none>        8000/TCP   26m
````

Nachdem Sie nun die Anwendung bereitgestellt haben, verwenden Sie Helm, um dessen Konfiguration zu ändern und die Bereitstellung zu aktualisieren.


## 4. Aktualisieren eines Release

Der Befehl ``helm upgrade`` kann dazu genutzt werden, ein Release mit einem neuen oder aktualisierten Chart zu aktualisieren oder deren Konfigurationsoptionen (Variablen) zu aktualisieren.

Sie werden eine einfache Änderung an dem installierten Release ``kubernetes-dashboard`` vornehmen, um den Aktualisierungs- und Rollback-Prozess zu testen: Ändern Sie den Namen des Dashboard-Dienstes in ``my-kubernetes-dashboard``.

Das Chart *kubernetes-dashboard* bietet eine Konfigurationsoption namens ``fullnameOverride``, um den Dienstnamen zu kontrollieren. Um das Release umzubenennen, führen Sie helm upgrade mit diesem Optionssatz aus:

```sh
helm upgrade kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --reuse-values \
--set fullnameOverride="my-kubernetes-dashboard" \
--set kong.fullnameOverride="my-kubernetes-dashboard-kong"
```

Durch Übergeben des Arguments ``--reuse-values`` stellen Sie sicher, dass die zuvor eingerichteten Chart-Variablen bei der Aktualisierung nicht zurückgesetzt werden.

Sie sehen eine Ausgabe, die dem anfänglichen Schritt ``helm install`` ähnelt.

Überprüfen Sie, ob Ihre Kubernetes-Dienste die aktualisierten Werte widerspiegeln:

````sh
kubectl get services
````

Die Ausgabe sieht ungefähr wie folgt aus:

````sh
NAME                                      TYPE        CLUSTER-IP        EXTERNAL-IP   PORT(S)    AGE
kubernetes                                ClusterIP   192.168.128.1     <none>        443/TCP    30h
my-kubernetes-dashboard-api               ClusterIP   192.168.181.116   <none>        8000/TCP   43s
my-kubernetes-dashboard-auth              ClusterIP   192.168.178.141   <none>        8000/TCP   43s
my-kubernetes-dashboard-kong-proxy        ClusterIP   192.168.144.204   <none>        443/TCP    43s
my-kubernetes-dashboard-metrics-scraper   ClusterIP   192.168.152.177   <none>        8000/TCP   43s
my-kubernetes-dashboard-web               ClusterIP   192.168.148.23    <none>        8000/TCP   43s
````

Beachten Sie, dass die Namen der Service auf den neuen Wert aktualisiert wurde.

### 4.1. Zugriff auf das Dashboard

Zu diesem Zeitpunkt möchten Sie vielleicht das Kubernetes Dashboard tatsächlich in Ihren Browser laden und überprüfen.
In unserem Cluster Setup hat die ControlPlane ebenfalls eine öffentliche IP erhalten.
Damit wir das Dashboard erreichen benötigen wir lediglich ein NodePort-Service:

````sh
kubectl expose deployment my-kubernetes-dashboard-kong --type NodePort
````

> **Anmerkung:** Je nach Konfiguration erlaubt das Dashboard einen schreibenden Zugriff auf den Cluster und sollte daher nicht unbedacht öffentlich bereitstehen.

Mit
````sh
https://${CP_01_PUBLIC_IP}:${DAHSBOARD_NODEPORT}
````

Da der Zugriff auf das Dashboard durch RBAC gesichert ist, sehen Sie beim Zugriff nur eine Anmeldemaske.
Dies ist jedoch nicht wie gewohnt ein regulärer Login, sondern es wird ein Token verlangt, der auf einen Service-Account zurückzuführen ist.
Wie gezeigt kann jetzt mittels ``kubectl create token <SERVICE_ACCOUNT>`` ein Token generiert werden.
Nutzten Sie hierzu einen der erstellen Service-Accounts wie folgt:

````sh
kubectl create token kubernetes-dashboard-web
````

Kopieren Sie den generierte Token und loggen Sie sich in dem Dashboard ein.

Sie werden feststellen, dass keine Ressourcen angezeigt werden.
- Warum ist dies wohl so und was ist zu tun, um dies zu ändern?

Anweisungen zur tatsächlichen Verwendung des Dashboards sind nicht Bestandteil dieses Tutorials.
Weitere Informationen finden Sie jedoch in den offiziellen [Dokumentation zum Kubernetes-Dashboard](https://github.com/kubernetes/dashboard/tree/master).

Als Nächstes sehen Sie sich die Möglichkeit von Helm an, Versionen zurückzusetzen und zu löschen.

## 5. Zurücksetzen und Löschen einer Version

Wenn Sie die Version ``kubernetes-dashboard`` im vorherigen Schritt aktualisiert haben, haben Sie eine zweite *Revision* der Version erstellt. Helm behält alle Details der vorherigen Versionen bei, falls Sie eine frühere Konfiguration oder ein früheres Chart wiederherstellen müssen.

Verwenden Sie ``helm list``, um die Version erneut zu prüfen.

Sie sehen die folgende Ausgabe:

````sh
NAME                    NAMESPACE       REVISION        UPDATED                                         STATUS          CHART                           APP VERSION
kubernetes-dashboard    default         2               2025-10-17 09:21:31.601723067 +0200 CEST        deployed        kubernetes-dashboard-7.13.0
````

Die Spalte ``REVISION`` teilt Ihnen mit, dass dies nun die zweite Revision ist.

Verwenden Sie helm rollback, um die erste Revision wiederherzustellen:

````sh
helm rollback kubernetes-dashboard 1
````

Sie sollten die folgende Ausgabe sehen, was bedeutet, dass das Zurücksetzen erfolgreich war:

````sh
Rollback was a success! Happy Helming!
````

Wenn Sie ``kubectl get services`` an dieser Stelle erneut ausführen, werden Sie feststellen, dass der Dienstname wieder seinen vorherigen Wert aufweist. Helm hat die Anwendung mit der Konfiguration von Revision 1 neu bereitgestellt.

Helm-Versionen können mit dem Befehl ``helm uninstall`` gelöscht werden:

````sh
helm uninstall kubernetes-dashboard
````

Der Output sieht wie folgt aus:

````sh
release "kubernetes-dashboard" uninstalled
````

Vergewissern Sie sich mit ``helm list --all``, dass das Helm-Release gelöscht wurde.

Sie sehen, dass es keine gibt:

````sh
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION
````

Jetzt ist die Version wirklich gelöscht!

In diesem Tutorial haben Sie das helm-Befehlszeilentool installiert und sich mit dem Installieren, Aktualisieren, Zurücksetzen und Löschen von Helm-Charts und -Versionen durch Verwalten des Chart kubernetes-dashboard vertraut gemacht.


## CleanUp

```bash
helm uninstall
kubectl delete svc my-kubernetes-dashboard-kong
# provide own created clusterrolebinding
MY_DASHBOARD_CRB=""
kubectl delete clusterrolebindings.rbac.authorization.k8s.io $MY_DASHBOARD_CRB
```


## Referenzen

- Weitere Informationen zu Helm und Helm-Charts finden Sie in der offiziellen [Helm-Dokumentation](https://docs.helm.sh/).
