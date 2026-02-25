[← Zurück zur Übersicht](../../../README.md)

# ConfigMaps und Secrets

In dieser Übung wollen wir uns mit ConfigMaps und Secrets vertraut machen.

## 1. ConfigMap

ConfigMaps erlauben eine Trennung zwischen Konfiguration und Anwendung, die in einem Container zu finden ist.
In der Regel wird die Konfiguration einer Anwendung durch Dateien oder Umgebungsvariablen realisiert.
Diese Daten kann man in ein ConfigMap-Manifest ablegen und entsprechend in ein Kubernetes-Cluster bekannt machen.
Beim Ausrollen der Anwendung, referenziert man die bekannte ConfigMap und injiziert die Werte entsprechend in den Container.

### 1.1 Umgebungsvariablen mit ConfigMap

In ConfigMaps sind oft Einstellungen definiert, welche in Form von Umgebungsvariablen dem Container bzw. der Anwendung übergeben werden.
Erstellen Sie eine einfache `.env`-Datei, welche ein paar Umgebungsvariablen definiert, beispielsweise:

````sh
cat << EOF > env-file
APP_VERSION="v3.2.1"
APP_NAME="Super App"
EOF
````

Erzeugen Sie damit eine neue ConfigMap ``nginx-env``, nutzen Sie den ``--from-env-file`` Schalter und sehen Sie sich das Ergebnis an:

````sh
kubectl create configmap nginx-env --from-env-file env-file
kubectl get configmap nginx-env -o yaml
````

Machen See sich eine Kopie des Nginx-Manifests und fügen sie die folgenden Modifikationen ein. (``cp yaml/nginx.yaml yaml/nginx-env.yaml``)

````yaml
...
# NOTE: INCOMPLETE YAML
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        # add envFrom config to container
        envFrom:
        - configMapRef:
            name: nginx-env
...
````

Rollen Sie das nginx-env.yaml Manifest (``kubectl apply -f ...``) aus und überprüfen Sie, ob die Einstellung im Container gesetzt sind:

````sh
# TODO: set NGINX_POD
kubectl exec -it $NGINX_POD -- env | grep ^APP
````

Als zweite Möglichkeit kann man auch einzelne Umgebungsvariablen setzen und auf den Wert einer Variable aus einer ConfigMap zurückgreifen.

Zuerst möchten wir in diesem kleinen Test überprüfen, welche Priorität welche Konfigurationen haben.
Wir möchten also für unseren fall feststellen, ob ``APP_VERSION`` von der ConfigMap genommen wird oder die Variable die extra gesetzt wird.
Hierzu definieren wir im Manifest eine separate Variable
``APP_VERSION`` und geben dieser einen neuen Wert.

````yaml
...
# NOTE: INCOMPLETE YAML
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        # add env config to container
        env:
        - name: APP_VERSION
          value: "v5.4.3"
...
````

Schauen Sie sich die Umgebungsvariablen an, welcher Wert wurde im Container gesetzt?

Als Letztes wollen wir einen spezifische Wert für eine Variable aus einer ConfigMap beziehen.
Kommentieren/Entfernen Sie die vorherigen Änderungen in _nginx-env_ Manifest und fügen sie den *env*-Abschnitt wie folgt ein:

````yaml
...
# NOTE: INCOMPLETE YAML
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        # change env config to container
        env:
        - name: APP_VERSION
          valueFrom:
            configMapKeyRef:
              key: APP_VERSION
              name: nginx-env
...
````

Welche *App*-Umgebungsvariable sind jetzt gesetzt?

````sh
# TODO: set NGINX_POD
kubectl exec -it $NGINX_POD -- env | grep ^APP
````

