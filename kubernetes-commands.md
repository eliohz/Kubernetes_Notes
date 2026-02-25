[← Zurück zur Übersicht](./README.md)

# Kubernetes Commands Cheat Sheet

---

## Cluster & Kontext

```bash
kubectl version                          # Client- und Server-Version
kubectl cluster-info                     # Cluster-Endpunkte
kubectl get nodes                        # Alle Nodes anzeigen
kubectl get nodes -o wide                # Nodes mit IP, OS, Kernel
kubectl describe node <node>             # Details zu einem Node

kubectl config get-contexts              # Alle Kontexte auflisten
kubectl config current-context          # Aktuellen Kontext anzeigen
kubectl config use-context <kontext>     # Kontext wechseln
```

---

## Namespaces

```bash
kubectl get namespaces                   # Alle Namespaces
kubectl create namespace <name>          # Namespace erstellen
kubectl delete namespace <name>          # Namespace löschen

# Namespace für alle folgenden Befehle setzen
kubectl config set-context --current --namespace=<name>

# Befehle in einem bestimmten Namespace ausführen
kubectl get pods -n <namespace>
kubectl get all -n <namespace>           # Alles in einem Namespace
kubectl get all -A                       # Alles in allen Namespaces
```

---

## Pods

```bash
kubectl get pods                         # Pods im aktuellen Namespace
kubectl get pods -o wide                 # Mit Node, IP
kubectl get pods --watch                 # Live-Updates (-w)
kubectl get pods -l app=nginx            # Pods nach Label filtern
kubectl get pods --sort-by=metadata.name

kubectl describe pod <pod>               # Details & Events
kubectl logs <pod>                       # Logs anzeigen
kubectl logs <pod> -f                    # Logs streamen (follow)
kubectl logs <pod> -c <container>        # Logs eines bestimmten Containers
kubectl logs <pod> --previous            # Logs des vorherigen Containers

kubectl exec <pod> -- <befehl>           # Befehl im Pod ausführen
kubectl exec -it <pod> -- /bin/bash      # Interaktive Shell öffnen
kubectl exec -it <pod> -c <c> -- bash   # In bestimmten Container

kubectl run <name> --image=nginx         # Pod schnell erstellen
kubectl delete pod <pod>                 # Pod löschen
kubectl delete pod <pod> --force         # Pod sofort löschen
```

---

## Deployments

```bash
kubectl get deployments
kubectl describe deployment <name>

kubectl create deployment <name> --image=nginx
kubectl apply -f deployment.yaml
kubectl delete deployment <name>

# Skalieren
kubectl scale deployment <name> --replicas=3

# Image aktualisieren (Rolling Update)
kubectl set image deployment/<name> <container>=nginx:1.25

# Rollout verwalten
kubectl rollout status deployment/<name>    # Status beobachten
kubectl rollout history deployment/<name>   # Versionen anzeigen
kubectl rollout undo deployment/<name>      # Letzten Rollout rückgängig
kubectl rollout undo deployment/<name> --to-revision=2
kubectl rollout restart deployment/<name>   # Pods neu starten
```

---

## ReplicaSets & DaemonSets

```bash
kubectl get replicasets                  # ReplicaSets anzeigen
kubectl describe replicaset <name>

kubectl get daemonsets
kubectl describe daemonset <name>
```

---

## Jobs & CronJobs

```bash
kubectl get jobs
kubectl get cronjobs
kubectl describe job <name>

kubectl create job <name> --image=busybox -- echo hello
kubectl delete job <name>

kubectl create cronjob <name> --image=busybox --schedule="*/5 * * * *" -- echo hello
kubectl delete cronjob <name>
```

---

## Services

```bash
kubectl get services                     # Services anzeigen (svc)
kubectl get svc -o wide
kubectl describe service <name>

kubectl expose pod <pod> --port=80 --target-port=8080 --type=ClusterIP
kubectl expose deployment <name> --port=80 --type=NodePort
kubectl delete service <name>

# Port-Forwarding (für lokalen Zugriff)
kubectl port-forward pod/<pod> 8080:80
kubectl port-forward svc/<service> 8080:80
kubectl port-forward deployment/<name> 8080:80
```

---

## ConfigMaps & Secrets

