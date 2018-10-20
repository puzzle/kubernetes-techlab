# Lab 10.2: DaemonSet

Ein DaemonSet funktioniert praktisch identisch wie ein normales Deployment, stellt aber sicher, dass auf jedem (oder bestimmten) der Nodes genau ein Pod läuft. Wird ein Node hinzugefügt, deployt das DaemonSet automatisch einen Pod darauf. Wird das DaemonSet gelöscht, werden auch sämtliche Pods gelöscht die es erstellt hat.

Der Einsatz eines DaemonSet eignet sich hervorragend, um z.B. einen Daemon zum Sammeln von Logs laufen zu lassen, wie fluentd, logstash oder den Splunk-Forwarder.

Weitere Informationen können der [Kubernetes DaemonSet Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) entnommen werden.

---

**Ende Lab 10.2**

<p width="100px" align="right"><a href="10_3_jobs.md">Jobs →</a></p>

[← zurück zur Kapitelübersicht "Weitere Konzepte"](10_additional_concepts.md)

[← zurück zur Gesamtübersicht](../README.md)
