# Lab 5: Unseren Service mittels online verfügbar machen

In diesem Lab werden wir die Applikation aus [Lab 4](04_deploy_dockerimage.md) über **http** vom Internet her erreichbar machen.

## Routen

Der `kubectl create deployment` Befehl aus dem vorherigen [Lab](04_deploy_dockerimage.md) erstellt zwar einen Pod aber keinen Service oder NodePort. Dafür ist der `kubectl expose` Befehl zuständig. Somit ist unser Service von *aussen* her erst erreichbar, wenn der Service exposed wurde. 


## Aufgabe: LAB5.1

Vergewissern Sie sich, dass Sie sich im Projekt `[USER]-dockerimage` befinden. **Tipp:** `kubectl config set-context $(kubectl config current-context) --namespace=[USER]-dockerimage`

Den Service `example-spring-boot` haben wir bereits im vorherigen Lab exposed, jedoch war die Loadbalancer IP noch auf Pending.


```
$ kubectl get services
NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
example-spring-boot   LoadBalancer   10.39.240.212   104.199.26.127   80:30100/TCP   22m
```
Im Service wurde mittlerweile die Externat-IP gesetzt. Unsere Applikation ist nun darüber verfügbar.

Rufen Sie im Browser entsprechnd http://[ExternalIP] auf


Die Applikation ist nun vom Internet her über die IP, Sie können also nun auf die Applikation zugreifen. http://[ExternalIP] auf


---

**Ende Lab 5**

<p width="100px" align="right"><a href="06_scale.md">Skalieren →</a></p>

[← zurück zur Übersicht](../README.md)
