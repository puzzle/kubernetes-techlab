# Lab 9: Persistent Storage anbinden und verwenden für Datenbank

Per se sind Daten in einem Pod nicht persistent, was u.a. auch in unserem Beispiel der Fall ist. Verschwindet also unser MySQL-Pod bspw. aufgrund einer Änderung des Images, sind die bis zuvor noch vorhandenen Daten im neuen Pod nicht mehr vorhanden. Um genau dies zu verhindern hängen wir nun Persistent Storage an unseren MySQL-Pod an.

## Aufgabe: LAB9.1:

### Storage anfordern

Das Anhängen von Persistent Storage geschieht eigentlich in zwei Schritten. Der erste Schritt beinhaltet als erstes das Erstellen eines sog. PersistentVolumeClaim(PVC) für unseren Namespace. Im Claim definieren wir u.a. dessen Namen sowie Grösse, also wie viel persistenten Speicher wir überhaupt haben wollen.

Der PersistentVolumeClaim stellt allerdings erst den Request dar, nicht aber die Ressource selbst. Er wird deshalb automatisch durch Kubernetes GKE mit einem zur Verfügung stehenden Persistent Volume verbunden, und zwar mit einem mit mindestens der angeforderten Grösse. Sind nur noch grössere Persistent Volumes vorhanden, wird eines dieser Volumes verwendet und die Grösse des Claims angepasst. Sind nur noch kleinere Persistent Volumes vorhanden, kann der Claim nicht verbunden werden und bleibt solange offen, bis ein Volume der passenden Grösse (oder eben grösser) auftaucht.


### Volume in Pod einbinden

Im zweiten Schritt wird der zuvor erstellte PVC im richtigen Pod eingebunden. In [LAB 6](06_scale.md) bearbeiteten wir das Deployment, um die Readiness Probe einzufügen. Dasselbe tun wir nun für das Persistent Volume.


Der folgende Befehl führt erstellt den PersistentVolumeClaim für ein 1Gi Volume und fordert dieses an:
```
$ kubectl create --namespace [USER]-dockerimage -f ./labs/08_data/mysql-persistent-volume-claim.yaml
```
Nun müssen wir dem mysql Deployment noch das Volume am korrekten Ort anhängen

```
$ kubectl edit deployment springboot-mysql --namespace [USER]-dockerimage
```
das Deployment muss wie folgt aussehen, volumes beachten:
```
...
    spec:
      containers:
      - env:
        - name: MYSQL_DATABASE
          value: springboot
        - name: MYSQL_USER
          value: springboot
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql-password
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql-root-password
        image: mysql:5.6
        imagePullPolicy: IfNotPresent
        name: mysql
        ports:
        - containerPort: 3306
          name: mysql
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
...
```

**Note:** Durch die veränderte Deployment deployt Kubernetes automatisch einen neuen Pod. D.h. leider auch, dass das vorher erstellte DB-Schema und bereits eingefügte Daten verloren gegangen sind.

Unsere Applikation erstellt beim Starten das DB Schema eigenständig.

**Tipp:** redeployen Sie den Applikations-Pod:

```
$ kubectl patch deployment example-spring-boot -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"date\":\"`date +'%s'`\"}}}}}" --namespace [USER]-dockerimage
```

Mit dem Befehl `kubectl get persistentvolumeclaim`, oder etwas einfacher `kubectl get pvc --namespace [USER]-dockerimage`, können wir uns nun den im Projekt frisch erstellten PersistentVolumeClaim anzeigen lassen:
```
$ kubectl get pvc --namespace [USER]-dockerimage
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    pvc-2cb78deb-d157-11e8-a406-42010a840034   1Gi        RWO            standard       11s
```
Die beiden Attribute Status und Volume zeigen uns an, dass unser Claim mit dem Persistent Volume pvc-2cb78deb-d157-11e8-a406-42010a840034 verbunden wurde.


## Aufgabe: LAB9.2: Persistenz-Test

### Daten wiederherstellen

Wiederholen Sie [Lab-Aufgabe 8.4](08_database.md#l%C3%B6sung-lab84).


### Test

Skalieren Sie nun den mysql Pod auf 0 und anschliessend wieder auf 1. Beobachten Sie, dass der neue Pod die Daten nicht mehr verliert.

---

**Ende Lab 9**

<p width="100px" align="right"><a href="10_additional_concepts.md">Weitere Konzepte →</a></p>

[← zurück zur Übersicht](../README.md)
