# Lab 8: Datenbank anbinden

Etliche Applikationen sind in irgendeiner Art stateful und speichern Daten persistent ab. Sei dies in einer Datenbank oder als Files auf einem Filesystem oder Objectstore. In diesem Lab werden wir in unserem Namespace einen MySQL Service anlegen und an unsere Applikation anbinden, sodass mehrere Applikationspods auf die gleiche Datenbank zugreifen können.

Für dieses Beispiel verwenden wir das Spring Boot Beispiel aus [LAB 4](04_deploy_dockerimage.md), `[USER]-dockerimage`.

**Tipp:**

```
Linux:
$ kubectl config set-context $(kubectl config current-context) --namespace=[USER]-dockerimage
```

```
Windows:
$ kubectl config set-context %KUBE_CONTEXT% --namespace=[USER]-dockerimage
```

## Aufgabe: LAB8.1: MySQL Service anlegen

Für unser Beispiel legen wir als ersten ein sogenanntes Secret an, in welchem wir das Passwort des Users für den Zugriff auf die Datenbank ablegen.

```bash
$ kubectl create secret generic mysql-password --from-literal=password=mysqlpassword
secret/mysql-password created
```

Das Passwort wird weder mit `kubectl get` noch mit `kubectl describe` angezeigt.

```bash
$ kubectl get secret mysql-password -o json
{
    "apiVersion": "v1",
    "data": {
        "password": "bXlzcWxwYXNzd29yZA=="
    },
    "kind": "Secret",
    "metadata": {
        "creationTimestamp": "2018-10-16T13:36:15Z",
        "name": "mysql-password",
        "namespace": "[namespace]",
        "resourceVersion": "3156527",
        "selfLink": "/api/v1/namespaces/[namespace]/secrets/mysql-password",
        "uid": "74a7f030-d148-11e8-a406-42010a840034"
    },
    "type": "Opaque"
}
```

Der String unter data --> password ist base64 encodiert und kann einfach decodiert werden: 

```bash
$ echo "bXlzcWxwYXNzd29yZA=="| base64 -d
mysqlpassword
```

**Note:** das Secret ist nicht verschlüsselt! Für das sichere Aufbewahren von Secrets eignen sich sogenannte Vaults beispielsweise: <https://www.vaultproject.io/>

Weiter erstellen wir ein secret für das MySQL Root Passwort.

```bash
$ kubectl create secret generic mysql-root-password --from-literal=password=mysqlrootpassword
secret/mysql-root-password created
```

Als nächstes legen wir das Deployment und den Service an. In unserem Beispiel verwenden wir zu Beginn eine Datenbank ohne persistent Storage angebunden. Dies ist nur für Testumgebungen zu verwenden, da beim Restart des MySQL Pods alle Daten verloren gehen. In einem späteren Lab werden wir aufzeigen, wie wir ein Persistent Volume an die MySQL Datenbank anhängen. Damit bleiben die Daten auch bei Restarts bestehen und ist so für den produktiven Betrieb geeignet.

Wie wir in vorherigen Labs gesehen haben, können sämtliche Kubernetes Resourcen (Deployments, Services, Secrets, ...) auch als yaml oder json angezeigt werden. Nicht nur das, man kann Resourcen auch aus yaml oder json files laden und kreieren.

In diesem Fall hier kreieren wir ein Deployment inkl. Service für die MySQL Datenbank.

```
$ kubectl create -f ./labs/08_data/mysql-deployment-empty.yaml
service/springboot-mysql created
deployment.apps/springboot-mysql created
```
So bald das Docker Image MySQL:5.6 gepulled und deployed wurde, wird unter `kubectl get pods` ein weiterer Pod angezeigt.

Die definierten Umgebungsvariablen im Deployment, konfigurieren den MySQL Pod und definieren somit wie später unser Frontend darauf zugreifen kann.

## Aufgabe: LAB8.2: Applikation an die Datenbank anbinden

Standardmässig wird bei unserer example-spring-boot Applikation eine H2 Memory Datenbank verwendet. Dies kann über das Setzen der folgenden Umgebungsvariablen entsprechend auf unseren neuen MySQL Service umgestellt werden:

- SPRING_DATASOURCE_USERNAME springboot
- SPRING_DATASOURCE_PASSWORD [WERT aus dem Secret]
- SPRING_DATASOURCE_DRIVER_CLASS_NAME com.mysql.jdbc.Driver
- SPRING_DATASOURCE_URL jdbc:mysql://[Adresse des MySQL Service]/springboot?autoReconnect=true

Für die Adresse des MySQL Service können wir entweder dessen Cluster IP (`kubectl get service`) falls gesetzt oder aber dessen DNS-Namen (`<service>`) verwenden. Alle Services und Pods innerhalb eines Projektes können über DNS aufgelöst werden.

So lautet der Wert für die Variable SPRING_DATASOURCE_URL bspw.:

```bash
Name des Services: springboot-mysql

jdbc:mysql://springboot-mysql/springboot?autoReconnect=true
```

