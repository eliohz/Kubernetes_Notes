[← Zurück zur Übersicht](../../../README.md)

# Persistenter Speicher

In dieser Übung beschäftigen wir uns mit dem Konzept des persistenten Speichers in Kubernetes.
Während Container in der Regel flüchtig sind – das heißt, ihre Daten beim Neustart oder bei der Skalierung verloren gehen – benötigen viele Anwendungen dauerhaft gespeicherte Daten, etwa für Datenbanken, Logs oder Konfigurationsdateien.

Kubernetes bietet hierfür ein flexibles Speicherkonzept, das auf Persistent Volumes (PVs) und Persistent Volume Claims (PVCs) basiert.
Diese Mechanismen trennen die Bereitstellung des Speichers von dessen Nutzung und ermöglichen eine dynamische, plattformunabhängige Verwaltung von Speicherressourcen.

In dieser Übung lernen Sie:
- den Unterschied zwischen ephemerem und persistentem Speicher in Kubernetes,
- wie man Persistent Volumes und Persistent Volume Claims definiert und nutzt,
- wie Storage Classes die dynamische Bereitstellung von Speicher automatisieren,
- und wie eine Anwendung (z. B. eine Datenbank) mithilfe eines PVCs auf dauerhafte Daten zugreift.

Am Ende dieser Übung werden Sie verstehen, wie Kubernetes Daten zuverlässig speichert – auch wenn Pods neu gestartet oder auf andere Nodes verschoben werden.

## Teil 1: Manuelle Erstellung von PV & PVC

In diesem ersten Abschnitt erstellen wir ein statisches Volume, fordern es mit einem PVC an und binden es in einen Pod ein.

Erstelle Sie ein Verzeichnis auf dem Host-System auf einen der Worker-Nodes (`ssh ...`), sowie eine HTML-Datei für Nginx:
````bash
sudo mkdir /mnt/data
sudo chmod 0777 /mnt/data/
echo "<h1>Hello from persistent storage</h1>" > /mnt/data/index.html
````

Erstelle ein ein PersistentVolume – pv.yaml, passe den HostNamen entsprechend an:
```bash
cat <<EOF > manual-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-pv
  labels:
    example: manual-pv
spec:
  capacity:
    storage: 10Mi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node1
EOF
```

Bevor Sie das PV auf den Cluster anwenden, passen Sie den Eintrag ``nodeAffinity``-Eintrag an, auf welchem die Daten liegen und wo schlussendlich der Pod laufen soll. (Ändere ``node1``)

Wir möchten in diesem Beispiel dafür sorgen, dass unser PersistentVolume erste mit dem PersistentVolumeClaim verbunden wird, wenn es ein entsprechender Pod (*Consumer*) gibt der den Speicher benötigt.
Eine [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) realisiert dieses Verhalten über die Eigenschaft ``volumeBindingMode``, kann jedoch noch deutlich mehr Einstellungen umfassen.
- Welche Modi gibt es noch?
- Was deckt eine ``reclaimPolicy`` ab?

```bash
cat > manual-sc.yaml << EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
  labels:
    example: manual-pv
provisioner: kubernetes.io/no-provisioner # indicates that this StorageClass does not support automatic provisioning
volumeBindingMode: WaitForFirstConsumer
EOF
```

Erstelle das Manifest für die StorageClass und wende es auf dem Cluster an.

Damit ein Pod den PersistentVolume verwenden kann, wird noch der *Claim* benötigt.
Ein PersistentVolumeClaim ist das Verbindungsstück zwischen Pod und Volume und stellt die Anforderungen dar, welche für Anwendung zu erfüllen sind.
Wichtig Eigenschaften sind dabei der ``accessMode`` und entsprechend die Größe des Speichers (``resources.requests.storage``)


```bash
cat > manual-pvc.yaml << EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manual-pvc
  labels:
    example: manual-pv
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 10Mi
EOF
```

Sind alle Ressourcen erstellt können wir schließlich den Pod erstellen, der den Speicher verwendet mit den darin enthaltenen Daten.
```bash
cat <<EOF > manual-volume-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-pv-pod
  labels:
    example: manual-pv
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html
    persistentVolumeClaim:
      claimName: manual-pvc
EOF
```

Testen Sie mit curl die Verfügbarkeit des Webservers und verifizieren Sie die erhaltene Antwort
- Entspricht die Antwort den Erwartungen?


### Cleanup

```bash
kubectl delete pv,pvc,sc,pod -l example=manual-pv
```

## Automatische Bereitstellung über StorageClass

Durch Provisioner kann man das manuelle erstellen von PVs umgehen und von dem entsprechenden Controller erledigen lassen.
Gerade in Cloud Umgebungen kann man so einfach und schnell die jeweiligen Speicherdienste seines Cloud-Providers nutzen.
In dieser Übung wollen wir das Szenario eines solchen Provisioners mit einem NFS-Dienst nachstellen und etwas untersuchen.

