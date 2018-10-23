# Lab 4: Ein Docker Image deployen

In diesem Lab werden wir gemeinsam das erste "pre-built" Docker Image deployen und die Kubernetes-Konzepte Pod, Service und Deployment etwas genauer anschauen.


## Aufgabe: LAB4.1

Nachdem wir im [Lab 3](03_first_steps.md) uns mit der Plattform vertraut gemacht haben, wenden wir uns nun dem Deployen eines pre-built Docker Images von Docker Hub oder einer anderen Docker-Registry zu.

Als ersten Schritt erstellen wir dafür erneut einen neuen Namespace. Ein Namespace ist eine Gruppierung von Ressourcen (Container und Docker Images, Pods, Services, Routen, Konfiguration, Quotas, Limiten und weiteres). Für den Namespace berechtigte User können diese Ressourcen verwalten. Innerhalb eines Kubernetes Clusters muss der Name eines Namespaces eindeutig sein.

Erstellen Sie daher einen neuen Namespace mit dem Namen `[USER]-dockerimage`:

```
$ kubectl create namespace [USER]-dockerimage
```

`kubectl create namespace` wechselt nicht automatisch in den eben neu angelegten Namespace.

Wechseln Sie daher in den entsprechend neu angelegten Namespace:

Linux:
```bash
$ kubectl config set-context $(kubectl config current-context) --namespace=[USER]-dockerimage
```

Windows:
```bash
$ kubectl config set-context %KUBE_CONTEXT% --namespace=[USER]-dockerimage
```

Als Alternative zum Wechseln des Contexts kann beim Befehl `kubectl` der Parameter `-n` mitgegeben werden.
Beispielsweise beim Anzeigen der Pods eines Namespace sieht der ganze Befehl dann wie folgt aus:

```bash
$ kubectl get pods -n <namespace>
```

Mit dem `kubectl get`-Befehl können Ressourcen von einem bestimmten Typ angezeigt werden.

Verwenden Sie:
```
$ kubectl get namespace
```
um alle Namespaces anzuzeigen, auf die Sie berechtigt sind.

## Aufgabe: LAB4.2 Pod Starten

Sobald der Namespace erstellt wurde, können wir nun unsere erste Applikation deployen. Als ersten Schritt starten wir direkt einen Pod:

```
$ kubectl run nginx --image=nginx --port=80 --restart=Never
```

Verwenden Sie `kubectl get pods` um den laufenden Pod anzuzeigen.

Schauen sie sich den nginx Pod in der [Web Console](https://console.cloud.google.com/kubernetes) unter dem Reiter "Workloads" an und löschen Sie ihn dort auch gleich wieder.

## Aufgabe: LAB4.3 Deployment

Einen einzelnen Pod zu starten kann in bestimmten Fällen Sinn machen, ist jedoch nur beschränkt Best Practice. Zusammen mit dem Pod wollen wir uns noch ein weiteres Konzept anschauen. Das sog. "Deployment". Es sorgt dafür, dass Pods überwacht werden und überprüft ob jeweils die Anzahl geforderter Pods auch tatsächlich laufen.

Sobald der Namespace erstellt wurde können wir mit dem folgenden Befehl das Docker Image im Namespace deployen:

```
$ kubectl create deployment example-spring-boot --image=appuio/example-spring-boot
```

Output:
```
deployment.apps/example-spring-boot created
```

Für unser Lab verwenden wir ein APPUiO-Beispiel (Java Spring Boot Applikation):
- Docker Hub: https://hub.docker.com/r/appuio/example-spring-boot/
- GitHub (Source): https://github.com/appuio/example-spring-boot-helloworld

Kubernetes legt die nötigen Ressourcen an, lädt das Docker Image in diesem Fall von Docker Hub herunter und deployt anschliessend den ensprechenden Pod.

Verwenden Sie den `kubectl get` Befehl mit dem `-w` Parameter, um fortlaufend Änderungen an den Ressourcen des Typs Pod anzuzeigen (**abbrechen mit ctrl+c**):
```
$ kubectl get pods -w
```

Je nach Internetverbindung oder abhängig davon, ob das Image auf Ihrem Kubernetes Node bereits heruntergeladen wurde, kann das eine Weile dauern. 

**Tipp** Um Ihre eigenen Docker Images für Kubernetes zu erstellen, sollten Sie [diese Best Practices befolgen](https://docs.openshift.com/container-platform/latest/creating_images/guidelines.html). Der Image Creation Guide ist zwar von OpenShift, jedoch auch für Kubernetes oder andere Container Plattformen anwendbar.


## Betrachten der erstellten Ressourcen

Als wir `kubectl create deployment example-spring-boot --image=appuio/example-spring-boot` vorhin ausführten, legte Kubernetes ein [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) für uns an.


### Deployment

```
$ kubectl get deployment
```

Im [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) werden folgende Punkte definiert:

- Update Strategy: wie werden Applikationsupdates ausgeführt, wie erfolgt das Austauschen der Container?
- Container
  - Welches Image soll deployed werden?
  - Environment Configuration für die Pods
  - ImagePullPolicy
- Replicas, Anzahl der Pods, die deployt werden sollen

Mit dem folgenden Befehl können zusätzliche Informationen zur Deployment ausgelesen werden:
```
$ kubectl get deployment example-spring-boot -o json
```

Nachdem das Image heruntergeladen wurde deployt Kubernetes anhand des Deployments einen Pod.

```
$ kubectl get pod
```

```
NAME                                   READY   STATUS    RESTARTS   AGE
example-spring-boot-69b658f647-xnm94   1/1     Running   0          52m
```

Im Deployment wurde definiert, dass ein Replica Pod deployt werden soll. Dieser Pod ist aktuell von ausserhalb noch nicht verfügbar.


## Aufgabe: LAB4.4 Deployment in der Web Console verifizieren

Versuchen Sie sich die Logs zur Springboot Applikation über die Web Console anzuzeigen.

---

**Ende Lab 4**

<p width="100px" align="right"><a href="05_expose_service.md">Service Exposen →</a></p>

[← zurück zur Übersicht](../README.md)
