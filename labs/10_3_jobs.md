# Lab 10.3: Jobs

Jobs unterscheiden sich von normalen Deployments insb. darin, dass sie eine zeitlich begrenzte Operation ausführen und über deren Resultat berichten sollen, sobald die Operation beendet wurde. Ein Job erstellt dafür einen Pod und führt die definierte Operation bzw. Befehl aus. Es muss sich dabei nicht zwingend um nur einen Pod handeln, sondern kann auch mehrere beinhalten. Wird ein Job gelöscht, werden auch die vom Job gestarteten (und wieder beendeten) Pods gelöscht.

Ein Job eignet sich also bspw. dafür, sicherzustellen, dass ein Pod verlässlich bis zu dessen Vervollständigung ausgeführt wird. Schlägt ein Pod fehl, zum Beispiel wegen eines Node-Fehlers, startet der Job einen neuen Pod. Ein Job kann auch dazu verwendet werden, mehrere Pods parallel zu starten.

Detailliertere Informationen können der [Kubernetes Jobs Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) entnommen werden.


## Aufgabe: LAB10.1 Job für MySQL-Dump erstellen

Ähnlich wie in [Aufgabe 8.4](08_database.md#aufgabe-lab84-dump-auf-mysql-db-einspielen) wollen wir nun einen Dump der laufenden MySQL-Datenbank erstellen, aber ohne uns in den Pod einloggen zu müssen.

Schauen wir uns zuerst die Job-Ressource an, die wir erstellen wollen. Sie ist unter [labs/10_data/job_mysql-dump.yaml](https://github.com/puzzle/kubernetes-techlab/blob/master/labs/10_data/job_mysql-dump.yaml) zu finden.
Unter `.spec.template.spec.containers[0].image` sehen wir, dass wir dasselbe Image verwenden wie für die laufende Datenbank selbst. Wir starten anschliessend aber keine Datenbank, sondern wollen einen `mysqldump`-Befehl ausführen, wie unter `.spec.template.spec.containers[0].command` aufgeführt. Dazu verwenden wir, wie schon im Datenbank-Deployment, dieselben Umgebungsvariablen, um Hostname, User oder Passwort innerhalb des Pods definieren zu können. Die `MYSQL_PASSWORD`-Umgebungsvariable bezieht dabei den Wert aus dem Secret, welches bereits für den Datenbank-Pod selbst gebraucht wird. So ist auf einfache Weise sichergestellt, dass der Dump mit denselben Credentials ausgeführt werden kann.

Erstellen wir nun also unseren Job:

```
$ kubectl create -f ./labs/10_data/job_mysql-dump.yaml --namespace [USER]-dockerimage
```

Überprüfen wir, ob der Job erfolgreich war:

```
$ kubectl describe jobs/mysql-dump --namespace [USER]-dockerimage
```

Den ausgeführten Pod können wir wie folgt anschauen:

```
$ kubectl get pods --namespace [USER]-dockerimage
```

Um alle Pods, welche zu einem Job gehören, in maschinenlesbarer Form auszugeben, kann bspw. folgender Befehl verwendet werden:

```
$ kubectl get pods --selector=job-name=mysql-dump --output=jsonpath={.items..metadata.name} --namespace [USER]-dockerimage
```


## Cron Jobs

Ein Kubernetes Cron Job ist nichts anderes als eine Ressource, welche zu definierten Zeitpunkten einen Job erstellt, welcher wiederum wie gewohnt einen Pod startet um einen Befehl auszuführen. Typische Anwendungsbeispiele sind ein Cleanup-Job, welcher für einen laufenden Pod alte Daten aufräumen soll, oder ein Job für regelmässiges Erstellen und Sichern eines Datenbank-Dumps.

Weitere Informationen können der [Kubernetes CronJob Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) entnommen werden.

---

**Ende Lab 10.3**

<p width="100px" align="right"><a href="10_4_configmap.md">ConfigMap →</a></p>

[← zurück zur Kapitelübersicht "Weitere Konzepte"](10_additional_concepts.md)

[← zurück zur Gesamtübersicht](../README.md)
