# Lab 5: Unseren Service mittels Load Balancer online verfügbar machen

In diesem Lab werden wir die Applikation aus [Lab 4](04_deploy_dockerimage.md) über **http** vom Internet her erreichbar machen.


## Basis

Der `kubectl create deployment`-Befehl aus dem vorherigen [Lab](04_deploy_dockerimage.md) erstellt zwar einen Pod, aber keinen Service. Dafür ist der `kubectl expose`-Befehl zuständig. Somit ist unser Service von *aussen* her erst erreichbar, wenn der Service exposed wurde.

Damit machen wir nun die Applikation vom Internet her verfügbar.


## Aufgabe: LAB5.1

Mit dem folgenden Befehl wird unser Deployment über den Type LoadBalancer auf Port 80 und Pod Target Port 8080 exponiert:

```
$ kubectl expose deployment example-spring-boot --type="LoadBalancer" --name="example-spring-boot" --port=80 --target-port=8080
```

[Services](https://kubernetes.io/docs/concepts/services-networking/service/) dienen innerhalb Kubernetes als Abstraktionslayer, Einstiegspunkt und Proxy/Loadbalancer auf die dahinterliegenden Pods. Der Service ermöglicht es, innerhalb Kubernetes eine Gruppe von Pods des gleichen Typs zu finden und anzusprechen.

Als Beispiel: Wenn eine Applikationsinstanz unseres Labs die Last nicht mehr alleine verarbeiten kann, können wir die das Deployment hochskalieren, also weitere Pods unserer Applikation deployen. Kubernetes mappt diese als Endpoints automatisch zum Service. Sobald die Pods bereit sind, werden Requests auch auf die neuen Pods verteilt.

**Note:** Die Applikation kann aktuell von aussen noch nicht erreicht werden, der Service ist ein Kubernetes-internes Konzept. Im folgenden Lab werden wir die Applikation öffentlich verfügbar machen.

Nun schauen wir uns unseren Service mal etwas genauer an:

```
$ kubectl get services
```

```bash
NAME                TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
example-spring-boot LoadBalancer   10.39.247.42   <pending>     80:30180/TCP   2s
```

**Note:** Die External IP wird erst nachträglich eingetragen, es braucht hier einen Moment bis sie verfügbar ist.

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
            "app": "example-spring-boot"
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
            "app": "example-spring-boot"
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

**Note:** Zuerst den Pod-Namen aus Ihrem Projekt abfragen (`kubectl get pods`) und im oberen Befehl ersetzen oder mittels Tab-Taste (bash completion) eintragen lassen.

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
Labels:                   app=example-spring-boot
Annotations:              <none>
Selector:                 app=example-spring-boot
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

Vergewissern Sie sich, dass Sie sich im Projekt `[USER]-dockerimage` befinden. 

**Tipp:**

Linux:
```
$ kubectl config set-context $(kubectl config current-context) --namespace=[USER]-dockerimage
```

Windows:
```
$ kubectl config set-context %KUBE_CONTEXT% --namespace=[USER]-dockerimage
```

Den Service `example-spring-boot` haben wir bereits im vorherigen Lab exponiert, jedoch war die LoadBalancer IP noch auf Pending.


```bash
$ kubectl get services
NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
example-spring-boot   LoadBalancer   10.39.240.212   104.199.26.127   80:30100/TCP   22m
```

Im Service wurde mittlerweile die EXTERNAL-IP gesetzt. Unsere Applikation ist nun darüber verfügbar.
Wie Sie am Output sehen, ist unser Service über eine IP und Port erreichbar (104.199.26.127.20:80).

**Note:** Ihre IP kann unterschiedlich sein.

**Note:** Service IPs bleiben während ihrer Lebensdauer immer gleich.

Rufen Sie im Browser entsprechend http://[ExternalIP] auf.

Jetzt ist Ihre Applikation in der [Web Console](https://console.cloud.google.com/kubernetes) auch unter dem Reiter "Services" zu finden.


## Aufgabe: LAB5.2

Nebst dem im LAB5.1 beschriebenen Art wie man Services von aussen erreichbar machen kann, gibt es etliche weitere Möglichkeiten. Schauen Sie sich die Möglichkeiten unter https://kubernetes.io/docs/concepts/services-networking/service/ an.

---

## Zusatzaufgabe für Schnelle ;-)

Schauen Sie sich die erstellten Ressourcen mit `kubectl get [ResourceType] [Name] -o json` und `kubectl describe [ResourceType] [Name]` aus dem Namespace `[USER]-dockerimage` an.

---

**Ende Lab 5**

<p width="100px" align="right"><a href="06_scale.md">Skalieren →</a></p>

[← zurück zur Übersicht](../README.md)
