# Lab 3: Erste Schritte auf der Lab Plattform

In diesem Lab werden wir gemeinsam das erste Mal mit der Lab Plattform interagieren, dies sowohl über kubectl wie auch über die Web Console

## Login

**Note:** Vergewissern Sie sich, dass Sie [Lab 2](02_cli.md) erfolgreich abgeschlossen haben.

Unser Kubernetes Cluster der Techlab Plattform läuft auf GKE (Google Kubernetes Engine) das login erfolgt mittels Google Cloud sdk.

### Installation gcloud 

Installieren Sie anhand der Anleitungen unter https://cloud.google.com/sdk/docs/quickstarts das Google Cloud sdk

### Login und auswahl Kubernetes Cluster

```
$ gcloud init
$ gcloud container clusters get-credentials [cluster] --zone europe-west1-b --project [project]
```
Die Informationen zum Cluster und Projekt erhalten Sie vom Teacher


## Namespace erstellen

Ein Napespace in Kubernetes ist das Top Level Konzept um Ihre Applikationen, Deployments, Container etc. zu organisieren. Siehe [Kubernetes Docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/).


## Aufgabe: LAB3.1

Erstellen Sie auf der Lab Plattform einen neuen Namespace.

**Note**: Verwenden Sie für Ihren Namespacenamen am besten Ihren Techlab-Benutzernamen oder sonstigen Identifier, bspw. `[USER]-lab3-1`

> Wie kann ein neuer Namespace erstellt werden?

**Tipp** :information_source:
```
$ kubectl help
```

**Tipp:** Mit dem folgenden Command können Sie in einen anderen Namespace wechseln:
```
$ kubectl config set-context $(kubectl config current-context) --namespace=[namespace]
```

## Aufgabe: LAB3.2 WebConsole erforschen

Loggen Sie sich unter https://console.cloud.google.com/kubernetes mit Ihrem Account ein und schauen sie sich ihren Namespace im WebUi an.
Aktuell gibts weder Deployments noch Pods oder Services


---

## Lösung: LAB3.1

```
$ kubectl create namespace [USER]-lab3-1
```
---

**Ende Lab 3**

<p width="100px" align="right"><a href="04_deploy_dockerimage.md">Ein Docker Image deployen →</a></p>

[← zurück zur Übersicht](../README.md)
