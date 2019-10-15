# Lab 11: Helm

[Helm](https://github.com/helm/helm) is a [Cloud Native Foundation](https://www.cncf.io/) project to define, install and manage applications in Kubernetes.
In this Lab we are going to explore Helm by deploying a simple application and then update the application.
In an later lab, you will have the opportunity to deploy your own application with Helm.

## Task: LAB11.1 install the Helm Client

In this lab, you need the Helm client. For instructions to install the Helm client, visit:
[here](https://docs.helm.sh/using_helm/#installing-the-helm-client)

The Helm server component (Tiller) can be installed globally for the whole Kubernetes cluster. This has some Security Downsides and therefore we will install Tiller later only in your Namespaces.

## Task: LAB11.2 Helm Overview

First, you have to understand the following 3 Helm concepts: "Chart", "Repository" and "Release".
They are explained in the [Helm Introduction](https://docs.helm.sh/using_helm/#three-big-concepts)

With the following command, we can create a template for our own chart. This will create a sub-directory `my-chart` in the current Directory:

```sh
helm create mychart
```

This template is already a valid and fully functional chart which deploys NGINX.
Have a look now on the generated files and their content.
For an explanation of the files, visit the [Helm Developer Documentation](https://docs.helm.sh/developing_charts/#the-chart-file-structure).
In a [later Section](https://docs.helm.sh/developing_charts/#templates-and-values) you find all the information about Helm templates


## Task: LAB11.3 Install an Application with Helm

### Helm and Tiller

Helm needs a component installed in the Kubernetes cluster for the communication with Kubernetes. This component is called Tiller. The local Helm client communicates with the Tiller instance. 

For this Exercise, you can install Tiller in your own Namespace, but we also need to create a ServiceAccount and a Role & Rolebinding for Helm to work correctly:

```
kubectl create sa "tiller-[USER]-dockerimage"
kubectl create role "tiller-role-[USER]-dockerimage" --namespace [USER]-dockerimage --verb=* --resource=*.,*.apps,*.batch,*.extensions
kubectl create rolebinding "tiller-rolebinding-[USER]-dockerimage" --role="tiller-role-[USER]-dockerimage" --serviceaccount="[USER]-dockerimage:tiller-[USER]-dockerimage"
helm init --service-account "tiller-[USER]-dockerimage" --tiller-namespace [USER]-dockerimage --upgrade
```

To not always add the namespace when calling `helm` (with `--tiller-namespace [USER]-dockerimage`), you can set the following environment variable:

```
export TILLER_NAMESPACE=[USER]-dockerimage
```

As mentioned, there are some security conserns with a central and always running Tiller Instance. Everybody with access to the Tiller instance, can create Deployments in all namespaces the ServiceAccount on which Tiller runs, has access to. Therefore we limit the permissions by using Tiller only for one Namespace.


Before actually deploying our generated chart, we can check the (to be) generated Kubernetes ressources with the following Command:

```sh
helm install --dry-run --debug --namespace [USER]-dockerimage mychart
```

Finally, the following command creates a new Release with the Helm chart and deploys the application::
```sh
helm install mychart --namespace [USER]-dockerimage
```

With `kubectl get pods --namespace [USER]-dockerimage` you should see a new Pod. You can list the newly created Helm release with the following command:

```sh
helm ls --namespace [USER]-dockerimage
```

## Task: LAB11.5 Update an Application with Helm

Your deployed NGINX is not yet accessible from external. To expose it, you have to change the Service Type to `NodePort`.
Search now for the service type definition in your chart and make the change.
You can apply your change with the following command:


```sh
helm upgrade [RELEASE] --namespace [namespace] mychart
```

As soon as the Service has a NodePort, you will see it with the following command (As we use -w (watch) you have to terminate the command with CTRL-C):


```sh
kubectl get svc --namespace [namespace] -w
```


NGINX is now available at the given NodePort and should display a welcome-page when accessing it with `curl` or you can also open the page in your browser:

**Hint:** Similar to LAB5.1 you have to set the correct URL.

## Task: LAB11.6 Remove an Application with HELM

To remove an application, you can simply remove the Helm release with the following command:


```sh
helm delete [RELEASE]
```

With `kubectl get pods --namespace [USER]-dockerimage` you should now longer see your application Pod.

---

**End Lab 11**

[← back to Overview](../README.md)
