# Lab 12: Kustomize

[kustomize](https://github.com/kubernetes-sigs/kustomize) ist ein Projekt aus den [Kubernetes Special Interest Groups (SIGs)](https://kubernetes.io/community/) zur Verwaltung komplexer Kubernetes-Konfigurationen.
In diesem Lab deployen wir eine via kustomize verwaltete Applikationskonfiguration in eine Integrations- und Produktionsumgebung.

## Aufgabe: LAB11.1 kustomize-Client instalieren

Für dieses Lab wird zusätzlich der kustomize-Client benötigt. Dessen Installation wird
[hier](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/INSTALL.md) beschrieben.

## Aufgabe: LAB11.2 kustomize-Konzepte verstehen

Wir werden eine einfach aufgebaute Applikation deployen:

* Ein Deployment startet einen nginx
* Ein Service exponiert dieses Deployment gegen das Internet
* Die Applikation wird einmal in eine Integrationsumgebung deployt und einmal in eine produktive Umgebung

Kustomize bietet "Vererbung" für Kubernetes-Konfigurationen. Wir werden das nutzen, um eine Basiskonfiguration sowie die davon erbenden Applikationsumgebungen Integration und Produktion zu definieren. So bleibt die Applikationskonfiguration DRY.

Überfliege die Kapitel (besonders die Bilder) _1)_ und _2)_ im [kustomize-Readme](https://github.com/kubernetes-sigs/kustomize). Unsere Basiskonfiguration entspricht in kustomize-Sprech einer _base_, Integration und Produktion werden als _overlay_ realisiert.

In den [Dateien zu diesem Lab](./12_data) befindet sich die kustomize-Konfiguration für Basis und Umgebungen. Eine gültige kubernetes-Konfiguration für die Integrationsumgebung generiert man daraus wie folgt:

```
kustomize build ./labs/12_data/overlays/integration
```

Diese Konfiguration lässt sich mittels einer Pipe an `kubectl apply` weitergeben um die Konfiguration anzuwenden. Das ist unser nächster Schritt.

## Aufgabe: LAB11.3 Umgebungen deployen

Lege erst zwei Namespaces für die Applikationsumgebungen an:

```
user=[Dein Techlab-Username]

kubectl create namespace $user-lab-12-integration
kubectl create namespace $user-lab-12-production
```

Deploye dann die Integration...

```
kustomize build ./labs/12_data/overlays/integration | kubectl apply -n $user-lab-12-integration -f -
```

und die Produktion:

```
kustomize build ./labs/12_data/overlays/production | kubectl apply -n $user-lab-12-production -f -
```

Eine entsprechende Abfage zeigt, dass in beiden Umgebungen Pods starten:

```
kubectl get pod -n $user-lab-12-integration
kubectl get pod -n $user-lab-12-production
```

Nach einer Weile zeigt `kubectl get service -n $user-lab-12-integration -w` (abbrechen mit Ctrl + C), welche externe IP dem Service zugewiesen wurde.

Rufe mittels `curl http://[EXTERNAL_IP]` oder Webbrowser die deployte Applikation auf.

Prüfe analog, ob auch die Produktion läuft. Falls ja: Gratulation! Zwei identische Applikationsumgebungen sind deployt.
Im nächsten Schritt verwenden wir kustomize, um umgebungsspezifische Anpassungen vorzunehmen.

## Aufgabe: LAB11.4 Ressourcenlimits setzen

Unser Kubernetes-Cluster hat beschränkte Ressourcen. Wir verhindern nun mittels [Ressourcenlimits](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container), dass eine Umgebung der andern alle CPU-Power oder den Arbeitsspeicher stehlen kann.

Die Applikation soll in der Integrationsumgebung maximal 10% einer CPU und 50MB RAM verwenden können, in der Produktionsumgebung aber das doppelte zur Verfügung haben, um den zu erwartenden Ansturm von Besuchern zu bewältigen.

Der folgende Befehl erstellt einen kustomize-Patch, in welchem diese Anpassung für die Integrationsumgebung spezifiziert wird:

```
cat > labs/12_data/overlays/integration/ressource-quotas.yaml <<EOF
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: app
spec:
  containers:
    - name: app
      resources:
        limits:
          cpu: 100m
          memory: 50Mi
EOF
```

Damit der Patch angewendet wird, muss `labs/12_data/overlays/integration/kustomization.yaml` wie folgt angepasst werden:

```
bases:
- ../../base
patches:
- ressource-quotas.yaml
```

Die Änderungen können danach wie im letzten Abschnitt gelernt angewendet werden:

```
user=[Dein Techlab-Username]
kustomize build ./labs/12_data/overlays/integration | kubectl apply -n $user-lab-12-integration -f -
```

Nach einer Weile zeigt `kubectl get svc -n $user-lab-12-integration -w` (abbrechen mit Ctrl + C), dass ein neuer Pod angelegt wurde und läuft.

Die Deploymentkonfiguration zeigt, dass neu ein Limit für CPU und Memory gilt:

```
kubectl -n $user-lab-12-integration describe deployment app
```

Übrigens: Kustomize patcht ziemlich clever, es verwendet [strategic merges](https://github.com/kubernetes/community/blob/master/contributors/devel/strategic-merge-patch.md). Für dich heisst das: Wenn du neue Werte für YAML-Arrays (z.B. Umgebungsvariabeln, Container eines Pods oder tief verschachtelte Optionen) angibst, führt kustomize die Objeke richtig zusammen und ersetzt sie nicht einfach mit den Inhalten deines Patches.

Konfiguriere nun analog die höheren Limits für die Produktion und freue dich darüber, wie DRY, aber doch flexibel, die Konfiguration ist ;)

## Aufgabe: LAB11.4 Umgebungsvariabeln setzen

Dir ist sicher aufgefallen, dass deine Applikation meldet, ihr Name sei "Test-Applikation"". Diesen Namen kriegt sie über die im Deployment gesetzte Umgebungsvariable `APPLICATION_NAME`.

* Ändere den Namen in einer der Umgebungen. Der Name steht stellvertretend für die vielen Konfigurationsoptionen verschiedenster Container, welche via Umgebungsvariable gesetzt werden können und mitunter nicht in jeder Applikationsumgebung gleich lauten.
* Bleibe ordentlich. Du könntest das natürlich im oben angelegten Ressource-Quota-Patch tun, solltest aber eine Lösung finden, um diesen Aspekt deiner Konfiguration in einem separaten Patch abzulegen.

## Weitere Infos

kustomize hat weitere hilfreiche Features.

* Verwende mehrere _bases_, um verschiedene Komponenten zu einer Konfiguration zu vereinen.
* Füge automatisch Labels zu den Ressourcen oder prefixe deren Namen.
* Erbe in deinen _overlays_ von andern _overlays_
* Und viel mehr, siehe die [Beispiele](https://github.com/kubernetes-sigs/kustomize/tree/master/examples) oder diese [kustomization.yaml](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/kustomization.yaml), welche viele der vorhandenen Features nutzt und erklärt.

---

**Ende Lab 12**

[← zurück zur Übersicht](../README.md)
