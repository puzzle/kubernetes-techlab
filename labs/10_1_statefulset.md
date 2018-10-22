# Lab 10.1: StatefulSet

Zustandslose Applikationen oder solche, deren Zustand in einem Backend abgebildet werden kann, können problemlos mit Deployments gehandhabt werden. Den Zustand so abzubilden ist allerdings nicht immer möglich, weshalb StatefulSets entwickelt wurden. 

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

1. Skalierung mittels `kubectl scale deployment rabbitmq --replicas=4`
1. `rabbitmq-3` wird gestartet
1. wenn `rabbitmq-3` fertig (Zustand: "Ready", siehe Readiness-Probe), wird `rabbitmq-4` gestartet

Beim Herunterskalieren läuft es in umgekehrter Reihenfolge ab. Es wird solange gewartet bis der "jüngste" Pod beendet wird, bevor der nächste beendet wird.
Reihenfolge beim Herunterskalieren: `rabbitmq-4`, `rabbitmq-3`, usw.


## Update-Verhalten / Rollout einer neuen Software
Bei einem Update der Anwendung wird ebenfalls mit dem jüngsten Pod angefangen und erst nach erfolgreichem Start mit dem zweijüngsten Pod weitergemacht.

1. Jüngster Pod wird beendet
1. Neuer Pod mit neuem Image wird gestartet
1. Bei erfolgreichem "Readiness-Probe" wird der zweitjüngste Pod beendet
1. etc...

Schlägt der Start eines aktualisierten Pods fehl, bleibt das Update/Rollout stehen, damit keine weiteren Pods und damit die komplette Architektur der Applikation behindert wrd.

## Trivia
Da Statefulsets vorhersagbare Namen der Pods besitzen und Namen wiederverwendet werden, besteht die Möglichkeit dass mit der Skalierung neue Volumes (via PVCs) aus einer StorageClass dynamisch erzeugt werden.
Eine 1-zu-1 Beziehung zwischen Pods und Volumes ist somit gegeben.

## Fazit
Das kontrolliert "vorhersagbare" Verhalten wird bei Applikationen wie __rabbitmq__ oder __etcd__, in denen bei der Cluster-Bildung eineindeutige Namen benutzt werden, genutzt. Mit Anwendungen des Typs *Deployments* wäre das nicht möglich.


FIXME

Weitere Informationen können der [Kubernetes StatefulSet Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) entnommen werden.

---

**Ende Lab 10.1**

<p width="100px" align="right"><a href="10_2_daemonset.md">DaemonSet →</a></p>

[← zurück zur Kapitelübersicht "Weitere Konzepte"](10_additional_concepts.md)

[← zurück zur Gesamtübersicht](../README.md)
