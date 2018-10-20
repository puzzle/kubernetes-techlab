# Lab 10.3: Jobs

Jobs unterscheiden sich von normalen Deployments insb. darin, dass sie eine zeitlich begrenzte Operation ausführen und über deren Resultat berichten sollen, sobald die Operation beendet wurde. Ein Job erstellt dafür einen Pod und führt die definierte Operation bzw. Befehl aus. Es muss sich dabei nicht zwingend um nur einen Pod handeln, sondern kann auch mehrere beinhalten. Wird ein Job gelöscht, werden auch die vom Job gestarteten (und wieder beendeten) Pods gelöscht.

Ein Job eignet sich also bspw. dafür, sicherzustellen, dass ein Pod verlässlich bis zu dessen Vervollständigung ausgeführt wird. Schlägt ein Pod fehl, zum Beispiel wegen eines Node-Fehlers, startet der Job einen neuen Pod. Ein Job kann auch dazu verwendet werden, mehrere Pods parallel zu starten.

Detailliertere Informationen können der [Kubernetes Jobs Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) entnommen werden.


## Aufgabe: LAB10.1 Job für MySQL-Dump erstellen

Ähnlich wie in [Aufgabe 8.4](08_database.md#aufgabe-lab84-dump-auf-mysql-db-einspielen) wollen wir nun einen Dump der laufenden MySQL-Datenbank erstellen, aber ohne uns in den Pod einloggen zu müssen.

FIXME


## Cron Jobs

Ein Kubernetes Cron Job ist nichts anderes als eine Ressource, welche zu definierten Zeitpunkten einen Job erstellt, welcher wiederum einen Pod startet. Ein gutes Anwendungsbeispiel ist ein Cleanup-Job, welcher für einen laufenden Pod alte Daten aufräumen soll, oder ein regelmässiges Erstellen und Sichern eines Datenbank-Dumps.

Weitere Informationen können der [Kubernetes CronJob Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) entnommen werden.

---

**Ende Lab 10.3**

<p width="100px" align="right"><a href="10_4_configmap.md">ConfigMap →</a></p>

[← zurück zur Kapitelübersicht "Weitere Konzepte"](10_additional_concepts.md)

[← zurück zur Gesamtübersicht](../README.md)
