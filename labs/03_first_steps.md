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

Unser Kubernetes Cluster der Techlab Plattform läuft auf GKE (Google Kubernetes Engine), das Login erfolgt mittels Google Cloud SDK.


### Installation `gcloud`

Installieren Sie anhand [der für Ihr Betriebssystemen entsprechenden Anleitungen](https://cloud.google.com/sdk/docs/quickstarts) das Google Cloud SDK.

**Note:** Sollte ein Proxy sein Unwesen treiben, funktioniert der interaktive Installer nicht und [muss als Archiv heruntergeladen werden](https://cloud.google.com/sdk/docs/downloads-versioned-archives). Das Zip, welches mit "Windows 64-bit (x86_64)" bezeichnet wird, herunterladen (ausser natürlich es handelt sich um ein 32-bit Windows).

**Note:** Wie bereits in [Lab 2](02_cli.md) erwähnt, kann auf die lokale Installation von `kubectl` und `gcloud` verzichtet und stattdessen die sog. "Cloud Shell" in der Web Console verwendet werden. In diesem Fall muss [Lab 3.2](03_first_steps.md#aufgabe-lab32-web-console-erforschen) vorgezogen werden, um anschliessend in der Cloud Shell das unten beschriebene Login durchzuführen.


### Login und Auswahl Kubernetes Cluster

**Note:** Verstellt ein Proxy den Weg ins weite Netz, [muss dieser zuvor konfiguriert werden](https://cloud.google.com/sdk/docs/proxy-settings#proxy_configuration). Alternativ kann der Parameter `--skip-diagnostics` dem Befehl `gcloud init` mitgegeben werden, kann aber u.U. später zu weiteren Fehlern führen, weshalb das Setzen des Proxy empfohlen wird.

**Note:** Sollte der Proxy zusätzlich TLS-Termination durchführen und so für `gcloud` Zertifikate mit unbekanntem Issuer anzeigen, muss die ausstellende Proxy-CA in `gcloud` konfiguriert werden:

```
$ gcloud config set custom_ca_certs_file <Pfad>
```

Mit Windows kann der Pfad gewohnt mit Backslashes angegeben werden, bspw. `gcloud config set custom_ca_certs_file c:\Users\...\curl-ca-bundle.crt`

Sobald alle nötigen Konfigurationen gemacht wurden, kann eingeloggt werden:

```
$ gcloud init
Welcome! This command will take you through the configuration of gcloud.

Your current configuration has been set to: [default]

You can skip diagnostics next time by using the following flag:
  gcloud init --skip-diagnostics

Network diagnostic detects and fixes local network connection issues.
Checking network connection...done.                                                                                                                                                                               
Reachability Check passed.
Network diagnostic (1/1 checks) passed.

You must log in to continue. Would you like to log in (Y/n)?  Y

Your browser has been opened to visit:

    https://accounts.google.com/o/oauth2/auth?redirect_uri=http%3A%2F%2Flocalhost%3A8085%2F&prompt=select_account&response_type=code&client_id=xxxxxxxxxxx.apps.googleusercontent.com&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fappengine.admin+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcompute+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Faccounts.reauth&access_type=offline


You are logged in as: [user@example.com].

Pick cloud project to use: 
 [1] mobi-kubernetes-schulung
 [2] Create a new project
Please enter numeric choice or text value (must exactly match list 
item):  1

Your current project has been set to: [mobi-kubernetes-schulung].

Do you want to configure a default Compute Region and Zone? (Y/n)?  Y

Which Google Compute Engine zone would you like to use as project 
default?
If you do not specify a zone via a command line flag while working 
with Compute Engine resources, the default is assumed.
 [1] us-east1-b
 [2] us-east1-c
 [3] us-east1-d
 [4] us-east4-c
 [5] us-east4-b
 [6] us-east4-a
 [7] us-central1-c
 [8] us-central1-a
 [9] us-central1-f
 [10] us-central1-b
 [11] us-west1-b
 [12] us-west1-c
 [13] us-west1-a
 [14] europe-west4-a
 [15] europe-west4-b
 [16] europe-west4-c
 [17] europe-west1-b
 [18] europe-west1-d
 [19] europe-west1-c
 [20] europe-west3-b
 [21] europe-west3-c
 [22] europe-west3-a
 [23] europe-west2-c
 [24] europe-west2-b
 [25] europe-west2-a
 [26] asia-east1-b
 [27] asia-east1-a
 [28] asia-east1-c
 [29] asia-southeast1-b
 [30] asia-southeast1-a
 [31] asia-southeast1-c
 [32] asia-northeast1-b
 [33] asia-northeast1-c
 [34] asia-northeast1-a
 [35] asia-south1-c
 [36] asia-south1-b
 [37] asia-south1-a
 [38] australia-southeast1-b
 [39] australia-southeast1-c
 [40] australia-southeast1-a
 [41] southamerica-east1-b
 [42] southamerica-east1-c
 [43] southamerica-east1-a
 [44] europe-north1-a
 [45] europe-north1-b
 [46] europe-north1-c
 [47] northamerica-northeast1-a
 [48] northamerica-northeast1-b
 [49] northamerica-northeast1-c
 [50] us-west2-a
Did not print [3] options.
Too many options [53]. Enter "list" at prompt to print choices fully.
Please enter numeric choice or text value (must exactly match list 
item):  17

Your project default Compute Engine zone has been set to [europe-west1-b].
You can change it by running [gcloud config set compute/zone NAME].

Your project default Compute Engine region has been set to [europe-west1].
You can change it by running [gcloud config set compute/region NAME].

Created a default .boto configuration file at [/home/baffolter/.boto]. See this file and
[https://cloud.google.com/storage/docs/gsutil/commands/config] for more
information about configuring Google Cloud Storage.
Your Google Cloud SDK is configured and ready to use!

* Commands that require authentication will use affolter@puzzle.ch by default
* Commands will reference project `mobi-kubernetes-schulung` by default
* Compute Engine commands will use region `europe-west1` by default
* Compute Engine commands will use zone `europe-west1-b` by default

Run `gcloud help config` to learn how to change individual settings

This gcloud configuration is called [default]. You can create additional configurations if you work with multiple accounts and/or projects.
Run `gcloud topic configurations` to learn more.

Some things to try next:

* Run `gcloud --help` to see the Cloud Platform services you can interact with. And run `gcloud help COMMAND` to get help on any gcloud command.
* Run `gcloud topic -h` to learn about advanced features of the SDK like arg files and output formatting
```

Die Informationen zum Cluster und Projekt in folgendem Befehl erhalten Sie vom Instruktor:

```
$ gcloud container clusters get-credentials [cluster] --project [project]
```


## Namespace erstellen

Ein Namespace in Kubernetes ist das Top Level-Konzept um Ihre Applikationen, Deployments, Container etc. zu organisieren. Siehe [Kubernetes Docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/).


## Aufgabe: LAB3.1

Erstellen Sie auf der Lab Plattform einen neuen Namespace.

**Note**: Verwenden Sie für Ihren Namespace-Namen am besten Ihren Techlab-Benutzernamen oder sonstigen Identifier, bspw. `[USER]-lab3-1`

> Wie kann ein neuer Namespace erstellt werden?

**Tipp** :information_source:
```
$ kubectl help
```

**Tipp:** Mit dem folgenden Command können Sie in einen anderen Namespace wechseln:

Linux:
```
$ kubectl config set-context $(kubectl config current-context) --namespace=[USER]-lab3-1
```

Windows:
```
$ export KUBE_CONTEXT=$(kubectl config current-context)
$ kubectl config set-context %KUBE_CONTEXT% --namespace=[USER]-lab3-1
```

## Aufgabe: LAB3.2 Web Console erforschen

Loggen Sie sich auf der [Web Console](https://console.cloud.google.com/kubernetes) mit Ihrem Account ein. Klicken Sie oben links auf den Button "Projekt auswählen" und wählen als Organisation "puzzle.ch" und anschliessend den entsprechenden Cluster aus.

Schauen Sie sich die verschiedenen Menüpunkte links an. Aktuell gibts weder Deployments noch Pods oder Services.


---

## Lösung: LAB3.1

```
$ kubectl create namespace [USER]-lab3-1
```
---

**Ende Lab 3**

<p width="100px" align="right"><a href="04_deploy_dockerimage.md">Ein Docker Image deployen →</a></p>

[← zurück zur Übersicht](../README.md)
