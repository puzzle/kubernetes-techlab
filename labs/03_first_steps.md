# Lab 3: First steps in the lab environment

In this excercise we will interact for the first time with the lab environment, both with `kubectl` as well as via web console.


## Preparation for the labs

Please clone the git repository, to have a local copy of all necessary excercises.

```
$ cd [Git Repo Project Folder]
$ git clone https://github.com/puzzle/kubernetes-techlab.git
```

As a fallback the repository can be downloaded as [zip file](https://github.com/puzzle/kubernetes-techlab/archive/master.zip).


## Login

**Note:** Please make sure, the be finshed with [Lab 2](02_cli.md).

Our Kubernetes Cluster of the techlab environment runs on [cloudscale.ch](cloudscale.ch) and has been provisioned with [Rancher](https://rancher.com/). You can login into the Cluster with a local Rancher User




### Login and choose Kubernetes Cluster

Login into the Rancher WebGUI with your assigned user and the choose the desired Cluster


On thr Cluster Dashboard you find top right a Button with `Kubeconfig File`. Save the config File into your Home-Directory `.kube/config`. Verify afterwards if `kubectl` works correctly e.g. wiht `kubectl version`

**Note:** If you already have a Kubeconfig File, you might need to merge the Rancher Entries with yours.


## Create a namespace

A namespace is the logical design used in Kubernetes to organize and separate your applications, deployments, services etc. on a top level base. Take a look at the [Kubernetes docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/).


**Note:** Additionaly Rancher does know the concept of a  [Projekt](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/projects-and-namespaces/) which encapsulates multiple Namespaces.

In the Rancher WebGUI you can now choose your Project.



## Exercise: LAB3.1

Create a new namespace in the lab environment.

**Note**: Please choose an identifying name for the namespace, in best case your the techlab username, e.g. `[USER]-lab3-1`

> How can a new namespace be created?

**Tip** :information_source:
```
$ kubectl help
```

**Tip:** By using the following command, you can switch into another namespace:
```
Linux:
$ kubectl config set-context $(kubectl config current-context) --namespace=[USER]-lab3-1
```

```
Windows:
$ kubectl config current-context
// Save the context in a variable
SET KUBE_CONTEXT=[Insert output of the upper command]
$ kubectl config set-context %KUBE_CONTEXT% --namespace=[USER]-lab3-1
```


**Note:** Namespaces created via `kubectl`,have to be assigned to a Project in Order to be seen inside the Rancher WebGUI. Ask your Teacher for the Assignement

## Exercise: LAB3.2 discover the web console


Please check the menu entries, there should neither appear any deployments nor any pods or services.


---

## Solution: LAB3.1

```
$ kubectl create namespace [USER]-lab3-1
```
---

It is bestpractice to explicitly select the Namespace in each `kubectl` command by adding `--namespace namespace` or in short`-n namespace`.

---

**Ende Lab 3**

<p width="100px" align="right"><a href="04_deploy_dockerimage.md">Deploy a docker image →</a></p>

[← back to the overview](../README.md)