Damit wir NFS über CSI verwenden können, benötigen wir:
- Einen funktionierenden NFS-Server (z. B. auf einer VM oder lokal mit Export-Verzeichnis)
- Einen NFS-Provisioner im Cluster installiert
- Entsprechende nfs-client Software/Libs auf den Worker-Knoten.

### NFS Setup

Führe die folgenden Befehle auf dem Admin-Knoten aus, er wird unser NFS-Server.

```bash
# update repo
sudo apt update

# install nfs-server
sudo apt install -y nfs-server

# create the shared directory
sudo mkdir /data
sudo chmod 0777 /data

# define the export and the accessible nodes
cat <<EOF | sudo tee /etc/exports
/data 10.0.0.0/24(rw,no_subtree_check,no_root_squash)
EOF

sudo systemctl enable --now nfs-server

sudo exportfs -ar
```

Diese Befehle installieren den NFS-Server und exportieren /data , auf das der Kubernetes-Cluster zugreifen kann.
Im Falle eines Kubernetes-Clusters mit mehreren Knoten sollten Sie alle Kubernetes-Arbeitsknoten zulassen.

Als letzten benötigen die Worker-Nodes für das NFS-Setup die entsprechenden Client-Pakete:
```bash
sudo apt install -y nfs-common
```

Zum Testen des Setups, wie folgt das geteilte Verzeichnis des NFS auf einem Worker wie folgt mounten.
```bash
sudo mount -t nfs 10.0.0.9:/data /mnt
# create test file
touch file /mnt/file
# check on admin /data directory
ll /data
rm /data/file
# unmount the share on the worker
umount /mnt
```

### NFS im Kubernetes Cluster

Du musst den NFS-Provisioner installieren, um PersistentVolume dynamisch mit StorageClasses bereitzustellen.
Wir verwende den nfs-subdir-external-provisioner, um dies zu erreichen.
Die folgenden Befehle installieren alles, was wir brauchen, mit dem Helm-Paketmanager.

```bash
# set ip of nfs server
NFS_SERVER='10.0.0.9'
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --create-namespace \
  --namespace nfs-provisioner \
  --set nfs.server=${NFS_SERVER} \
  --set nfs.path=/data
```

> **Helm** ist ein Paketmanagement-Werkzeug für Kubernetes.
> In einem späteren Kapitel gehen wir näher darauf ein.

Prüfen Sie was uns die Installation des ``nfs-subdir-external-provisioner`` im Cluster erstellt.
- Welche Standard Ressourcen gibt es im ``nfs-provisioner`` Namespace und was sind die groben Eigenschaften?
- Welche neue StorageClass gibt es jetzt und was sind dessen Eigenschaften?


Damit wir Speicher einer Anwendung zuweisen können benötigen wir erneut eine *PersistentVolumeClaim*.
Im Manifest referenzieren wir als `storageClassName` die gerade erstellte StorageClass.

```bash
cat <<EOF > nfs-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-client
EOF
```

Wenden Sie das PVC auf den Cluster mit ``kubectl apply ...`` an und überprüfen Sie dessen Status.
- Was genau ist der Status?
- Wurde ein PV angelegt?
- Was befindet sich in dem geteilten Verzeichnis des NFS-Share?


Als letzten benötigen wir noch eine Anwendung die unseren neuen Speicher nutzt.
Beachten Sie dabei die Einstellungen im Abschnitt ``volumes`` und ``volumeMounts`` des Manifests.

```bash
cat <<EOF > nfs-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-nfs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-nfs
  template:
    metadata:
      labels:
        app: nginx-nfs
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: nfs-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nfs-volume
        persistentVolumeClaim:
          claimName: nfs-pvc
EOF
```

Erstellen Sie das Manifest und wenden Sie auf den Cluster an.
- Startet der Pod bzw. Container?
- Erstellen Sie ein port-forward oder ein Service (`kubectl expose`) auf den Pod und führen Sie ein curl aus, warum tritt ein Fehler auf?

> **Hinweis:** Durch die angegeben Einstellung unter ``volumeMounts`` kann man bestehende Verzeichnisse und Dateien überlagern.

Damit unser Nginx-Webserver wieder eine Antwort liefert, müssen wir eine *index.html* in dem Verzeichnis, welche das PersistentVolume spiegelt, erstellen.
Ersetzten Sie den Platzhalter ``<PV_DIR>``

```bash
echo '<h1>Hello from NFS!</h1>' > /data/<PV_DIR>/index.html
```

Testen Sie einen erneuten curl auf den Webserver.



## Referenzen

- [Kubernetes Local Persistent Volumes](https://vocon-it.com/2018/12/20/kubernetes-local-persistent-volumes/)
