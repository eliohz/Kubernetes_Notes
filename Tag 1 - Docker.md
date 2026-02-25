## Container
leichtgewichtiges, isoliertes Softwarepaket, das eine Anwendung inklusive aller benötigten Abhängigkeiten (Code, Laufzeitumgebung, Systemwerkzeuge, Bibliotheken) enthält

![[Pasted image 20260225094218.png]]

### Open Container initiative (OCI)
https://opencontainers.org/
Definition wie Container/Container Images aufgebaut und strukturiert sind. 

### Grundlagen
#### Kernel Funktionalität
- namespace ()
- cgroups (ressourcen trennen (Wievil Cores, Ram, etc.))
- seccomp-bpf (Security modes)
- -> isolierte Prozesse (container)
Vorteile: 
- isolation
- konsistente Umgebung
- Überall ausführbar

### Container Runtime
Die grundlegende Softwarekomponente auf einem Host-Betriebssystem, die für das Ausführen, Stoppen und Verwalten von Containern zuständig ist. (Kubernetes muss via API ansprechen können)
### Container Engines
- ContainerD
- CRI-O
- Docker
### Docker
Dockerfile --build-> Docker image --run-> Docker Container

## Registry
Images die gespeichert werden auf einem Server