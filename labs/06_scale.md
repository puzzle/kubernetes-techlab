# Lab 6: Pod Scaling, Readiness Probe und Self Healing

In diesem Lab zeigen wir auf, wie man Applikationen in Kubernetes skaliert. Des Weiteren zeigen wir, wie Kubernetes dafür sorgt, dass jeweils die Anzahl erwarteter Pods gestartet wird und wie eine Applikation der Plattform zurückmelden kann, dass sie bereit für Requests ist.

## Example Applikation hochskalieren

Erstellen Sie in Ihrem Namespace ein neues Deployment


```
$ kubectl create deployment appuio-php-docker --image=appuio/example-php-docker-helloworld --namespace [USER]-dockerimage
```

und stellen den Service zur Verfügung (expose)

```
$ kubectl expose deployment appuio-php-docker --type="NodePort" --name="appuio-php-docker" --target-port=8080 --namespace [USER]-dockerimage
```

Wenn wir unsere Example Applikation skalieren wollen, müssen wir unserem Deployment mitteilen, dass wir bspw. stets 3 Replicas des Images am Laufen haben wollen.

Schauen wir uns mal das Replicaset etwas genauer an:

```
$ kubectl get replicasets --namespace [USER]-dockerimage

NAME                           DESIRED   CURRENT   READY   AGE
appuio-php-docker-86d9d584f8   1         1         1       110s
```

Für mehr Details:

```
$ kubectl get replicaset appuio-php-docker-86d9d584f8 -o json --namespace [USER]-dockerimage
```

Das Replicaset sagt uns, wieviele Pods wir erwarten (spec) und wieviele aktuell deployt sind (status).

## Aufgabe: LAB6.1 skalieren unserer Beispiel Applikation
Nun skalieren wir unsere Example Applikation auf 3 Replicas:

```
$ kubectl scale deployment appuio-php-docker --replicas=3 --namespace [USER]-dockerimage
```

Überprüfen wir die Anzahl Replicas auf dem ReplicationController:

```
$ kubectl get replicasets --namespace [USER]-dockerimage

NAME                           DESIRED   CURRENT   READY   AGE
appuio-php-docker-86d9d584f8   3         3         1       4m33s

```

und zeigen entsprechend die Pods an:

```
$ kubectl get pods --namespace [USER]-dockerimage
NAME                                 READY   STATUS    RESTARTS   AGE
appuio-php-docker-86d9d584f8-7vjcj   1/1     Running   0          5m2s
appuio-php-docker-86d9d584f8-hbvlv   1/1     Running   0          31s
appuio-php-docker-86d9d584f8-qg499   1/1     Running   0          31s

```

Zum Schluss schauen wir uns den Service an. Der sollte jetzt alle drei Endpoints referenzieren:

```bash
$ kubectl describe service appuio-php-docker --namespace [USER]-dockerimage
Name:                     appuio-php-docker
Namespace:                philipona-scale
Labels:                   app=appuio-php-docker
Annotations:              <none>
Selector:                 app=appuio-php-docker
Type:                     LoadBalancer
IP:                       10.39.245.205
Port:                     <unset>  80/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32193/TCP
Endpoints:                10.36.0.10:8080,10.36.0.11:8080,10.36.0.9:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
  Normal  EnsuringLoadBalancer  2s    service-controller  Ensuring load balancer
```

Skalieren von Pods innerhalb eines Services ist sehr schnell, da Kubernetes einfach eine neue Instanz des Docker Images als Container startet.

**Tipp:** Kubernetes unterstützt auch Autoscaling, die Dokumentation dazu ist unter dem folgenden Link zu finden: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/


## Unterbruchsfreies Skalieren überprüfen

Mit dem folgenden Befehl können Sie nun überprüfen, ob Ihr Service verfügbar ist, während Sie hoch und runter skalieren.
Ersetzen Sie dafür `[ip]` mit Ihrer definierten External IP:

**Tipp:** kubectl get service appuio-php-docker --namespace [USER]-dockerimage

```
Linux:
while true; do sleep 1; curl -s http://[LoadBalance Ingress ip]/pod/; date "+ TIME: %H:%M:%S,%3N"; done
```

```
Windows (ab Powershell-Version 3.0):
while(1) { ^
	Start-Sleep -s 1 ^
	Invoke-RestMethod http://35.205.165.31/pod/ ^
	Get-Date -Uformat "+ TIME: %H:%M:%S,%3N" ^
} ^
```