> **Hinweis:** Es können auch andere Quellen als Referenz angegeben werden, sogar Werte aus dem aktuellen Manifest.
> Für weitere Informationen hierzu siehe [Kubernetes.io - Inject Data into Application](https://kubernetes.io/docs/tasks/inject-data-application/) oder ``kubeclt explain pod.spec.containers.env.valueFrom``


### 1.1 ConfigMap als Volume

ConfigMaps können auch als Volumes in ein Pod bzw. Container eingebunden werden.
Was es damit auf sich hat werden wir in dieser Übung ansehen.

Wir erstellen zunächst eine neue ConfigMap mit den Daten, welche die index.html Datei des Nginx ersetzen sollen.

````sh
kubectl create configmap nginx-configmap --from-literal index.html="Hello World"
kubectl get configmap nginx-configmap -o yaml
````

Was können Sie feststellen im Vergleich zur ConfigMap ``nginx-env`` aus der vorhergehenden Aufgabe?

Als Nächstes muss die ConfigMap im Pod bzw. im Container eingebunden werden.
Machen Sie eine Kopie der bereitgestellten Manifest-Datei (``cp yaml/nginx.yaml yaml/nginx-volume.yaml``) und fügen sie die nachfolgenden Änderungen ein.
Damit die Daten in ein Volume geladen werden, benötigt man einen _volumes_-Abschnitt der die ConfigMap referenziert:

````yaml
...
# NOTE: INCOMPLETE YAML
spec:
  template:
    spec:
      volumes:
      - name: nginx-configmap
        configMap:
          name: nginx-configmap
      containers:
...
````

> **Hinweis**: Der _volumes_-Abschnitt gilt für alle Container des Manifestes und befindet daher auf der gleichen Ebene (Einrückung) wie _containers_.
> Für mehr Informationen darüber verwenden Sie ``kubectl explain ... ``

Als nächstes müssen die Daten aus der ConfigMap im Ziel-Container referenziert werden; dies geschieht wie folgt und ist in dem nginx Container einzutragen:

````yaml
...
# NOTE: INCOMPLETE YAML
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        # new volumesMount config
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: nginx-configmap
...
````

Ist das Manifest korrekt und konnte ausgerollt werden, kann mittels ``curl`` eine Anfrage auf den Pod (Pod-IP oder `kubectl port-forward ...`) gesendet werden.
Ändern Sie nochmals die nginx-index.html Daten aus der ConfigMap. Was können Sie bei einem erneuten ``curl`` feststellen?

### Cleanup

```bash
kubectl delete deployments.apps nginx
kubectl delete cm nginx-configmap nginx-env
```

## 2. Secrets

[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) sind zu ConfigMaps sehr ähnlich.
Sie erlauben eine Separation zwischen Konfigurationsdaten oder Skripten, welche man welche man in einer ConfigMap hält.
Zugangsdaten, Zugangsschlüssel und Ähnliches sind besser in Secrets abgelegt.

> **Hinweis:** Secrets sind nicht, wie wie ihr Name vermutet, automatisch verschlüsselte Daten.
> Die Daten welche in ein Secret einfließen, werden lediglich als mit Base64-Codierte abgelegt.
> Hier müssen weitere Bemühungen angestrebt werden wie ein [verschlüsselter etcd-Speicher](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) und eine genaue Zugriffskontrolle auf Secrets und Pods, welche Secrets verwenden.

Kubernetes bietet im Standard 3 verschiedene Typen von Secrets an.
Der Typ ``docker-registry`` hält die Informationen, um Container-Images herunterzuladen, welche durch eine Zugangsinformation gesichert sind, beispielsweise ein privates Image auf DockerHub.
Mit dem Typ ``tls`` kann man den Key und das Zertifikat, welche für eine TLS-Verbindung essenziell  sind, ablegen.
Mit dem letzten Typ ``generic`` können wir, analog zu einer ConfigMap, beliebige Daten durch Secret speichern .

Im Folgenden wollen wir unseren Nginx mit einem selbst signierten Zertifikat absichern. Hierzu legen wir Key und Zertifikat an, eine angepasste nginx-config und legen die Dateien als ConfigMap und Secret ab. Danach muss das Deployment-Manifest modifiziert werden, um beide Objekte zu verwenden.

Durch das folgende Skript erstellen wir unsere TLS-Key und TLS-Zertifikat:

````sh
KEY_FILE="tls.key"
CERT_FILE="tls.crt"
CERT_DAYS=365

openssl genrsa -out $KEY_FILE 2048

# create CSR (Certificate Signing Request)
openssl req -new -key $KEY_FILE -out tls.csr -subj "/CN=example.com"

# create self-signed certificate
openssl x509 -req -days $CERT_DAYS -in tls.csr -signkey $KEY_FILE -out $CERT_FILE

# cleanup
rm tls.csr
````

Als nächstes müssen wir eine nginx-config bereitstellen, welche für TLS konfiguriert ist und Key und Zertifikat referenziert:

````sh
cat << EOF >> tls-default.conf
server {
    listen 443 ssl;
    server_name example.com;

    # Pfade zu deinem selbstsignierten Zertifikat und Schlüssel
    ssl_certificate /etc/nginx/ssl/example.com/tls.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com/tls.key;

    # SSL-Protokolle und -Ciphers
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
EOF
````

Als nächstes müssen wir beides als Secret und ConfigMap in Kubernetes bereitstellen:

````sh
kubectl create secret tls nginx-tls --key tls.key --cert tls.crt
kubectl create configmap nginx-tls-config --from-file tls-default.conf
````

Im weiteren folgen die Anpassungen für das Deployment, damit beides von dem Container verwendet wird.
Da beide Objekte, Secret und ConfigMap, Dateien beinhalten, sind entsprechende Volumes im Manifest zu definieren.
Machen Sie sich wieder eine Kopie der nginx-Vorlage und passen diese ab (``cp yaml/nginx.yaml yaml/nginx-secret.yaml``).

````yaml
...
# NOTE: INCOMPLETE YAML
spec:
  template:
    spec:
      # add volumes config
      volumes:
      - name: nginx-tls-config
        configMap:
          name: nginx-tls-config
      - name: nginx-tls
        secret:
          secretName: nginx-tls
      container:
...
````

Danach noch die Dateien aus den jeweiligen Objekten auf ihre Pfade mappen:

````yaml
...
# NOTE: INCOMPLETE YAML
spec:
  template:
    spec:
      container:
      - name: nginx
        image: nginx:latest
        # insert volumeMounts config
        volumeMounts:
        - mountPath: /etc/nginx/conf.d/tls-default.conf
          name: nginx-tls-config
          subPath: tls-default.conf
        - mountPath: /etc/nginx/ssl/example.com/
          name: nginx-tls
...
````

Abschließend das modifizierte Deployment anwenden (``kubectl apply -f ...``) und mit ``curl ...`` den Pod ansprechen.
Den curl-Request entsprechend mit der Pod-IP vom Cluster aus machen oder mit `kubectl port-forward ...` ein *Port-Forwarding* direkt auf den Pod einrichten.

Als Beispiel entsprechend ``curl --insecure https://${POD_IP}:443`` mit der POD_IP den Nginx ansprechen.
Die Option ``--insecure`` muss hierbei gesetzt werden, da ja ein selbst signiertes Zertifikat verwendet wird.

### Fragen

- Warum wird ``subPath`` im Volume Mount der ConfigMap verwendet?
  - Was passiert, wenn man ihn weg lässt?
  - Wie könnte man die ConfigMap noch mounten?

### Cleanup

```bash
kubectl delete deployments.apps nginx
kubectl delete configmap nginx-tls-config
kubectl delete secrets nginx-tls
```
