[← Zurück zur Übersicht](../../../README.md)

# Services

Pods besitzen zwar eine IP Adresse und sind im Cluster darüber ansprechbar, jedoch sind Pods kurzlebig und diese IP kann sich relativ häufig ändern.
Damit abhängige Pods, beispielsweise ein 3-Tier Applikation aus Frontend-, Backend- und Datenbank-Server im Cluster-Netzwerk einfach kommunizieren können sollten Services eingesetzt werden.
In dieser Übung wollen wir uns mit den verschiedenen Typen von Services vertraut machen.

## Cluster IP

Zuerst wollen wir überprüfen welche Services bereits es im Cluster (``default`` namespace) gibt und diese etwas genauer untersuchen.

```sh
kubectl get service
#kubectl get svc
```

- Es sollte minimum ein Service existieren, welcher ist das?
- Von welchem Typ ist der Service?
- Wie lautet der Port des Services?
- Wie lautet der TargetPort des Service?
- Was ist nochmal der Unterschied zwischen Port und TargetPort?
- Was wurde als selector gesetzt?
- Welche IP wurde dem Service zugeteilt?
- Was ist als Endpoint eingetragen und wo könnten wir das finden?

> **Hinweis:**  Endpoints ist eine eigene API-Ressource und wird in der Regel automatisch beim erstellen des Services erstellt.
> Sie halten die Information zu welchem Pod/Endpunkt der Datenverkehr weitergeleitet wird.


Als nächstes wollen wir ein Cluster-IP Service für einen Deployment erstellen und es somit für andere Pods erreichbar machen.

````sh
kubectl create deployment frontend --image jmalloc/echo-server
````

Damit wir das frontend über einen eindeutigen Namen ("Cluster-DNS") im Server erreichen, legen wir wie folgt den Service an.

````sh
kubectl expose deployment frontend --type ClusterIP --port 8080
````

Um die Namensauflösung innerhalb des Clusters zu testen, erstellen wir einen Pod in dem wir verschiedene Kommandos absetzten können.

````sh
kubectl create deployment toolbox --image nicolaka/netshoot -- nc -kl 8080
````

Jetzt können wir über die erstellte toolbox-Pod weitere Kommandos absetzten.

````sh
TOOLBOX_POD=$(kubectl get pod -l app=toolbox -o name)
echo $TOOLBOX_POD

#
kubectl exec $TOOLBOX_POD -- curl -s frontend:8080

# scale frontend to test load balancing function
kubectl scale deployment frontend --replicas 2
# make some requests and check response
````

- welche Endpoints hat der Service nach der Skalierung?

## NodePort

Durch Service vom Typ NodePort erreichen wir, das Datenverkehr über alle Cluster-Knoten über einen der "high ports" (30000-32767) zu einem entsprechendem Ziel, weitergeleitet wird.
In diesem zweiten Abschnitt wollen wir unser Frontend durch ein solchen Service erreichbar machen.

Erstellen des Service Templates mit ``kubectl create``

````sh
kubectl create svc nodeport frontend-nodeport --tcp 8080:8080 --node-port 30080
````

Der Service sollte erstellt sein, ist er aber schon funktionsfähig bzw. ist der Pod erreichbar?

````sh
curl 10.0.0.11:30080
curl 10.0.0.13:30080
curl 10.0.0.15:30080
````

- Endpoints betrachten
- Selector des Service betrachten

````sh
kubectl edit svc frontend-nodeport
# selector abändern
````

Nochmals die Funktionsfähigkeit mit ``curl`` testen.

## Cleanup

Wir verwenden in der nachfolgenden Übung Ingress und Gateway-API ein paar der erstellten Ressourcen.
Sie müssen daher die Ressourcen nicht abräumen.
