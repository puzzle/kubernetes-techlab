# Vorbereitungen für das Techlab

## Plattform setup

* GKE Cluster erstellen, minimal 3 Nodes mit je 8 GB Memory

### Berechtigungen User

Die Techlab User benötigen mindestens folgende Berechtigungen

* Kubernetes Engine-Entwickler
* Log-Betrachter
* Monitoring-Betrachter 

### Setup Helm

Vorgänging muss man als Cluster Admin Helm installieren und `helm init` ausführen

Anschliessend mit folgenden Befehlen noch die nötigen Rechte erteilen:

```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

## Allgemein

* Account Blätter ausdrucken
* Folien vorbereiten
