# Lab 4: Deploy a Docker Image

In this lab, we are going to deploy our first pre-built Docker image and look at the Kubernetes concepts pod, service and deployment.


## Task: LAB4.1

After we've familiarized ourselves with the platform in [lab 3](03_first_steps.md), we are going to have a look at deploying a pre-built Docker image from Docker Hub or any other public Docker registry.

As a first step we are going to create a new namespace. A namespace represents a grouping of resources (containers and Docker images, pods, services, configurations, quotas, limits and more). Authorized users inside that namespace are able to manage those resources. Inside a Kubernetes cluster, the name of a namespace has to be unique.



Create a new Namespace with Name `[USER]-dockerimage`:

```
$ kubectl create namespace [USER]-dockerimage
```

Display all existing pods in your namespace (there should not yet be any!):
```bash
$ kubectl get pods -n=[USER]-dockerimage
```

With the command `kubectl get` you can display all kinds of resources of different types.

Use:

```
$ kubectl get namespace
```
in order to display all namespaces you have access to.

## Task: LAB4.2 Start a Pod

As soon as your new namespace has been created, we are able to deploy our first application inside of it. First, we are going to directly start a new pod:

```
$ kubectl run nginx --image=nginx --port=80 --restart=Never --namespace [USER]-dockerimage
```

Use `kubectl get pods --namespace [USER]-dockerimage` in order to show the running pod:
```
$ kubectl get pods --namespace [USER]-dockerimage
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          1m
```


Verwenden Sie `kubectl get pods` um den laufenden Pod anzuzeigen.

Have a look at your nginx pod inside the Rancher WebGUI under "Workloads" an delete the Pod right afterwards.

## Task: LAB4.3 Deployment

In some usecases it makes sense to start a single pod but has its downsides and is not really best practice. Let's look at another Kubernetes concept which is tightly coupled with the pod: the so-called deployment. A deployment makes sure that a pod is monitored and checks that the number of running pods corresponds to the number of requested pods.

With the following command we can create a deployment inside our already created namespace:


```
$ kubectl create deployment example-spring-boot --image=appuio/example-spring-boot --namespace [USER]-dockerimage
```

The output should be:
```
deployment.apps/example-spring-boot created
```

We're using an example from APPUiO (a Java Spring Boot application), which you can find on [Docker Hub](https://hub.docker.com/r/appuio/example-spring-boot/) and [GitHub (Source)](https://github.com/appuio/example-spring-boot-helloworld).

Kubernetes creates the defined and necessary resources, pulls the Docker image (in this case from Docker Hub) and deploys the pod.

Use the command `kubectl get` with the `-w` parameter in order to get the requested resources and afterwards watch for changes. (**This command will never end unless you terminate it with ctrl+c**):


```
$ kubectl get pods --namespace [USER]-dockerimage -w
```

This process can last for some time depending on your internet connection and if the image is already available locally.

**Tip**: If you want to create your own Docker images and use them with Kubernetes, you definitely should have a look at [these best practices](https://docs.openshift.com/container-platform/latest/creating_images/guidelines.html) and apply them. The Image Creation Guide may be from OpenShift, however it also applies to Kubernetes and other container platforms.




## Viewing the Created Resources

When we executed the command `kubectl create deployment example-spring-boot --image=appuio/example-spring-boot --namespace [USER]-dockerimage`, Kubernetes created a deployment resource.


### Deployment

Display the created deployment using the following command:

```
$ kubectl get deployment --namespace [USER]-dockerimage
```
A [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) defines the following facts:

- Update strategy: How application updates should be executed and how the pods are being exchanged
- Containers
  - Which image should be deployed
  - Environment configuration for pods
  - ImagePullPolicy
- The number of pods/replicas that should be deployed

By using the `-o` (or `--output`) parameter we get a lot more information about the deployment itself:
```
$ kubectl get deployment example-spring-boot -o json --namespace [USER]-dockerimage
```

After the image has been pulled, Kubernetes deploys a pod according to the deployment:

```
$ kubectl get pod --namespace [USER]-dockerimage
```

```
NAME                                   READY   STATUS    RESTARTS   AGE
example-spring-boot-69b658f647-xnm94   1/1     Running   0          52m
```

The deployment defines that one replica should be deployed, which is running as we can see in the output. This pod is not yet reachable from outside of the cluster.

## Task: LAB4.4 Verify the Deployment in the Web Console

Try to display the logs from the Springboot application via web console at https://kubedash.kubedev.mobicorp.test/#!/overview?namespace=puzzle-[U-NUMBER]. Don't forget to replace the placeholder inside the URL!


---

**End of lab 4**

<p width="100px" align="right"><a href="05_expose_service.md">Exposing a Service →</a></p>

[← back to the overview](../README.md)