und skalieren Sie von **3** Replicas auf **1**.
Der Output zeigt jeweils den Pod an, der den Request verarbeitete:

```
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:07,289
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:08,357
POD: appuio-php-docker-86d9d584f8-hbvlv TIME: 17:33:09,423
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:10,494
POD: appuio-php-docker-86d9d584f8-qg499 TIME: 17:33:11,559
POD: appuio-php-docker-86d9d584f8-hbvlv TIME: 17:33:12,629
POD: appuio-php-docker-86d9d584f8-qg499 TIME: 17:33:13,695
POD: appuio-php-docker-86d9d584f8-hbvlv TIME: 17:33:14,771
POD: appuio-php-docker-86d9d584f8-hbvlv TIME: 17:33:15,840
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:16,912
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:17,980
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:19,051
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:20,119
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:21,182
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:22,248
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:23,313
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:24,377
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:25,445
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:33:26,513
```
Die Requests werden an die unterschiedlichen Pods geleitet, sobald man runterskaliert auf einen Pod, gibt dann nur noch einer Antwort

Was passiert nun, wenn wir nun während dem der While Befehl oben läuft, ein neues Deployment starten:

**Tipp:** in Gitbash ausführen. Powershell funktioniert nicht.

```
$ kubectl patch deployment appuio-php-docker -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"date\":\"`date +'%s'`\"}}}}}" --namespace [USER]-dockerimage
```
Währen einer kurzen Zeit gibt die öffentliche Route keine Antwort
```
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:37:24,121
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:37:25,189
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:37:26,262
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:37:27,328
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:37:28,395
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:37:29,459
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:37:30,531
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:37:31,596
POD: appuio-php-docker-86d9d584f8-7vjcj TIME: 17:37:32,662
# keine Antwort
POD: appuio-php-docker-f4c5dd8fc-4nx2t TIME: 17:37:33,729
POD: appuio-php-docker-f4c5dd8fc-4nx2t TIME: 17:37:34,794
POD: appuio-php-docker-f4c5dd8fc-4nx2t TIME: 17:37:35,862
POD: appuio-php-docker-f4c5dd8fc-4nx2t TIME: 17:37:36,929
POD: appuio-php-docker-f4c5dd8fc-4nx2t TIME: 17:37:37,995
POD: appuio-php-docker-f4c5dd8fc-4nx2t TIME: 17:37:39,060
POD: appuio-php-docker-f4c5dd8fc-4nx2t TIME: 17:37:40,118
POD: appuio-php-docker-f4c5dd8fc-4nx2t TIME: 17:37:41,187
```

In unserem Beispiel verwenden wir einen sehr leichtgewichtigen Pod.
Das Verhalten ist ausgeprägter, wenn der Container länger braucht bis er Requests abarbeiten kann.
Bspw. Java Applikation von [Lab 4](04_deploy_dockerimage.md): **Startup: 30 Sekunden**

```
Pod: example-spring-boot-2-73aln TIME: 16:48:25,251
Pod: example-spring-boot-2-73aln TIME: 16:48:26,305
Pod: example-spring-boot-2-73aln TIME: 16:48:27,400
Pod: example-spring-boot-2-73aln TIME: 16:48:28,463
Pod: example-spring-boot-2-73aln TIME: 16:48:29,507
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
 TIME: 16:48:33,562
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
 TIME: 16:48:34,601
 ...
Pod: example-spring-boot-3-tjdkj TIME: 16:49:20,114
Pod: example-spring-boot-3-tjdkj TIME: 16:49:21,181
Pod: example-spring-boot-3-tjdkj TIME: 16:49:22,231

```

Es kann dann sogar sein, dass der Service gar nicht mehr online ist und der Routing Layer ein **503 Error** zurück gibt.

Im Folgenden Kapitel wird beschrieben, wie Sie Ihre Services konfigurieren können, dass unterbruchsfreie Deployments möglich werden.


## Unterbruchsfreies Deployment mittels Readiness Probe und Rolling Update

Die Update Strategie [Rolling](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/) ermöglicht unterbruchsfreie Deployments. Damit wird die neue Version der Applikation gestartet, sobald die Applikation bereit ist, werden Request auf den neuen Pod geleitet und die alte Version undeployed.

Zusätzlich kann mittels [Container Health Checks](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) die deployte Applikation der Plattform detailliertes Feedback über ihr aktuelles Befinden geben.

Grundsätzlich gibt es zwei Checks, die implementiert werden können:

- Liveness Probe
  - sagt aus, ob ein laufender Container immer noch sauber läuft.
