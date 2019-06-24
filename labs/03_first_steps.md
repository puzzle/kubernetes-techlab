# Lab 3: Erste Schritte auf der Lab Plattform

In diesem Lab werden wir gemeinsam das erste Mal mit der Lab Plattform interagieren, dies sowohl über `kubectl` wie auch über die Web Console.


## Vorbereitung Labs

Als erstes ist es sinnvoll, wenn das Git Repository des vorliegenden Labs gecloned wird, damit die nötigen Aufgaben lokal auf dem PC vorhanden sind.

```
$ cd [Git Repo Project Folder]
$ git clone https://github.com/puzzle/kubernetes-techlab.git
```

Als Alternative kann das Repository als [Zip](https://github.com/puzzle/kubernetes-techlab/archive/master.zip) heruntergeladen werden.


## Login

**Note:** Vergewissern Sie sich, dass Sie [Lab 2](02_cli.md) erfolgreich abgeschlossen haben.

Unser Kubernetes Cluster der Techlab Plattform läuft auf cloudscale.ch und wurde mit Hilfe von [Rancher](https://rancher.com/) erstellt. Das Login erfolgt mit in Rancher erstellten User.


### Login und Auswahl Kubernetes Cluster

Melden Sie sich im Rancher WebGUI mit Ihrem User an und wählen sie anschliessend den gewünschten Cluster aus.


Auf dem Cluster Dashboard finden sie oben Rechts ein Button mit dem Ihr `Kubeconfig File` angezeigt werden kann. Speichern sie dieses im Ihrem Home-Verzeichniss unter `.kube/config`. Prüfen sie anschliessend ob `kubectl` damit richtig konfiguriert ist z.B. mit `kubectl version`

**Note:** Wenn sie bereits ein vorhandeses config File haben, müssen Sie ggfs. die Einträge aus dem Rancher config File in Ihr bestehendes übertragen.





## Namespace erstellen

Ein Namespace in Kubernetes ist das Top Level-Konzept um Ihre Applikationen, Deployments, Container etc. zu organisieren. Siehe [Kubernetes Docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/).

**Note:** Rancher kennt zusätzlich noch das Konzept des [Projekt](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/projects-and-namespaces/) mit dem mehrere Namespaces zusammengefasst werden können.

Im Rancher GUI können nun zusätzlich noch ihr Projekt auswählen.


## Aufgabe: LAB3.1

Erstellen Sie auf der Lab Plattform einen neuen Namespace.

**Note**: Verwenden Sie für Ihren Namespace-Namen am besten Ihren Techlab-Benutzernamen oder sonstigen Identifier, bspw. `[USER]-lab3-1`

> Wie kann ein neuer Namespace erstellt werden?

**Tipp** :information_source:
```
$ kubectl help
```

oder verwenden Sie das Rancher GUI um den Namespace zu erstellen.

**Note:** Namespaces die via `kubectl` erstellt werden, müssen anschliessend in Rancher noch in ein Projekt übertagen werden, damit diese im GUI sichtbar sind.

## Aufgabe: LAB3.2 Rancher-GUI erforschen


Schauen Sie sich die verschiedenen Menüpunkte in Rancher an. Aktuell gibts weder Deployments noch Pods oder Services.


---

## Lösung: LAB3.1

```
$ kubectl create namespace [USER]-lab3-1
```
---

Bestpractice ist den jeweiligen Namespace bei sämtlichen `kubectl` Befehlen über `--namespace namespace` oder in Kurzform `-n namespace` explizit anzugeben

---

**Ende Lab 3**

<p width="100px" align="right"><a href="04_deploy_dockerimage.md">Ein Docker Image deployen →</a></p>

[← zurück zur Übersicht](../README.md)
