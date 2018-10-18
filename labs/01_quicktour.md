# Lab 1: Quicktour durch Kubernetes

In diesem Lab werden die Grundkonzepte von Kubernetes vorgestellt. 

Die hier aufgeführten Begriffe und Ressourcen sind ein Auszug aus der offiziellen Kubernetes Dokumentation, weiterführende Informationen zu Kubernetes können hier entnommen werden:

> https://kubernetes.io/docs/concepts/


## Grundkonzepte

Kubernetes is Open Source  und bietet eine Plattform, mit der Software in Containern deployt und betrieben werden kann. Kubernetes kann als Container Platform oder Platform as a Service (PaaS) bezeichnet werden.

### Docker

[Docker](https://www.docker.com/) ist die offene Plattform für Entwickler und Sysadmins und ihre Applikationen. Wählen Sie das für Ihre Technologie passende Basis-Docker-Image aus, Kubernetes deployt für Sie nach jedem Build einen aktualisierten Docker-Container.


## Übersicht

Kubernetes besteht aus Kubernetes Master und Kubernetes Nodes. 
Der Master ist die Schnittstelle gegen aussen, um dem Cluster mitzuteilen, welche Applikationen er wie deployen soll. Des Weiteren ist er für das Verwalten des aktuellen Zustands zuständig.
Die Nodes sind sogenannte Compute Nodes und führen die Applikationen und Workloads aus. Der Master verwaltet die Nodes.

### Container und Docker Images

Die Basiselemente von Kubernetes Applikationen sind Container. Mit Docker Containern können Prozesse auf einem Linuxsystem so isoliert werden, dass sie nur mit den definierten Ressourcen interagieren können. So können viele unterschiedliche Container auf dem gleichen System laufen, ohne dass sie einander "sehen" (Dateien, Prozesse, Netzwerk). Typischerweise beinhaltet ein Container einen einzelnen Service (Webserver, Datenbank, Mailservice, Cache). Innerhalb eines Docker Containers können beliebige Prozesse ausgeführt werden.

Docker Container basieren auf Docker Images. Ein Docker Image ist eine binary Datei, die alle nötigen Komponenten beinhaltet, damit ein einzelner Container ausgeführt werden kann.

Docker Images werden anhand von DockerFiles (textueller Beschrieb, wie das Docker Image Schritt für Schritt aufgebaut ist) gebaut. Grundsätzlich sind Docker Images hierarchisch angewendete Filesystem Snapshots.

**Beispiel Tomcat**
- Basis Image (CentOs 7)
  + Install Java
  + Install Tomcat
  + Install App

Die gebauten Docker Images werden in einer Docker Registry versioniert abgelegt und stehen der Plattform nach dem Bau ("Build") zum Deployment zur Verfügung.

### Namespaces

In Kubernetes werden Ressourcen (Container und Docker Images, Pods, Services, Konfiguration, Quotas und Limiten etc.) in Namespaces strukturiert.

Innerhalb eines Namespaces können berechtigte User ihre Ressourcen selber verwalten und organisieren.

Die Ressourcen innerhalb eines Namespaces sind über ein transparentes [SDN](https://de.wikipedia.org/wiki/Software-defined_networking) verbunden. So können die einzelnen Komponenten eines Namespaces in einem Multi-Node Setup auf verschiedene Nodes deployed werden. Dabei sind sie über das SDN untereinander sicht- und zugreifbar.

### Pods

Ein Pod besteht aus ein oder mehreren Containern, die zusammen auf den gleichen Host deployed werden. Ein Pod ist die kleinste zu deployende Einheit auf Kubernetes.
Typischerweise haben mehrere Container innerhalb eines Pods den gleichen Lifecycle.

Ein Pod ist innerhalb eines Kubernetes Namespaces über den entsprechenden Service verfügbar.

### Services

Ein Service repräsentiert einen internen Loadbalancer oder eine virtuelle IP-Adresse für die dahinterliegenden Pods (Replicas vom gleichen Typ). Der Service dient als Proxy zu den Pods und leitet Anfragen an diese weiter. So können Pods willkürlich einem Service hinzugefügt und entfernt werden, während der Service verfügbar bleibt.

Einem Service ist innerhalb eines Namespaces eine IP-Adresse und ein Port zugewiesen und verteilt Requests entsprechend auf die Pod Replicas.

### Deployment

Siehe: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

### Volume

Siehe: https://kubernetes.io/docs/concepts/storage/volumes/

### Job

Siehe: https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/

---

**Ende Lab 1**

<p width="100px" align="right"><a href="02_cli.md">Kubernetes CLI installieren →</a></p>

[← zurück zur Übersicht](../README.md)