```bash
# ConfigMaps
kubectl get configmaps
kubectl describe configmap <name>
kubectl create configmap <name> --from-literal=key=value
kubectl create configmap <name> --from-file=config.properties
kubectl delete configmap <name>

# Secrets
kubectl get secrets
kubectl describe secret <name>
kubectl create secret generic <name> --from-literal=password=geheim
kubectl create secret docker-registry <name> \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass

# Secret-Wert dekodieren
kubectl get secret <name> -o jsonpath='{.data.password}' | base64 --decode
```

---

## Persistent Volumes

```bash
kubectl get persistentvolumes            # PV anzeigen
kubectl get persistentvolumeclaims       # PVC anzeigen (pvc)
kubectl get pvc
kubectl describe pvc <name>
kubectl delete pvc <name>
```

---

## RBAC

```bash
kubectl get roles
kubectl get rolebindings
kubectl get clusterroles
kubectl get clusterrolebindings

kubectl describe role <name>
kubectl describe rolebinding <name>

# Berechtigungen prüfen
kubectl auth can-i create pods
kubectl auth can-i create pods --as=<user>
kubectl auth can-i create pods -n <namespace>
```

---

## Ingress & Gateway

```bash
kubectl get ingress
kubectl describe ingress <name>

kubectl get gateways
kubectl get httproutes
```

---

## Ressourcen-Verwaltung

```bash
# Ressourcenverbrauch (benötigt metrics-server)
kubectl top nodes
kubectl top pods
kubectl top pods -A

# Alle Ressourcen eines Typs anzeigen
kubectl api-resources                    # Alle verfügbaren Ressourcentypen

# Ressource erklären
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers --recursive
```

---

## Output-Formate & Filtern

```bash
kubectl get pods -o yaml                 # YAML-Ausgabe
kubectl get pods -o json                 # JSON-Ausgabe
kubectl get pod <pod> -o jsonpath='{.status.podIP}'

# Labels anzeigen / setzen
kubectl get pods --show-labels
kubectl label pod <pod> env=prod
kubectl label pod <pod> env-              # Label entfernen

# Annotationen
kubectl annotate pod <pod> description="mein pod"
```

---

## Apply, Delete & Dry-Run

```bash
kubectl apply -f datei.yaml              # Ressource erstellen/aktualisieren
kubectl apply -f ./verzeichnis/          # Alle YAMLs in einem Ordner
kubectl delete -f datei.yaml             # Ressource aus Datei löschen

# Dry-Run (nichts wird wirklich erstellt)
kubectl apply -f datei.yaml --dry-run=client
kubectl apply -f datei.yaml --dry-run=server

# YAML aus bestehendem Objekt generieren
kubectl get pod <pod> -o yaml > pod.yaml
kubectl get deployment <name> -o yaml > deployment.yaml
```

---

## Debugging & Troubleshooting

```bash
kubectl describe <ressource> <name>      # Events & Details anzeigen
kubectl get events                       # Cluster-Events
kubectl get events --sort-by=.lastTimestamp

# Temporären Debug-Pod starten
kubectl run debug --image=busybox -it --rm -- /bin/sh
kubectl run debug --image=nicolaka/netshoot -it --rm -- bash

# Netzwerk testen
kubectl exec <pod> -- curl http://<service>
kubectl exec <pod> -- nslookup <service>
kubectl exec <pod> -- env                # Umgebungsvariablen prüfen
```

---

## Helm

```bash
helm repo add <name> <url>               # Repository hinzufügen
helm repo update                         # Repositories aktualisieren
helm repo list

helm search repo <suche>                 # Charts suchen
helm show values <chart>                 # Default-Values anzeigen

helm install <release> <chart>           # Chart installieren
helm install <release> <chart> -f values.yaml
helm install <release> <chart> --set key=value

helm list                                # Installierte Releases
helm status <release>
helm upgrade <release> <chart>           # Upgrade
helm rollback <release> <revision>       # Rollback
helm uninstall <release>                 # Deinstallieren

helm template <release> <chart>          # YAML generieren ohne installieren
```

---

## Kustomize

```bash
kubectl kustomize ./                     # Kustomization rendern
kubectl apply -k ./                      # Kustomization anwenden
kubectl delete -k ./                     # Kustomization löschen

kustomize build ./                       # Alternativ mit kustomize CLI
kustomize build ./ | kubectl apply -f -
```
