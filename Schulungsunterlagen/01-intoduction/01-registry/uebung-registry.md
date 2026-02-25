[← Zurück zur Übersicht](../../../README.md)

# Installation einer privaten Registry

In dieser Übung wollen wir uns eine kleine eigene Registry für Container bereitstellen.

Mit der Ausführung von

````bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
````
wird die Registry als Container gestartet.


Nachfolgend laden wir eine Image herunter und laden es danach in die eingene lokale Registry.

Mit _pull_ wird der Download des alpine-Image von dem Docker hub gestartet.
````bash
docker pull alpine
````

Durch die Erstellung eines neuen Tags für ein existierendes Image, kann man festlegen auf welche Registry man es transferieren möchte.
Der erste Teil des Tags ist der Domainname (in unserem fall localhost und Port).
Docker interpretiert dies als den Standort von wo das Image kommt bzw. wo es bei einem *push* hin geladen werden soll.

````bash
docker tag docker.io/alpine localhost:5000/my-alpine
````

> **Hinweis:** Listet man sich die Images jetzt in docker auf (``docker images``), sieht man zwar ein neues Image, ebenfalls mit einer Größe angegeben, jedoch belegt es kein extra Speicher, da es die gleiche Image ID wie das Image zuvor hat. Der Tag kann als Referenz auf ein Image mit seiner ID betrachtet werden.

Push des Images zu der lokalen Registry
````bash
docker push localhost:5000/my-alpine
````

Falls es Probleme beim pushen gibt entsprechend die folgende Änderung vornehmen:

````bash
# open daemon.json
sudo vi etc/docker/daemon.json
# and add/insert
#{
#    “insecure-registries“:[“localhost:5000“]
#}
````

nachdem auf jeden Fall: ``systemctl restart docker``


Entfernen des lokal gespeicherte alpine-Image sowie der neue Tag localhost:5000/my-alpine, um zu testet, ob man das Image aus der lokalen Registry laden kann (pull).

````bash
docker rmi alpine
docker rmi localhost:5000/my-alpine
````


Jetzt Test , ob Image aus der Registry geladen werden kann:

````bash
docker pull localhost:5000/my-alpine
````


Check des Images:

````bash
docker images
````

> **Hinweis:** Die Registry bietet eine API an über die auch Docker kommuniziert. Über einfache API-Endpunkte wie ``/v2/_catalog`` und ``/v2/<IMAGE_REPO>/tags/list`` kann man sich ein Überblick über die verfügbaren Images verschaffen. Beispiel ``curl localhost:5000/v2/_catalog`` und ``curl localhost:5000/v2/my-alpine/tags/list``. Es gibt aber auch Tools ([Skopeo][github-skopeo]) die hier helfen und mit Registries interagieren können und entsprechend auch Authentisierung abdecken.


Die lokale Registry stoppen und den Container entfernen:

````bash
docker container stop registry && docker container rm -v registry
````

## Registry Management

Nachfolgend wollen wir noch ein paar wichtige Punkte bei der Handhabung der Registry betrachten. Die [Dokumentation der Registry][cncf-distribution-docs] bietet eine umfassende Sammlung zu den unterschiedlichsten Themen.

**Container Auto-Restarts:** \
Durch die Option ``--restart`` erreicht man einen automatisch startenden Container, sobald Docker auf dem Host gestartet ist:

````bash
docker run -d \
-p 5000:5000 \
--restart=always \
--name registry \
registry:2
````

**Registry Konfiguration**\
Die Registry lässt sich mittels Umgebungsvariablen konfigurieren.
So kann man Beispielsweise die Adresse/Port der Registry festlegen.

````bash
docker run -d \
-e REGISTRY_HTTP_ADDR=0.0.0.0:5001 \
-p 5001:5001 \
--name registry \
registry:2
````

**Container Bind-Mount**\
Standardmäßig werden die Daten der Registry in einem Docker-Volume auf dem Host-Dateisystem gespeichert.
Wenn Sie Ihre Registry-Inhalte an einem bestimmten Ort auf Ihrem Host-Dateisystem speichern möchten, z. B. wenn Sie eine SSD oder ein SAN in ein bestimmtes Verzeichnis gemountet haben, können Sie stattdessen einen Bind-Mount verwenden.
Ein Bind-Mount ist stärker vom Dateisystem-Layout des Docker-Hosts abhängig, ist aber in vielen Situationen performanter.
Das folgende Beispiel bindet das Host-Verzeichnis ``/mnt/registry`` in den Registry-Container unter ``/var/lib/registry/`` ein.

````bash
docker run -d \
-p 5000:5000 \
--restart=always \
--name registry \
-v /mnt/registry:/var/lib/registry \
registry:2
````

## Referenzen

- [Github Skopeo][github-skopeo]
- [CNCF Distribution Registry][cncf-distribution-docs]

[github-skopeo]: https://github.com/containers/skopeo
[cncf-distribution-docs]: https://distribution.github.io/distribution/
