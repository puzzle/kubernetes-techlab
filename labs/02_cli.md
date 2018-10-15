# Lab 2: Kubernetes CLI installieren

In diesem Lab werden wir gemeinsam das kubectl Client installieren und konfigurieren, damit wir danach die ersten Schritte auf der Techlab Plattform durchführen können.

## Command Line Interface

Das **kubectl** stellt ein Interface zu einem Kubernetes Cluster bereit.

Der Client ist in Go programmiert und kommt als einzelnes Binary für die folgenden Betriebsysteme daher:

- Microsoft Windows
- Mac OS X
- Linux


## kubectl herunterladen und installieren

Die Instruktionen auf https://kubernetes.io/docs/tasks/tools/install-kubectl/ folgen

**Wichtig:** bitte die Version beachten, aktuelle Version des Kubernetes Masters: 1.9.7-gke.6 


## Manuelle Installtion von kubectl

Falls die Installation über die Paketquellen nicht funktioniert, oder bspw. eine explizite Version benötigt wird, kann das kubectl auch als Binary herunter geladen werden.
Dafür empfehlen sich folgende Phade

**Linux**

```
~/bin
```

**Mac OS X**

```
~/bin
```

**Windows**

```
C:\OpenShift\
```

## Korrekte Berechtigung auf Linux und macOS erteilen

Das kubectl muss ausgeführt werden können.

```
cd ~/bin
chmod +x oc
```

## Das kubectl im PATH registrieren

Unter **Linux** und **Mac OS X** ist das Verzeichnis ~/bin bereits im PATH, daher muss hier nichts gemacht werden.

Falls das kubectl in einem anderen Verzeichnis abgelegt wurde, kann der PATH wie folgt gesetzt werden:
```
$ export PATH=$PATH:[path to kubectl]
```

### Windows

Unter Windows kann der PATH in den erweiterten Systemeinstellungen konfiguriert werden. Dies ist abhängig von der entsprechenden Windows Version:

- [Windows 7](http://geekswithblogs.net/renso/archive/2009/10/21/how-to-set-the-windows-path-in-windows-7.aspx)
- [Windows 8](http://www.itechtics.com/customize-windows-environment-variables/)
- [Windows 10](http://techmixx.de/windows-10-umgebungsvariablen-bearbeiten/)

**Windows Quick Hack**

Legen sie das kubectl direkt im Verzeichnis *C:\Windows* ab.


## Installation verifizieren

Das kubectl sollte jetzt korrekt installiert sein. Am besten überprüfen wir das, indem wir den folgenden Command ausführen:
```
$ kubectl
```
Der folgende Output sollte angezeigt werden:
```
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.1", GitCommit:"4ed3216f3ec431b140b1d899130a69fc671678f4", GitTreeState:"clean", BuildDate:"2018-10-05T16:46:06Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
[...]
```

Ist dies nicht der Fall, ist möglicherweise die PATH Variable nicht korrekt gesetzt.

---

## bash/zsh completion (optional)

Mit Linux und Mac kann die bash completion mit folgendem Befehl temporär eingerichtet werden:

```
source <(kubectl completion bash)
```

Oder für zsh:
```
source <(kubectl completion zsh)
```

für die Installation der Bash Completion für zukünftige Shells, kann folgender Befehl ausgeführt werden

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

Damit die bash completion funktioniert, muss vorher das Paket `bash-completion` installiert werden.

Ubuntu:

```
sudo apt install bash-completion
```

---

**Ende Lab 2**

<p width="100px" align="right"><a href="03_first_steps.md">Erste Schritte auf der Lab Plattform →</a></p>

[← zurück zur Übersicht](../README.md)
