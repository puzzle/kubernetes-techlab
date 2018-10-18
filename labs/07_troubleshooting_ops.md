# Lab 7: Troubleshooting, was ist im Pod?

In diesem Lab wird aufgezeigt, wie man im Fehlerfall und Troubleshooting vorgehen kann und welche Tools einem dabei zur Verfügung stehen.

## In Container einloggen

Wir verwenden dafür wieder das Projekt aus [Lab 4](04_deploy_dockerimage.md) `[USER]-dockerimage`. **Tipp:** `kubectl config set-context $(kubectl config current-context) --namespace=[USER]--dockerimage`

Laufende Container werden als unveränderbare Infrastruktur behandelt und sollen generell nicht modifiziert werden. Dennoch gibt es Usecases, bei denen man sich in die Container einloggen muss. Zum Beispiel für Debugging und Analysen.

## Aufgabe: LAB7.1

Mit Kubernetes können Remote Shells in die Pods geöffnet werden ohne dass man darin vorgängig SSH installieren müsste. Die kann mittels `kubectl exec` command erreicht werden. Der Befehl ist generell dafür da, Commands in Container auszuführen. Mit den Paramentern `-it` kann allerdings die Shell und Verbindung offen behalten werden.

Wählen Sie mittels `kubectl get pods` einen Pod aus und führen Sie den folgenden Befehl aus:
```
$ kubectl exec -it [POD] -- /bin/bash
```

Sie können nun über diese Shell Analysen im Container ausführen:

```
bash-4.2$ ls -la
total 40
drwxr-xr-x 1 default root 4096 Jun 21  2016 .
drwxr-xr-x 1 default root 4096 Jun 21  2016 ..
drwxr-xr-x 6 default root 4096 Jun 21  2016 .gradle
drwxr-xr-x 3 default root 4096 Jun 21  2016 .pki
drwxr-xr-x 9 default root 4096 Jun 21  2016 build
-rw-r--r-- 1 root    root 1145 Jun 21  2016 build.gradle
drwxr-xr-x 3 root    root 4096 Jun 21  2016 gradle
-rwxr-xr-x 1 root    root 4971 Jun 21  2016 gradlew
drwxr-xr-x 4 root    root 4096 Jun 21  2016 src
```

## Aufgabe: LAB7.2

Einzelne Befehle innerhalb des Containers können über `kubectl exec` ausgeführt werden:

```
$ oc exec [POD] env
```


```
$ kubectl exec example-spring-boot-69b658f647-xnm94 env
PATH=/opt/app-root/src/bin:/opt/app-root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=example-spring-boot-4-8mbwe
KUBERNETES_SERVICE_PORT_DNS_TCP=53
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=172.30.0.1
KUBERNETES_PORT_53_UDP_PROTO=udp
KUBERNETES_PORT_53_TCP=tcp://172.30.0.1:53
...
```

## Logfiles anschauen

Die Logfiles zu einem Pod können wie folgt angezeigt werden.

```
$ kubectl logs [POD]
```
Der Parameter `-f` bewirkt analoges Verhalten wie `tail -f`

Befindet sich ein Pod im Status **CrashLoopBackOff** bedeutet dies, dass er auch nach wiederholtem Restarten nicht erfolgreich gestartet werden konnte. Die Logfiles können auch wenn der Pod nicht läuft mit dem folgenden Befehl angezeigt werden.

 ```
$ kubectl logs -p [POD]
```


## Aufgabe: LAB7.3 Port Forwarding

Kubernetes erlaubt es, beliebige Ports von der Entwicklungs-Workstation auf einen Pod weiterzuleiten. Dies ist z.B. nützlich, um auf Administrationskonsolen, Datenbanken, usw. zuzugreifen, die nicht gegen das Internet exponiert werden und auch sonst nicht erreichbar sind. Der Zugriff eines Port Forwarding erfolgt über den Kubernetes Master und wird vom Client bis zum Master über HTTPS getunnelt. Dies erlaubt es auch dann auf Kubernetes Platformen zuzugreifen, wenn sich restriktive Firewalls und/oder Proxies zwischen Workstation und Kubernetes befinden.

Übung: Auf die Spring Boot Metrics aus [Lab 4](04_deploy_dockerimage.md) zugreifen.

```
kubectl get pod --namespace="[USER]-dockerimage"
kubectl port-forward example-spring-boot-1-xj1df 9000:9000 --namespace="[USER]-dockerimage"
```

Nicht vergessen den Pod Namen an die eigene Installation anzupassen. Falls installiert kann dafür Autocompletion verwendet werden.

Die Metrics können nun unter folgendem Link abgerufen werden: [http://localhost:9000/metrics/](http://localhost:9000/metrics/) Die Metrics werden Ihnen als JSON angezeigt. Mit demselben Konzept können Sie nun bspw. mit Ihrem lokalen SQL Client auf eine Datenbank verbinden.

Unter folgendem Link sind weiterführende Informationen zu Port Forwarding zu finden: https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

**Note:** Der `kubectl port-forward`-Prozess wird solange weiterlaufen, bis er vom User abgebrochen wird. Sobald das Port-Forwarding also nicht mehr benötigt wird, kann er mit ctrl+c gestoppt werden.

---

**Ende Lab 7**

<p width="100px" align="right"><a href="08_database.md">Datenbank deployen und anbinden →</a></p>

[← zurück zur Übersicht](../README.md)
