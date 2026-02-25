[← Zurück zur Übersicht](../../../README.md)

# Gateway API

Die Gateway-API ist der moderne Nachfolger des Kubernetes-Ingress und bietet eine flexiblere und klarer strukturierte Möglichkeit, Netzwerkverkehr in Kubernetes zu steuern.
Im Gegensatz zu Ingress trennt die Gateway-API die Zuständigkeiten zwischen Infrastruktur-Administratoren und Anwendungsteams deutlicher und unterstützt neben HTTP auch weitere Protokolle wie TCP, UDP oder gRPC.
In dieser Übung lernst du, wie du mit einer GatewayClass, einem Gateway und einer HTTPRoute den HTTP-Traffic zu einem Beispielservice weiterleitest.
Ziel ist es, die grundlegende Funktionsweise der Gateway-API zu verstehen und einen einfachen, funktionsfähigen Gateway in Kubernetes bereitzustellen.

## Ziel der Übung

Nach Abschluss dieser Übung kannst du:

- die grundlegenden Komponenten der Gateway-API (GatewayClass, Gateway, HTTPRoute) erklären,
- eine einfache Anwendung über die Gateway-API erreichbar machen,
- und verstehen, wie moderne Traffic-Steuerung in Kubernetes umgesetzt wird.

## Setup Gateway API

Diese Übung setzt voraus, dass die Übung *Service* mit dem Abschnitt *ClusterIP* durchgeführt wurde.

Wie auch beim Ingress ist bei der Gateway-API ein Controller für das Weiterleiten der Requests verantwortlich.
Da diese Ressourcen nicht im Standard enthalten sind, müssen wir diese entsprechend dem Cluster hinzufügen.

Gateway-API-CRDs dem Cluster hinzufügen:
```bash
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v2.1.4" | kubectl apply -f -
```

