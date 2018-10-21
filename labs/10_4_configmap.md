# Lab 10.4: ConfigMap

ConfigMaps erlauben ähnlich wie das setzen von Umgebungsvariablen die Konfiguration für eine Applikation vom Image zu trennen und bei Laufzeit dem Pod zur Verfügung zu stellen um Applikationen innerhalb von Containern möglichst portabel zu halten.

In diesem Lab lernen Sie wie man ConfigMaps erstellt und entsprechend verwendet.

# ConfigMap in Kubernetes Namespace anlegen:

Um eine ConfigMap in einem Namespace anzulegen kann folgender Befehl verwendet werden:

```
kubectl create configmap [name der ConfigMap] [Data Source]
```
Die `[Data Source]` kann sowohl ein File, ein Verzeichnis oder eine direkte Kommandozeileneingabe sein.


## Java properties Files als ConfigMap

Ein klassisches Beispiel für ConfigMaps sind Property Files bei Java Applikationen, welche in erster Linie nicht via Umgebungsvariablen konfiguriert werden können.

Wir wechseln dafür in den Namespace des Labs 4 `$ kubectl config set-context $(kubectl config current-context) --namespace=[USER]-dockerimage`

Mit dem folgenden Befehl legen wir nun die erste ConfigMap auf Basis eines localen Files an:

```
$ kubectl create configmap javaconfiguration --from-file=labs/10_data/properties.properties 
```

Mit 

```
$ kubectl get configmaps
NAME                DATA   AGE
javaconfiguration   1      7s
```
kann nun verifiziert werden ob die ConfigMap erfolgreich angelegt wurde.

Oder mittels `$ kubectl get configmaps javaconfiguration -o json` kann der Inhalt angezeigt werden. 


## Configmap in Pod zur Verfügung stellen

Als nächstest wollen wir die ConfigMap im Pod verfügbar machen.

Grundsätzlich gibt es dafür die folgenden Möglichkeiten: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/


* ConfigMap Properties als Umgebungsvariablen im Deployment 
* Commandline Arguments via Umgebungsvariablen
* als Volumes in den Container gemountet

Im Beispiel hier wollen wir, dass das File als File auf einem Volume liegt.

dafür müssen wir entweder den Pod oder in unserem Fall das Deployment `kubectl edit deployment example-spring-boot` bearbeiten.

unter spec --> tempalte --> spec --> container

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "8"
  creationTimestamp: 2018-10-15T13:53:08Z
  generation: 8
  labels:
    app: spring-boot-example
  name: spring-boot-example
  namespace: [NAMESPACE]
  resourceVersion: "3990918"
  selfLink: /apis/extensions/v1beta1/namespaces/philipona/deployments/spring-boot-example
  uid: a5c2f455-d081-11e8-a406-42010a840034
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: spring-boot-example
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: spring-boot-example
        date: "1539788777"
    spec:
      containers:
      - env:
        - name: SPRING_DATASOURCE_USERNAME
          value: springboot
        - name: SPRING_DATASOURCE_PASSWORD
          value: mysqlpassword
        - name: SPRING_DATASOURCE_DRIVER_CLASS_NAME
          value: com.mysql.jdbc.Driver
        - name: SPRING_DATASOURCE_URL
          value: jdbc:mysql://springboot-mysql/springboot?autoReconnect=true
        image: appuio/example-spring-boot
        imagePullPolicy: Always
        name: example-spring-boot
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /etc/config
          name: config-volume
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          defaultMode: 420
          name: javaconfiguration
        name: config-volume

```


anschliessend kann im Container im file /etc/config/properties.properties auf die Werte zugegriffen werden.

```
```bash
$ kubectl exec -it [POD] -- cat /etc/config/properties.properties
key=value
key2=value2
```

Diese Property File kann nun so von der Java Applikation im Conainer gelesen und verwendet werden. Das Image bleibt dabei umgebungsneutral.

## Aufgabe: LAB10.4.1 ConfigMap Data Sources

Erstellen Sie jeweils eine ConfigMap und verwenden Sie dafür die verschiedenen Arten von Data Sources https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

Machen Sie die Werte innerhalb von Pods auf die unterschiedlichen Arten verfügbar.


---

**Ende Lab 10.4**

<p width="100px" align="right"><a href="11_helm.md">Helm →</a></p>

[← zurück zur Kapitelübersicht "Weitere Konzepte"](10_additional_concepts.md)

[← zurück zur Gesamtübersicht](../README.md)
