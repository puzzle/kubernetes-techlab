# Lab 7: Troubleshooting, what is a Pod?

This Lab helps you to Troubleshoot your Application and shows you some Tools to make Troubleshooting easier.

## Login to a Container

Running Container should be treated as immutable Infrastructure and should therefore not be modified. Although, there are some Use-cases in which you have to login into your running Container. Debugging and Analyze is one Example for this.


## Task: LAB7.1 Shell into POD


With Kubernetes you can open a remote Shell into a Pod without installing SSH by Using the Command `kubectl exec`. The command is used to executed anything in a Pod. With the Parameter `-it` you can leave open an Connection. We can use `winpty` for this.

Choose a POD with `kubectl get pods --namespace [TEAM]-dockerimage` and execute the following Command:

```bash
$ kubectl exec -it [POD] --namespace [TEAM]-dockerimage -- /bin/bash
```

With this, you can work inside the POD, e.g.:

```bash
bash-4.2$ ls -la
total 40
drwxr-xr-x 1 default root 4096 Jun 21  2016 .
drwxr-xr-x 1 default root 4096 Jun 21  2016 ..
drwxr-xr-x 6 default root 4096 Jun 21  2016 .gradle
drwxr-xr-x 3 default root 4096 Jun 21  2016 .pki
drwxr-xr-x 9 default root 4096 Jun 21  2016 build
-rw-r--r-- 1 root    root 1145 Jun 21  2016 build.gradle
drwxr-xr-x 3 root    root 4096 Jun 21  2016 gradle
-rwxr-xr-x 1 root    root 4971 Jun 21  2016 gradlew
drwxr-xr-x 4 root    root 4096 Jun 21  2016 src
```

With `exit` you can leave the Pod and close the Connection

```bash
bash-4.2$ exit
```

## Task: LAB7.2 Single Command

Single Commands inside a Container can be executed with `kubectl exec`:


```bash
$ kubectl exec [POD] --namespace [TEAM]-dockerimage env
```

```bash
$ kubectl exec example-spring-boot-69b658f647-xnm94 --namespace [TEAM]-dockerimage env
PATH=/opt/app-root/src/bin:/opt/app-root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=example-spring-boot-4-8mbwe
KUBERNETES_SERVICE_PORT_DNS_TCP=53
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=172.30.0.1
KUBERNETES_PORT_53_UDP_PROTO=udp
KUBERNETES_PORT_53_TCP=tcp://172.30.0.1:53
...
```

## Watch Logfiles

Logfiles of a POD can with shown with the following Command:


```bash
$ kubectl logs [POD] --namespace [TEAM]-dockerimage
```

The Parameter `-f` allows you to follow the Logfile (same as `tail -f`). With this, Logfiles are Streamed and new Entries are shown directly

When a POD is in State **CrashLoopBackOff** it means, that even after some Restarts, the POD could not be started successfully. Even if the Pod is not running, Logfiles can be viewed with the following Command:


 ```bash
$ kubectl logs -p [POD] --namespace [TEAM]-dockerimage
```


## Task: LAB7.3 Port Forwarding

Kubernetes allows you to Forward arbitrary Ports to your Development-Workstation. This allows you to access Admin-Consoles, Databases etc, even when they are not exposed externaly. Port Forwarding are handled by the Kubernetes Master and therefore tunneled from the Client via HTTPS. This allows you to access the Kubernetes Platform even when there are restrictive Firewalls and/or Proxies between your Workstation and Kubernetes.

Exercise: Access the Spring Boot Metrics from [Lab 4](04_deploy_dockerimage.md).


```bash
$ kubectl get pod --namespace [TEAM]-dockerimage
$ kubectl port-forward example-spring-boot-1-xj1df 9000:9000 --namespace [TEAM]-dockerimage
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```

Don't forget to change the POD Name to your own Installation. If configured, you can use Auto-Completion.

The Metricy are now available with the following Link: [http://localhost:9000/metrics/](http://localhost:9000/metrics/).
Those metrics are shown in JSON. With the same Concept you can Access Databases from your local Client or connect your local Development Environment via Remote-Debugging to your Application in the POD.

With the following Link you find more Information about Port-Forwarding: <https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/>

**Note:** The `kubectl port-forward`-Process runs as long as it is not terminated by the User. So when done, stop it with CTRL-C.

---

**End Lab 7**

<p width="100px" align="right"><a href="08_database.md">deploy Datebases and bind to it →</a></p>

[← back to Overview](../README.md)