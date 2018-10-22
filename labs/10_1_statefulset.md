# Lab 10.1: StatefulSet

Zustandslose Applikationen oder solche, deren Zustand in einem Backend abgebildet werden kann, können problemlos mit Deployments gehandhabt werden. Den Zustand so abzubilden ist allerdings nicht immer möglich, weshalb StatefulSets entwickelt wurden. StatefulSets lösen dieses komplexe Problem, indem sie nicht nur den Storage implementieren, sondern auch dafür sorgen, dass eine gewünschte Reihenfolge eingehalten oder die Identität eines Pods immer dieselbe bleibt.

Weitere Informationen können der [Kubernetes StatefulSet Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) oder [diesem auf opensource.com erschienenen Artikel](https://opensource.com/article/17/2/stateful-applications) entnommen werden.

---

**Ende Lab 10.1**

<p width="100px" align="right"><a href="10_2_daemonset.md">DaemonSet →</a></p>

[← zurück zur Kapitelübersicht "Weitere Konzepte"](10_additional_concepts.md)

[← zurück zur Gesamtübersicht](../README.md)
