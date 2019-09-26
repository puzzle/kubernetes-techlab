# Vorbereitungen für das Techlab

## Plattform setup

* Cluster mit Rancher erstellen, 1 x Worker, minimal 3 Nodes mit je 8 GB Memory

### Cloudscale CSI

* App cloudscale-csi in Project System und Namespace kube-system deployen. Values `cloudscale.access_token=...`


### Berechtigungen User

Die Techlab User benötigen mindestens folgende Berechtigungen

* Auf Cluster Ebene: View all Projects
* Auf Cluster Ebene: View Nodes
* Project Owner im entsprechenden Projekt


## Allgemein

* Account Blätter ausdrucken
* Folien vorbereiten
