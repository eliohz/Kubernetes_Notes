[← Zurück zur Übersicht](../../../README.md)

# Übung Job/CronJob

In dieser Übung wollen wir uns mit Jobs bzw. CronJobs in einem Kubernetes Cluster beschäftigen und wie deren Verhalten ist.

## Übersicht: Was ein Job/CronJob genau ist

Ein Job wird verwendet, um **einmalige**, **nicht wiederkehrende Aufgaben** auszuführen.
Einsatzszenarien:
- Datenmigration oder einmalige Datenverarbeitung.
- Ein Skript ausführen, das einmalig eine Konfiguration berechnet oder Änderungen in einer Datenbank durchführt.
- Temporäre Task-Execution, z.B. Verarbeitung einer Warteschlange.


Ein CronJob erzeugt Jobs nach einem **definierten Zeitplan** und ist für **regelmäßig wiederkehrende Aufgaben** geeignet.
Einsatzszenarien:
- Automatische Sicherung von Datenbanken oder Systemen (z.B. täglich um Mitternacht).
- Regelmäßiges Abrufen und Speichern von Daten aus einer API oder einem externen Service.
- Periodische Bereinigung von Logs oder alten Datenbanken-Inhalten.


| Funktion                      | Beispiel                                              |
| :---                          | :---                                                  |
| Zeige alle Jobs               | *kubectl get jobs*                                    |
| Zeige alle CronJobs           | *kubectl get cronjobs*                                |
| Beschreibung Abrufen          | *kubectl describe job job*                            |


## Job

Erstelle mit ``kubectl create ...`` ein Job mit den folgenden Eigenschaften oder verwende das vorgefertigte Manifest (``yaml/random-fail.yaml``).
- Name: random-fail
- Image: bash
- Command: ``bash -c 'env; [[ "$(( $RANDOM %2))" == "1" ]] && exit 0 || exit 1'``

- Wie wird in Kubernetes ein fehlerhafter Pod festgestellt?

Führen Sie die nachfolgenden Szenarien an dem Manifest (``yaml/random-fail.yaml``) durch.
Belassen Sie die Änderungen der einzelnen Szenarien.
Vergessen Sie nicht, den Job wieder zu löschen und dann zu deployen.

```bash
kubectl delete -f yaml/random-fail.yaml
kubectl apply -f yaml/random-fail.yaml
```

Szenario 1:
- Erhöhen Sie die Anzahl der *completions* auf 10, was wird passieren?
- Erhöhen Sie danach *parallelism* auf 4. was wird passieren?
- Speichern Sie sich den log eines beliebigen Pods (``kubectl logs random-fail-... > random-fail.log``)

Szenario 2:
- Setzen Sie den *completionMode* von ``NonIndexed`` auf ``Indexed``.
- Wie verändern sich die Pods?
- Speichern Sie nochmals einen log eines beliebigen Pods (``kubectl logs random-fail-... > random-fail.log1``), achten Sie darauf in eine neue Datei zu schreiben.
- was fällt auf bei einem diff der logs (``diff random-fail.log random-fail.log1 ``)

### Job Cleanup

```bash
kubectl delete -f yaml/random-fail.yaml
```


## Cronjob

Erstellen Sie das Deployment httpbin, das eine Anwendung mit einer einfachen API für Testing bereitstellt (Online: https://httpbin.org/).

```bash
kubectl apply -f yaml/httpbin.yaml
```

Verwenden Sie kubectl und finden Sie die IP-Adresse des neuen httpbin-Pods heraus. (``kubectl get ...``)
Ersetzen Sie in dem ``yaml/curl-cronjob.yaml`` Manifest den Platzhalter ``<HTTPBIN_POD_IP>`` mit der richtigen IP des Pods.

- Wie oft wird ein Job durch den CronJob erstellt?
- Für was könnte die Einstellung *ttlSecondsAfterFinished* im Manifest stehen?
- Was genau macht der Container/Pod des Jobs?
- Wann wird der erste Pod ausgeführt?

> **Hinweis:** Falls Sie mit der Linux-Cron Syntax nicht vertraut sind, bietet https://crontab.guru/ eine gute Hilfestellung.

Wenden Sie das angepasste Manifest an und überprüfen Sie ihre Annahmen.


```bash
kubectl apply -f yaml/curl-cronjob.yaml
```

- Wie sind die Jobs/Pods benannt?

###  Cleanup CronJob

```bash
kubectl delete -f yaml/curl-cronjob.yaml
kubectl delete -f yaml/httpbin.yaml
```