Den Gateway-Controller mit Helm installieren, in unserem Fall verwenden wir den [nginx-gateway-fabric](https://docs.nginx.com/nginx-gateway-fabric/install/helm/):
```bash
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
--version 2.1.4 \
--create-namespace -n nginx-gateway \
--set nginx.service.type=NodePort \
--set-json 'nginx.service.nodePorts=[{"port":31437,"listenerPort":80}]'
```

> **Helm** ist ein Paketmanagement-Werkzeug für Kubernetes.
> In einem späteren Kapitel gehen wir näher darauf ein.

- Welche Standard-Ressourcen wurden im Namespace `nginx-gateway` erstellt?

Um das eigentliche Gateway auszurollen, wenden wir das bereitgestellte Gateway-Manifest an:
```bash
kubectl apply -f yaml/gateway.yaml
```
- Machen Sie sich mit dem Manifest vertraut. Was wird angelegt?
- Was könnte die Eigenschaft `allowedRoutes` des Gateways definieren?
- Was befindet sich im Namespace *infra*?
- Was für einen Service gibt es und welche Eigenschaften hat dieser?

## Konfiguration der HTTP Route

Im vergangenen Abschnitt haben wir den Gateway-Controller (``nginx-gateway-fabric``) und das eigentliche Gateway deployt.
Aus der Sicht des Cluster-Operators hat man definiert, dass das Gateway im Namespace *infra* läuft.
Als Anwendungsbetreiber wollen wir im nächsten Schritt die Kommunikation mit der freizugebenden Anwendung ermöglichen. Hierzu legen wir ein HTTPRoute-Objekt an.

Machen Sie sich mit dem `yaml/httproute.yaml` Manifest vertraut.
- Was sagt die Einstellung `parentRefs` aus?
- Auf welchen Hostnamen ist die Route festgelegt?
- Wie muss die Anfrage aussehen, damit ein Routing passiert (URL)?
- Wie ist festgelegt, wohin geroutet wird?

Wenden Sie die Route im Cluster an:
```bash
kubectl apply -f yaml/httproute.yaml
```

Wie auch beim Ingress können wir nun eine entsprechende Anfrage an den Cluster schicken, die vom Gateway aufgenommen und an den festgelegten Service weitergeleitet wird.
Wir benötigen somit wieder eine IP eines der Worker und den entsprechenden NodePort.
Der NodePort wurde bei der Installation des Gateways definiert und lautet *31437*, man kann sich aber nochmal vergewissern, indem man den Service im *infra*-Namespace betrachtet.
Bei der Host-IP des Workers gibt es jetzt aufgrund einer Einstellung des Gateway-Service ein "aber".
Aufgrund der Einstellung *externalTrafficPolicy* im Service werden nur Requests an den Node geroutet, auf dem der Pod mit dem Gateway läuft.
Für mehr Informationen siehe nachfolgenden Hinweis.

> **[Hinweis: externalTrafficPolicy](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip)** \
> Die Einstellung externalTrafficPolicy in einem Kubernetes-Service (vom Typ LoadBalancer oder NodePort) legt fest, wie eingehender externer Traffic an die Pods weitergeleitet wird.
> Sie kann zwei Werte annehmen:
>
> *Cluster* (Standard):\
> Der eingehende Traffic kann auf alle Nodes im Cluster verteilt werden, unabhängig davon, wo die Ziel-Pods laufen. Das sorgt für gleichmäßiges Load-Balancing, führt aber dazu, dass die ursprüngliche Client-IP verloren geht.
>
> *Local*:\
> Der Traffic wird nur an Pods auf dem Node weitergeleitet, auf dem die Anfrage angekommen ist. Dadurch bleibt die Client-IP erhalten, aber es kann zu ungleichmäßiger Lastverteilung kommen, wenn nicht auf jedem Node ein passender Pod läuft.

Die IP des entsprechenden Worker-Nodes greifen wir wie folgt ab:
```bash
# only one ip should return
kubectl -n nginx-gateway get pods -l app.kubernetes.io/instance=ngf -o jsonpath='{.items[*].status.hostIP}'
# create variable
HOST_IP=''
```

Mit dem folgenden Aufruf können wir das Routing testen:

```bash
# will result in 404
curl -H "host: example.com" $HOST_IP:31437/api
```

Jedoch wird dieser Aufruf noch mit **404** quittiert!
Aufgrund unseres Setups mit dem Gateway im *infra*-Namespace und unserer Anwendung und HTTPRoute im *Default*-Namespace, bildet dieses Szenario ein [Cross-Namespace Route Attachment](https://gateway-api.sigs.k8s.io/guides/multiple-ns/) und muss daher entsprechend konfiguriert sein.
Aus der Rolle des Cluster-Operators muss für ein funktionierendes Routing im Gateway eine ``allowedRoutes`` Regel definiert sein und erlaubte Namensräume müssen mit dem entsprechenden Label versehen werden.

Das entsprechende Labeling wurde im Setup ausgelassen und ist im Cluster einzufügen.
```bash
# fix missing cross-namespace label for HTTPRoutes
kubectl label namespaces default team=app
```

Ein erneuter Aufruf:
```bash
curl -H "host: example.com" $HOST:31437/api
```

sollte zu einer ähnlichen Ausgabe, wie gezeigt führen:
```
Request served by frontend-745f6f954d-fjgdx

GET /api HTTP/1.1

Host: example.com
Accept: */*
Connection: close
User-Agent: curl/7.61.1
X-Forwarded-For: 10.0.0.9
X-Forwarded-Host: example.com
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Real-Ip: 10.0.0.9
```


## Clean up

```bash
# delete app with httproute
kubectl delete service frontend
kubectl delete deployments.apps frontend
kubectl delete httproutes.gateway.networking.k8s.io frontend
# delete gateway
kubectl label namespaces default team-
kubectl delete -n infra gateway web
kubectl delete namespaces infra
```

# Referenzen

- [codecentric: Demystifying-the-Kubernetes-Gateway-API](https://www.codecentric.de/en/knowledge-hub/blog/demystifying-the-kubernetes-gateway-api-what-the-heck-is-it-and-why-should-you-care)
- [gateway api](https://gateway-api.sigs.k8s.io/)
- [nginx-gateway-fabric Get started](https://docs.nginx.com/nginx-gateway-fabric/get-started/)
