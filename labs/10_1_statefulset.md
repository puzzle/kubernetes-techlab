# Lab 10.1: StatefulSet

Zustandslose Applikationen oder solche, deren Zustand in einem Backend abgebildet werden kann, können problemlos mit Deployments gehandhabt werden. Den Zustand so abzubilden ist allerdings nicht immer möglich, weshalb StatefulSets entwickelt wurden. StatefulSets lösen dieses komplexe Problem, indem sie nicht nur den Storage implementieren, sondern auch dafür sorgen, dass eine gewünschte Reihenfolge eingehalten oder die Identität eines Pods immer dieselbe bleibt.

## Konsistente Hostnamen
Im Vergleich zu Deployments, in denen ein Zufalls-Hash im Namen des Pods (Hostname innerhalb des Systems) benutzt, verwenden Statefulsets einen fest konfigurierten Namen.
Beispiel von rabbitmq-cluster mit drei Knoten als Pods:

```
rabbitmq-0
rabbitmq-1
rabbitmq-2
```

## Skalierung
Weiterhin verhält sich die Skalierung eines eines Statefulset anders. Beim Hochskalieren von 3 auf 5 könnten beim Deployment, je nach Konfiguration 2 zusätzliche Pods auf einmal gestartet werden. Beim Statefulset läuft das "geregelt" ab.
Beispiel anhand von Rabbitmq

1. Skalierung mittels `kubectl scale deployment rabbitmq --replicas=5`
1. `rabbitmq-3` wird gestartet
1. wenn `rabbitmq-3` fertig (Zustand: "Ready", siehe Readiness-Probe), wird `rabbitmq-4` gestartet

Beim Herunterskalieren läuft es in umgekehrter Reihenfolge ab. Es wird solange gewartet bis der "jüngste" Pod beendet wird, bevor der nächste beendet wird.
Reihenfolge beim Herunterskalieren: `rabbitmq-4`, `rabbitmq-3`, usw.


## Update-Verhalten / Rollout einer neuen Software
Bei einem Update der Anwendung wird ebenfalls mit dem jüngsten Pod angefangen und erst nach erfolgreichem Start mit dem zweijüngsten Pod weitergemacht.

1. Jüngster Pod wird beendet
1. Neuer Pod mit neuem Image wird gestartet
1. Bei erfolgreichem "readinessProbe" wird der zweitjüngste Pod beendet
1. etc...

Schlägt der Start eines aktualisierten Pods fehl, bleibt das Update/Rollout stehen, damit keine weiteren Pods und damit die komplette Architektur der Applikation behindert wrd.

## Trivia
Da Statefulsets vorhersagbare Namen der Pods besitzen und Namen wiederverwendet werden, besteht die Möglichkeit dass mit der Skalierung neue Volumes (via PVCs) aus einer StorageClass dynamisch erzeugt werden.
Eine 1-zu-1 Beziehung zwischen Pods und Volumes ist somit gegeben.
Durch das Setzen einer _Partition_ kann man Updates in zwei Schritten gezielt ausführen.

## Fazit
Das kontrolliert "vorhersagbare" Verhalten wird bei Applikationen wie __rabbitmq__ oder __etcd__, in denen bei der Cluster-Bildung eineindeutige Namen benutzt werden, genutzt. Mit Anwendungen des Typs *Deployments* wäre das nicht möglich.


## Aufgabe: LAB10.1

### Statefulset erstellen
1. Erstellen eines Statefulsets mittels YAML-Datei _nginx-sfs.yaml_ :
```YAML
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-cluster
spec:
  serviceName: "nginx"
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.12
        ports:
        - containerPort: 80
          name: nginx
```

1. Starten des Statefulsets
```bash
kubectl create -f nginx-sfs.yaml
```

### Statefulset skalieren

1. Zur Beobachtung ein zweites Konsolenfenster öffnen, statefulsets anzeigen lassen und Pods beobachten:
```bash
kubectl get statefulset
kubectl get pods -l app=nginx -w
```

1. Statefulset hochskalieren
```bash
kubectl scale statefulset nginx-cluster --replicas=3
```

### Statefulset Image aktualisieren

1. Zur Beobachtung der Veränderungen der Pods, bitte ein zweites Konsolenfenster öffnen und folgendes ausführen:
```bash
kubectl get pods -l app=nginx -w
```

1. Neue Version des Images für das statefulset setzen
```bash
kubectl set image statefulset nginx-cluster nginx=nginx:latest
```

1. Neue Version der Software im Statefulset zurückrollen
```bash
kubectl rollout undo statefulset nginx-cluster
```

Weitere Informationen können der [Kubernetes StatefulSet Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) oder [diesem auf opensource.com erschienenen Artikel](https://opensource.com/article/17/2/stateful-applications) entnommen werden.


---

**Ende Lab 10.1**

<p width="100px" align="right"><a href="10_2_daemonset.md">DaemonSet →</a></p>

[← zurück zur Kapitelübersicht "Weitere Konzepte"](10_additional_concepts.md)

[← zurück zur Gesamtübersicht](../README.md)
