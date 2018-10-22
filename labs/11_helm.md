# Lab 11: Helm

[Helm](https://github.com/helm/helm) ist ein [Cloud Native Foundation](https://www.cncf.io/) Projekt mit
dem Applikationen auf Kubernetes installiert und aktualisiert werden können.
In diesem Lab werden wir gemeinsam Helm anschauen und eine einfache Applikation damit deployen und anpassen.
In einem späteren Lab habt ihr dann noch die Gelegenheit eine eigene Applikation mit Helm zu deployen.

## Aufgabe: LAB11.1 Helm Client instalieren

Für dieses Lab wird zusätzlich der Helm Client benötigt. Dessen Installation wird
[hier](https://docs.helm.sh/using_helm/#installing-the-helm-client) beschrieben:

Den Helm Server (Tiller) haben wir auf dem Cluster bereits installiert.

## Aufgabe: LAB11.2 Helm Überblick

Zuerst ist es wichtig die 3 Helm Konzepte "Chart", "Repository" und "Release" zu verstehen.
Diese sind in der [Helm Einführung](https://docs.helm.sh/using_helm/#three-big-concepts) kurz beschrieben.

Nun können wir mit folgenden Befehl eine Vorlage für ein eigenes Chart erstellen lassen
(erzeugt Unterverzeichnis `mychart` im aktuellen Verzeichnis):

```sh
helm create mychart
```

Die so generierte Vorlage ist bereits ein gültiges und voll funktionsfähiges Chart
welches Nginx deployed.
Verschaft euch nun einen Überblick über die generierten Dateien und deren Inhalt.
Sinn und Zweck der einzelnen Dateien sind in der [Helm Entwicklerdokumentation](https://docs.helm.sh/developing_charts/#the-chart-file-structure) kurz beschrieben.
In einem [späteren Abschnitt](https://docs.helm.sh/developing_charts/#templates-and-values) finden
sich weitere Informationen zu Helm Templates, welche nicht mit Kubernetes Templates zu verwechseln sind
und einen anderen Syntax verwenden.

## Aufgabe: LAB11.3 Applikation mit Helm installieren

Bevor wir mit dem generierten Chart ein Deployment machen können wir mit folgendem Befehl
anschauen welche Ressourcen Helm aus dem Chart generiert:

```sh
helm install --dry-run --debug mychart
```

Schliesslich erstellt folgender Befehl ein neues Release aus dem Chart und deployed damit die Applikation:
```sh
helm install mychart
```

In `kubectl get pods` sollte nun ein neuer Pod auftauchen, während der neu erstellte
Release mit folgendem Befehl aufgelistet wird:

```sh
helm ls
```

## Aufgabe: LAB11.4 Applikation mit Helm aktualisieren

Der deployte Nginx ist noch nicht von extern verfügbar. Um ihn
von aussen verfügbar zu machen muss der Service Type auf
`LoadBalancer` geändert werden.  
Sucht die Definition der Service Types im Chart und passt diese
entsprechend an. Die Änderung kann danach wie folgt übernommen werden:

```sh
helm upgrade [RELEASE] mychart
```

Sobald dem Service eine externe IP zugeordnet wurde erscheint diese hier (Befehl terminiert nicht, muss mit CTRL-C beendet werden):

```sh
kubectl get svc -w
```

Nginx ist nun unter der zugeordneten externen IP verfügbar und sollte eine Willkommensseite anzeigen (Alternativ die IP in einem Webbrowser eingeben):

```sh
curl http://[EXTERNAL_IP]
```

## Aufgabe: LAB11.5 Applikation mit Helm deinstallieren

Um eine Applikation zu deinstallieren kann einfach der dazugehörige Release gelöscht werden:

```sh
helm delete [RELEASE]
```

---

**Ende Lab 11**

[← zurück zur Übersicht](../README.md)