- Readiness Probe
  - gibt Feedback darüber, ob eine Applikation bereit ist, um Requests zu empfangen. Ist v.a. im Rolling Update relevant.

Diese beiden Checks können als HTTP Check, Container Execution Check (Shell Script im Container) oder als TCP Socket Check implementiert werden.

In unserem Beispiel soll die Applikation der Plattform sagen, ob sie bereit für Requests ist. Dafür verwenden wir die Readiness Probe. Unsere Beispielapplikation gibt auf der folgenden URL auf Port 8080  ein Status Code 200 zurück, sobald die Applikation bereit ist.
```
http://[ip]/health/
```

## Aufgabe: LAB6.3

Im Deployment (dc) definieren im Abschnitt der Rolling Update Strategie, dass bei einem Update die App immer verfügbar sein soll: `maxUnavailable: 0%`

Dies kann in der Deployment Config (dc) konfiguriert werden:

**YAML:**
```
...
spec:
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0%
    type: RollingUpdate
...
```

Das Deployment Config kann direkt über `kubectl` editiert werden.
```
$ kubectl edit deployment appuio-php-docker --namespace [USER]-dockerimage
```

Oder im JSON-Format editieren:
```
$ kubectl edit deployment appuio-php-docker -o json --namespace [USER]-dockerimage
```
**json**
```
"strategy": {
    "rollingUpdate": {
        "maxSurge": "25%",
        "maxUnavailable": "0%"
    },
    "type": "RollingUpdate"
},

```

Die Readiness Probe muss im Deployment hinzugefügt werden, und zwar unter:

spec --> template --> spec --> containers oberhalb von `resources: {  }`

**YAML:**

```bash
...
        readinessProbe:
          httpGet:
            path: /health/
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 10
          timeoutSeconds: 1
        resources: {  }
...
```

**json:**

```bash
...
                        "resources": {},
                        "readinessProbe": {
                            "httpGet": {
                                "path": "/health/",
                                "port": 8080,
                                "scheme": "HTTP"
                            },
                            "initialDelaySeconds": 10,
                            "periodSeconds": 10,
                            "timeoutSeconds": 1
                        },

...
```

Passen Sie das entsprechend analog oben an.

Die Konfiguration unter Container muss dann wie folgt aussehen:
**YAML:**

```bash
      containers:
      - image: appuio/example-php-docker-helloworld
        imagePullPolicy: Always
        name: example-php-docker-helloworld
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /health/
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File

```

**json:**

```bash
                "containers": [
                    {
                        "image": "appuio/example-php-docker-helloworld",
                        "imagePullPolicy": "Always",
                        "name": "example-php-docker-helloworld",
                        "readinessProbe": {
                            "failureThreshold": 3,
                            "httpGet": {
                                "path": "/health/",
                                "port": 8080,
                                "scheme": "HTTP"
                            },
                            "initialDelaySeconds": 10,
                            "periodSeconds": 10,
                            "successThreshold": 1,
                            "timeoutSeconds": 1
                        },
                        "resources": {},
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File"
                    }
                ],
```

Verifizieren Sie während eines Deployment der Applikation, ob nun auch ein Update der Applikation unterbruchsfrei verläuft:

Einmal pro Sekunde ein Request:

```bash
while true; do sleep 1; curl -s http://[ip]/pod/; date "+ TIME: %H:%M:%S,%3N"; done
```

Starten des Deployments:

```bash
$ kubectl patch deployment appuio-php-docker -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"date\":\"`date +'%s'`\"}}}}}" --namespace [USER]-dockerimage
```


## Self Healing

Über das Replicaset haben wir nun der Plattform mitgeteilt, dass jeweils **n** Replicas laufen sollen. Was passiert nun, wenn wir einen Pod löschen?

Suchen Sie mittels `kubectl get pods` einen Pod im Status "running" aus, den Sie *killen* können.

Starten sie in einem eigenen Terminal den folgenden Befehl (anzeige der Änderungen an Pods)
```
kubectl get pods -w --namespace [USER]-dockerimage
```
Löschen Sie im anderen Terminal einen Pod mit folgendem Befehl
```
kubectl delete pod appuio-php-docker-3-788j5
```


Kubernetes sorgt dafür, dass wieder **n** Replicas des genannten Pods laufen.


---

**Ende Lab 6**

<p width="100px" align="right"><a href="07_troubleshooting_ops.md">Troubleshooting, was ist im Pod? →</a></p>

[← zurück zur Übersicht](../README.md)
