# Lab 4: Ein Docker Image deployen

In diesem Lab werden wir gemeinsam das erste "pre-built" Docker Image deployen und die Kubernetes-Konzepte Pod, Service und Deployment etwas genauer anschauen.


## Aufgabe: LAB4.1

Nachdem wir im [Lab 3](03_first_steps.md) uns mit der Plattform vertraut gemacht haben, wenden wir uns nun dem Deployen eines pre-built Docker Images von Docker Hub oder einer anderen Docker-Registry zu.

> [Weiterführende Dokumentation](https://docs.openshift.com/container-platform/3.9/dev_guide/application_lifecycle/new_app.html#specifying-an-image)

Als ersten Schritt erstellen wir dafür einen neuen Namespace. Ein Namespace ist eine Gruppierung von Ressourcen (Container und Docker Images, Pods, Services, Routen, Konfiguration, Quotas, Limiten und weiteres). Für den Namespace berechtigte User können diese Ressourcen verwalten. Innerhalb eines Kubernetes Clusters muss der Name eines Namespaces eindeutig sein.

Erstellen Sie daher einen neuen Namespace mit dem Namen `[USER]-dockerimage`:

```
$ kubectl creat namespace [USER]-dockerimage
```

`kubectl create namespace` wechselt nicht automatisch in den eben neu angelegten Namespace. 

Wechseln Sie daher in den entsprechend neu angelegten Namespace
```
$ kubectl config set-context $(kubectl config current-context) --namespace=[namespace]
```

als Alternative zum switchen des Contexts, kann beim Befehl `kubectl` der parameter `-n` mitgegeben werden.
Beispielsweise beim Anzeigen der Pods eines Namespaces sieht der ganze Befehl dann wie folgt aus:
```bash
$ kubectl get pods -n=namespace
```

Mit dem `kubectl get` Command können Ressourcen von einem bestimmten Typ angezeigt werden.

Verwenden Sie
```
$ kubectl get namespace
```
um alle Namespaces anzuzeigen, auf die Sie berechtigt sind.

Sobald der Namespace erstellt wurde, können wir mit dem folgenden Befehl das Docker Image deployen im Namespace deployen:

```
$ kubectl create deployment spring-boot-example --image=appuio/example-spring-boot
```
Output:
```
deployment.apps/spring-boot-example created
```

Für unser Lab verwenden wir ein APPUiO-Beispiel (Java Spring Boot Applikation):
- Docker Hub: https://hub.docker.com/r/appuio/example-spring-boot/
- GitHub (Source): https://github.com/appuio/example-spring-boot-helloworld

Kubernetes legt die nötigen Ressourcen an, lädt das Docker Image in diesem Fall von Docker Hub herunter und deployt anschliessend den ensprechenden Pod.

Verwenden Sie den `kubectl get` Befehl mit dem `-w` Parameter, um fortlaufend Änderungen an den Ressourcen des Typs Pod anzuzeigen (abbrechen mit ctrl+c):
```
$ kubectl get pods -w
```

Je nach Internetverbindung oder abhängig davon, ob das Image auf Ihrem Kubernetes Node bereits heruntergeladen wurde, kann das eine Weile dauern. 

**Tipp** Um Ihre eigenen Docker Images für Kubernetes zu erstellen, sollten Sie die folgenden Best Practices befolgen: https://docs.openshift.com/container-platform/3.9/creating_images/guidelines.html Der Image Creation Guide ist zwar von OpenShift, jedoch auch für Kubernetes pur anwendbar


## Betrachten der erstellten Ressourcen

Als wir `kubectl create deployment spring-boot-example --image=appuio/example-spring-boot` vorhin ausführten, hat Kubernetes ein Deployment für uns angelegt:

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

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


Nach dem das Image heruntergeladen wurde deployt Kubernetes anhand des Deployments einen Pod

```
$ kubectl get pod
```

```
NAME                                   READY   STATUS    RESTARTS   AGE
spring-boot-example-69b658f647-xnm94   1/1     Running   0          52m
```

Im Deployment wurde definiert, das ein Replica Pod deployt werden soll. Dieser Pod ist aktuell von ausserhalb noch nicht verfügbar.

Als erste Variante "exposen" wir den Pod nun via [Service](https://kubernetes.io/docs/concepts/services-networking/service/) und sogenannten NodePort

Damit machen wir nun die Applikation vom Internet her verfügbar.

### Service

Mit dem folgenden Befehl wird unser Deployment über den Type LoadBalancer auf Port 80 und Pod Target Port 8080 exposed

```
$ kubectl expose deployment spring-boot-example --type="LoadBalancer" --name="example-spring-boot" --port=80 --target-port=8080
```

[Services](https://kubernetes.io/docs/concepts/services-networking/service/) dienen innerhalb Kubernetes als Abstraktionslayer, Einstiegspunkt und Proxy/Loadbalancer auf die dahinterliegenden Pods. Der Service ermöglicht es, innerhalb OpenShift eine Gruppe von Pods des gleichen Typs zu finden und anzusprechen.

Als Beispiel: Wenn eine Applikationsinstanz unseres Beispiels die Last nicht mehr alleine verarbeiten kann, können wir die Applikation bspw. auf drei Pods hochskalieren. Kubernetes mapt diese als Endpoints automatisch zum Service. Sobald die Pods bereit sind, werden Requests automatisch auf alle drei Pods verteilt.

**Note:** Die Applikation kann aktuell von aussen noch nicht erreicht werden, der Service ist ein Kubernetes-internes Konzept. Im folgenden Lab werden wir die Applikation öffentlich verfügbar machen.

Nun schauen wir uns unseren Service mal etwas genauer an:

```
$ kubectl get services
```

```
NAME                TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
example-spring-boot LoadBalancer   10.39.247.42   <pending>     80:30180/TCP   2s
```

**Note:** die External IP wird erst nachträglich eingetragen, es braucht hier einen Moment bis sie verfügbar ist.

Wie Sie am Output sehen, ist unser Service (example-spring-boot) über eine IP und Port erreichbar (172.30.124.20:8080) **Note:** Ihre IP kann unterschiedlich sein.

**Note:** Service IPs bleiben während ihrer Lebensdauer immer gleich.

Mit dem folgenden Befehl können Sie zusätzliche Informationen über den Service auslesen:
```
$ kubectl get service example-spring-boot -o json
```

```
{
    "apiVersion": "v1",
    "kind": "Service",
    "metadata": {
        "creationTimestamp": "2018-10-15T14:53:37Z",
        "labels": {
            "app": "spring-boot-example"
        },
        "name": "example-spring-boot",
        "namespace": "user-dockerimage",
        "resourceVersion": "3046097",
        "selfLink": "/api/v1/namespaces/user-dockerimage/services/example-spring-boot",
        "uid": "18dea209-d08a-11e8-a406-42010a840034"
    },
    "spec": {
        "clusterIP": "10.39.240.212",
        "externalTrafficPolicy": "Cluster",
        "ports": [
            {
                "nodePort": 30100,
                "port": 80,
                "protocol": "TCP",
                "targetPort": 8080
            }
        ],
        "selector": {
            "app": "spring-boot-example"
        },
        "sessionAffinity": "None",
        "type": "LoadBalancer"
    },
    "status": {
        "loadBalancer": {
            "ingress": [
                {
                    "ip": "104.199.26.127"
                }
            ]
        }
    }
}

```

Mit dem entsprechenden Befehl können Sie auch die Details zu einem Pod anzeigen:
```
$ kubectl get pod example-spring-boot-3-nwzku -o json
```

**Note:** Zuerst den pod Namen aus Ihrem Projekt abfragen (`kubectl get pods`) und im oberen Befehl ersetzen.

Über den `selector` Bereich im Service wird definiert, welche Pods (`labels`) als Endpoints dienen. Dazu können die entsprechenden Konfigurationen von Service und Pod zusammen betrachtet werden.

Service (`kubectl get service <Service Name> -o json`):
```
...
"selector": {
    "app": "example-spring-boot",
},

...
```

Pod (`kubectl get pod <Pod Name>`):
```
...
"labels": {
    "app": "example-spring-boot",
},
...
```

Diese Verknüpfung ist besser mittels `kubectl describe` Befehl zu sehen:
```
$ kubectl describe service example-spring-boot
```

```
Name:                     example-spring-boot
Namespace:                philipona
Labels:                   app=spring-boot-example
Annotations:              <none>
Selector:                 app=spring-boot-example
Type:                     LoadBalancer
IP:                       10.39.240.212
LoadBalancer Ingress:     104.199.26.127
Port:                     <unset>  80/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30100/TCP
Endpoints:                10.36.0.8:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age    From                Message
  ----    ------                ----   ----                -------
  Normal  EnsuringLoadBalancer  7m20s  service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   6m28s  service-controller  Ensured load balancer

```

Unter Endpoints finden Sie nun den aktuell laufenden Pod.

---

## Zusatzaufgabe für Schnelle ;-)

Schauen Sie sich die erstellten Ressourcen mit `kubectl get [ResourceType] [Name] -o json` und `kubectl describe [ResourceType] [Name]` aus dem ersten Namespace `[USER]-example1` an.

---

**Ende Lab 4**

<p width="100px" align="right"><a href="05_create_route.md">Routen erstellen →</a></p>

[← zurück zur Übersicht](../README.md)