Diese Umgebungsvariablen können wir nun im Deployment example-spring-boot setzen. Nach dem **ConfigChange** (ConfigChange ist in der DeploymentConfig als Trigger registriert) wird die Applikation automatisch neu deployed. Aufgrund der neuen Umgebungsvariablen verbindet die Applikation an die MySQL DB und [Liquibase](http://www.liquibase.org/) kreiert das Schema und importiert die Testdaten.

**Note:** Liquibase ist Open Source. Es ist eine Datenbank unabhängige Library um Datenbank Änderungen zu verwalten und auf der Datenbank anzuwenden. Liquibase erkennt beim Startup der Applikation, ob DB Changes auf der Datenbank angewendet werden müssen oder nicht. Siehe Logs.

Die Umgebungsvariablen können nun im example-spring-boot Deployment wie folgt gesetzt werden:
```
$ kubectl set env deployment/example-spring-boot SPRING_DATASOURCE_USERNAME=springboot SPRING_DATASOURCE_PASSWORD=mysqlpassword SPRING_DATASOURCE_DRIVER_CLASS_NAME="com.mysql.jdbc.Driver" SPRING_DATASOURCE_URL="jdbc:mysql://springboot-mysql/springboot?autoReconnect=true"
```

oder direkt via
```
$ kubectl edit deployment example-spring-boot
```

```
$ kubectl get deployment example-spring-boot
```
```
...
      - env:
        - name: SPRING_DATASOURCE_USERNAME
          value: springboot
        - name: SPRING_DATASOURCE_PASSWORD
          value: mysqlpassword
        - name: SPRING_DATASOURCE_DRIVER_CLASS_NAME
          value: com.mysql.jdbc.Driver
        - name: SPRING_DATASOURCE_URL
          value: jdbc:mysql://springboot-mysql/springboot?autoReconnect=true

...
```

Um zu verifizieren ob dies funktioniert hat, können sie einerseits die logs des springboot Containers anschauen **Tipp:** `kubectl logs [POD]`
oder aber ein paar Hellos erfassen, und dann den Springboot Container löschen und schauen ob die Hellos nach dem restart noch da sind.

**Vorsicht:** wir der Datenbank Pod gelöscht, sind die Daten weg und müssen zuerst initialisiert werden (Restart des Springboot Containers)

## Aufgabe: LAB8.3: In MySQL Service Pod einloggen und manuell auf DB verbinden

Wie im Lab [07](07_troubleshooting_ops.md) beschrieben kann mittels `kubectl exec -it [POD] -- /bin/bash` in einen Pod eingeloggt werden:
```
$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
example-spring-boot-574544fd68-qfkcm   1/1     Running   0          2m20s
springboot-mysql-f845ccdb7-hf2x5       1/1     Running   0          31m
```

Danach in den MySQL Pod einloggen:
```
$ kubectl exec -it springboot-mysql-f845ccdb7-hf2x5 -- /bin/bash
```

Nun können Sie mittels mysql Tool auf die Datenbank verbinden und die Tabellen anzeigen:
```
$ mysql -u$MYSQL_USER -p$MYSQL_PASSWORD springboot
Warning: Using a password on the command line interface can be insecure.
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 5.6.41 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

Anschliessend können Sie mit
```
show tables;
```

alle Tabellen anzeigen.


## Aufgabe: LAB8.4: Dump auf MySQL DB einspielen

Die Aufgabe ist es, in den MySQL Pod den [Dump](https://raw.githubusercontent.com/appuio/techlab/lab-3.3/labs/data/08_dump/dump.sql) einzuspielen.


**Tipp:** Mit `kubectl cp` können Sie lokale Dateien in einen Pod kopieren.

**Achtung:** Beachten Sie, dass dabei der tar-Befehl des Betriebssystems verwendet wird. Auf UNIX-Systemen kann tar mit dem Paketmanager, auf Windows kann bspw. [cwRsync](https://www.itefix.net/cwrsync) installiert werden. Ist eine Installation von tar nicht möglich, kann stattdessen bspw. in den Pod eingeloggt und via `curl -O <URL>` der Dump heruntergeladen werden.

**Tipp:** Verwenden Sie das Tool mysql um den Dump einzuspielen.

**Tipp:** Die bestehende Datenbank muss vorgängig leer sein. Sie kann auch gelöscht und neu angelegt werden.

## Aufgabe: LAB8.5: Springboot Deployemnt Passwort als Secret

Setzen Sie analog der `MYSQL_PASSWORD`-Umgebungsvariable das Passwort für `SPRING_DATASOURCE_PASSWORD` anstelle des Plaintext-Key als Secret-Referenz.


---

## Lösung: LAB8.4

Ein ganzes Verzeichnis (dump) syncen. Darin enthalten ist das File `dump.sql`. Beachten Sie zum rsync-Befehl auch obenstehenden Tipp sowie den fehlenden trailing slash.
```
kubectl cp ./labs/08_data/dump/ springboot-mysql-f845ccdb7-hf2x5:/tmp/
```
In den MySQL Pod einloggen:

```
$ kubectl exec -it springboot-mysql-f845ccdb7-hf2x5 -- /bin/bash
```

Bestehende Datenbank löschen:
```
$ mysql -u$MYSQL_USER -p$MYSQL_PASSWORD  springboot
...
mysql> drop database springboot;
mysql> create database springboot;
mysql> exit
```
Dump einspielen:
```
$ mysql -u$MYSQL_USER -p$MYSQL_PASSWORD springboot < /tmp/dump/dump.sql
```

**Note:** Den Dump kann man wie folgt erstellen:

```
mysqldump --user=$MYSQL_USER --password=$MYSQL_PASSWORD springboot > /tmp/dump.sql
```


---

**Ende Lab 8**

<p width="100px" align="right"><a href="09_persistent_storage.md">Persistent Storage anbinden und verwenden für Datenbank →</a></p>

[← zurück zur Übersicht](../README.md)
