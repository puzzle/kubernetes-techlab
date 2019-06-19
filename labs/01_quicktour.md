# Lab 1: Quicktour durch Kubernetes

In diesem Lab werden die Grundkonzepte von Kubernetes vorgestellt. 

Die hier aufgeführten Begriffe und Ressourcen sind ein Auszug aus der offiziellen Kubernetes Dokumentation, weiterführende Informationen zu Kubernetes können [der offiziellen Dokumentation](https://kubernetes.io/docs/concepts/) entnommen werden.


## Grundkonzepte

Kubernetes ist Open Source Software und bietet eine Plattform, mit der Software in Container deployt und betrieben werden kann. Kubernetes kann als Container Platform oder auch Container-as-a-Service (CaaS) bezeichnet werden, je nach Ausprägung auch als Platform-as-a-Service (PaaS).


### Docker

[Docker](https://www.docker.com/) kann als Container Engine zusammen mit Kubernetes eingesetzt werden und ist eine Plattform für Entwickler, Sysadmins und ihre Applikationen. Wählen Sie das für Ihre Technologie passende Base Image aus, Kubernetes deployt für Sie nach jedem Build einen aktualisierten Container.


## Übersicht

Kubernetes besteht aus Kubernetes Master und Kubernetes Nodes. 
Der Master ist die Schnittstelle gegen aussen, um dem Cluster mitzuteilen, welche Applikationen er wie deployen soll. Desweiteren ist er für das Verwalten des aktuellen Zustands zuständig.
Die Nodes sind sog. Compute Nodes und führen die Applikationen und Workloads aus. Der Master verwaltet die Nodes.


### Container und Images

Die Basiselemente von Kubernetes Applikationen sind Container. Mit Containern können Prozesse auf einem Linuxsystem so isoliert werden, dass sie nur mit den definierten Ressourcen interagieren können. So können viele unterschiedliche Container auf dem gleichen System laufen, ohne dass sie einander "sehen" (Dateien, Prozesse, Netzwerk). Typischerweise beinhaltet ein Container einen einzelnen Service (Webserver, Datenbank, Mailservice, Cache). Innerhalb eines Containers können beliebige Prozesse ausgeführt werden.

Container basieren auf Images. Ein Image ist eine binäre Datei, die alle nötigen Komponenten beinhaltet, damit ein einzelner Container ausgeführt werden kann.

Container Images werden anhand von bspw. Dockerfiles (textueller Beschrieb, wie das Image aufgebaut ist) gebaut. Grundsätzlich sind Container Images hierarchisch angewandte Filesystem Snapshots.

**Beispiel Tomcat**
- Basis Image (CentOS 7)
- + Install Java
- + Install Tomcat
- + Install App

Die gebauten Container Images werden in einer Image Registry versioniert abgelegt und stehen der Plattform nach dem Bau ("Build") zum Deployment zur Verfügung.


### Namespaces

In Kubernetes werden Ressourcen (Container und Docker Images, Pods, Services, Konfiguration, Quotas und Limiten etc.) in Namespaces strukturiert.

Innerhalb eines Namespaces können berechtigte User ihre Ressourcen selber verwalten und organisieren.

Die Ressourcen innerhalb eines Namespace sind über ein transparentes [SDN](https://de.wikipedia.org/wiki/Software-defined_networking) verbunden. So können die einzelnen Komponenten eines Namespace in einem Multi-Node Setup auf verschiedene Nodes deployed werden. Dabei sind sie über das SDN untereinander sicht- und zugreifbar.


### Pods

Ein Pod besteht aus ein oder mehreren Containern, die zusammen auf dem gleichen Node deployed werden. Ein Pod ist die kleinste zu deployende Einheit auf Kubernetes.
Typischerweise haben mehrere Container innerhalb eines Pods den gleichen Lifecycle.

Ein Pod ist innerhalb eines Kubernetes Namespace über den entsprechenden Service verfügbar.


### Services

Ein Service repräsentiert einen internen Loadbalancer oder eine virtuelle IP-Adresse für die dahinterliegenden Pods (Replicas desselben Typs). Der Service dient als Proxy zu den Pods und leitet Anfragen an diese weiter. So können Pods willkürlich einem Service hinzugefügt und entfernt werden, während der Service verfügbar bleibt.

Einem Service ist innerhalb eines Namespace eine IP-Adresse und ein Port zugewiesen und verteilt Requests entsprechend auf die Pod Replicas.


### Deployment

Siehe <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>


### Volume

Siehe <https://kubernetes.io/docs/concepts/storage/volumes/>


### Job

Siehe <https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/>

---

**Ende Lab 1**

<p width="100px" align="right"><a href="02_cli.md">Kubernetes CLI installieren →</a></p>

[← zurück zur Übersicht](../README.md)
