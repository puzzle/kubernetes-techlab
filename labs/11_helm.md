# Lab 11: Helm

[Helm](https://github.com/helm/helm) is a [Cloud Native Foundation](https://www.cncf.io/) Project to define, install and manage Applications in Kubernetes.
In this Lab we are going to explore Helm by deploying a simple Application and then update the Application.
In an later Lab, you will have the Opportunity to deploy your own Application with Helm.

## Task: LAB11.1 install the Helm Client

In this Lab, you need the Helm Client. For Instructions to install the Helm Client, visit:
[here](https://docs.helm.sh/using_helm/#installing-the-helm-client)

The Helm Server Component (Tiller) can be installed globally for the whole Kubernetes Cluster. This has some Security Downsides and therefore we will install Tiller later only in your Namespaces.

## Task: LAB11.2 Helm Overview

First, you have to understand the following 3 Helm Concepts: "Chart", "Repository" and "Release".
They are explained in the [Helm Introduction](https://docs.helm.sh/using_helm/#three-big-concepts)

With the following Command, we can create a Template for our own Chart. This will create a Sub-Directory `my-chart` in the current Directory:

```sh
helm create mychart
```

This Template is already a valid and fully functional Chart which deploys NGINX.
Have a look now on the generated Files and their Content.
For an Explanation of the Files, visit the [Helm Developer Documentation](https://docs.helm.sh/developing_charts/#the-chart-file-structure).
In a [later Section](https://docs.helm.sh/developing_charts/#templates-and-values) you find all the Information about Helm Templates


## Task: LAB11.3 Install an Application with Helm

### Helm and Tiller

Helm needs a Component installed in the Kubernetes Cluster for the Communication with Kubernetes. This Component is called Tiller. The local Helm Client communicates with the Tiller Instance. Mobiliar does not run a central Tiller in their Kubernetes Cluster (as stated earlier)

For this Exercise, you can easily install Tiller in your own Namespace:

```
helm init --tiller-namespace [USER]-dockerimage --upgrade
```

To not always add the Namespace when calling `helm` (with `--tiller-namespace [USER]-dockerimage`), you can set the following Environment Variable:

```
export TILLER_NAMESPACE=[USER]-dockerimage
```

As mentioned, there are some Security Conserns with a central and always running Tiller Instance. Everybody with access to the Tiller Instance, can create Deployments in all Namespaces the ServiceAccount on which Tiller runs, has access to. Therefore we limit the Permissions by using Tiller only for one Namespace.


Before actually deploying our generated Chart, we can check the (to be) generated Kubernetes Ressources with the following Command:

```sh
helm install --dry-run --debug --namespace [USER]-dockerimage mychart
```

Finally, the following Command creates a new Release with the Helm Chart and deploys the Application::
```sh
helm install mychart --namespace [USER]-dockerimage
```

With `kubectl get pods --namespace [USER]-dockerimage` you should see a new Pod. You can list the newly created Helm Release with the following Command:

```sh
helm ls --namespace [USER]-dockerimage
```

## Task: LAB11.5 Update an Application with Helm

Your deployed NGINX is not yet accessible from External. To expose it, you have to change the Service Type to `NodePort`.
Search now for the Service Type Definition in your Chart and make the Change.
You can apply your Change with the following Command:


```sh
helm upgrade [RELEASE] --namespace [namespace] mychart
```

As soon as the Service has a NodePort, you will see it with the following Command (As we use -w (watch) you have to terminate the Command with CTRL-C):


```sh
kubectl get svc --namespace [namespace] -w
```


NGINX is now available at the given NodePort and should Display a Welcome-Page when accessing it with `curl` or you can also open the page in your Browser:

**Hint:** Similar to LAB5.1 you have to set the correct URL.

## Task: LAB11.6 Remove an Application with HELM

To remove an Application, you can simply remove the Helm Release with the following Command:


```sh
helm delete [RELEASE] --namespace [namespace]
```

With `kubectl get pods --namespace [USER]-dockerimage` you should now longer see your Application Pod.

---

**End Lab 11**

[← back to Overview](../README.md)
