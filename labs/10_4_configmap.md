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
```
kann nun verifiziert werden ob die ConfigMap erfolgreich angelegt wurde.

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
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
```


```
    volumes:
    - name: config-volume
      configMap:
        name: javaconfiguration
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
