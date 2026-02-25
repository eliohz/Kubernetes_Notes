[← Zurück zur Übersicht](../../../README.md)

# Übung Ingress

In dieser Übung möchten wir uns mit dem Ingress und der Ingress-Ressource beschäftigen.
Dabei möchten wir vor allem die Funktionsweise kennenlernen und wie wir TrafficRouten entsprechend unseren Vorgaben definieren können.
Bei [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) und der einhergehenden Ingress-Konfiguration wird festgelegt, wie der eingehende Datenverkehr vom Ingress-Controller verarbeitet und durch die Referenz durch einen Service an den entsprechenden Pod weitergeleitet wird.
Im Vergleich zu den vorgehenden Services arbeitet der Ingress auf Level 7 des [OSI-Modells](https://de.wikipedia.org/wiki/OSI-Modell) und nicht auf Level 4.
Damit Ingress im Cluster verwendet werden kann, muss ein [Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) installiert sein. Die unterschiedlichen Hersteller der Ingress-Controller bieten einfach installierbare Manifeste an.

Wir verwenden im Weiteren den [ingress-nginx-Controller](https://kubernetes.github.io/ingress-nginx/deploy/). Da dessen Konfiguration für unseren Versuch nicht optimal ist, muss noch eine Anpassung vorgenommen werden.

> **Hinweis:** Die gemachte Änderung ist nur für Demonstrationszwecke, und sollte nicht unbedacht in Produktivsysteme gemacht werden!

````sh
# get ingress-nginx manifest for installation
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
sed "s/externalTrafficPolicy: Local/externalTrafficPolicy: Cluster/" deploy.yaml > deploy-mod.yaml
# CHECK externalTrafficPolicy should be set to Cluster
grep externalTrafficPolicy deploy-mod.yaml
# apply on cluster
kubectl apply -f deploy-mod.yaml
````

- Es sollte ein neuer Namespace existieren. Welcher ist das?
- Welche Objekte wurden in dem Namespace erstellt?
- Welche Services gibt es in dem Namespace und welcher NodePort ist gesetzt (Hinweis)?

> Der Service ``ingress-nginx-controller`` ist zwar vom Typ LoadBalancer, arbeitet aber wie ein NodePort-Service. Er besitzt daher auch einen entsprechenden.

Nachdem der Ingress-Controller bereit ist, können wir eine Ingress-Resource anlegen.
Ein Service existiert ja bereits für unser Frontend.
Da wir keinen offiziellen Hostnamen haben, setzten wir einfach einen Dummy-Hostnamen ``example.com``.
Für unser Beispiel reicht kubectl zum Anlegen der Resource, für den Produktiveinsatz sollte man eine entsprechende [Ingress-Resource als YAML](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource) anlegen.

````sh
# show example in kubectl help
kubectl create ingress --help

kubectl create ingress frontend-ingress --rule=example.com/=frontend:8080 --class nginx
````

- Die erstellte Ingress-Ressource näher untersuchen (``kubectl get ...``, ``kubectl describe ...``)

Die Ingress-Ressource sollte jetzt existieren und somit der Service bzw. der Pod entsprechend erreichbar sein.
Da wir, wie beschrieben, nur einen "Dummy-Hostnamen" gesetzt haben, muss dieser beim Aufrufen mit curl oder manuell durch einen Eintrag in ``/etc/hosts`` gesetzt werden.

````sh
# Complex jsonpath filter to get nodeport of http port configuration
NODE_PORT=$(kubectl -n ingress-nginx get services ingress-nginx-controller -o jsonpath="{.spec.ports[?(.name==\"http\")].nodePort}")
curl -H "host: example.com" 10.0.0.11:$NODE_PORT
curl -H "host: example.com" 10.0.0.13:$NODE_PORT
curl -H "host: example.com" 10.0.0.15:$NODE_PORT
````

Jeder der curl aufrufe sollte zu einer entsprechenden Antwort führen.

> **Hinweis:** In der Antwort des Pods sieht man entsprechende HTTP-Header, welche gesetzt sind die den Echo Server erreichen, beispielsweise die ``X-Real-Ip``.
> Je nachdem welchen Knoten man für den curl-Aufruf, verwendet ist dort eine andere IP eingetragen.
> Warum ist das so bzw. welche IPs sind das und wo sind diese zu finden?.
