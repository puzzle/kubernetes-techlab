# Lab: Deploy an Application with a Helm Charts

In this extended lab, we are going to deploy an existing application with a Helm chart

## Helm Hub

Check out [Helm Hub](https://hub.helm.sh/), there you find a lot of Helm charts. For this lab, we are going to install [hackmd](https://hub.helm.sh/charts/stable/hackmd) a realtime, multiplatform collaborative markdown note editor.

## HackMD

The official HackMD Helm-Chart ist published in the stable Helm repository. First, make sure that you have the stable repo added in your Helm client.

```
helm repo list
NAME           	URL                                              
stable         	https://kubernetes-charts.storage.googleapis.com 
local          	http://127.0.0.1:8879/charts                              
```

Which should be the case. If, for any reason, you don't have the stable repo, you can add it by typing:

```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

Let's check the available configuration for this Helm chart. Normally you find them in the [values.yaml](https://github.com/helm/charts/blob/master/stable/hackmd/values.yaml) File inside the repository or described in the charts readme. 

We are going to override some of the values, for that purpose, create a new values.yaml file locally on your workstation with the following content:

```yaml
image:
  tag: 1.3.0-alpine
persistence:
  storageClass: cloudscale-volume-ssd
  size: 1Gi
ingress:
  enabled: true
  hosts:
    - hack-md-team1.[USER]-dockerimage.[NodeIP].xip.io
postgresql:
  persistence:
    size: 1Gi
    storageClass: cloudscale-volume-ssd
  postgresPassword: my-secret-password
```

**Note:** Remember Lab5 to get the NodeIP

**Note:** As `cloudscale-volume-ssd` is anyway the default StorageClass, this would not be necessary

If you look inside the [requirements.yaml](https://github.com/helm/charts/blob/master/stable/hackmd/requirements.yaml) file of the HackMD Chart you see a dependency to the `postgresql` Helm chart. All the `postgresql` values are used by this dependent Helm chart and the chart is automaticly deployed when installing HackMD.

Now deploy the application with:

```
helm install -f values.yaml stable/hackmd
```

Watch the deployed application with `helm ls` and also check the Rancher WebGUI for the newly created deployments, the ingress and also the PersistenceVolumeClaim.

```
helm ls
NAME             	REVISION	UPDATED                 	STATUS  	CHART       	APP VERSION 	NAMESPACE        
altered-billygoat	1       	Thu Sep 26 14:06:59 2019	DEPLOYED	hackmd-1.2.1	1.3.0-alpine	team1-dockerimage
```

As soon as all deployments are ready (hackmd and postgres) you can open the application with the URL from your `values.yaml` file.

### Upgrade

We are now going to upgrade the application to a newer Container image. You can do this with:

```
helm upgrade --reuse-values --set image.tag=1.3.1-alpine quiet-squirrel stable/hackmd
```

And then observe how the deployment was changed to a the new container image tag. 

**Remark / Warning** The HackMD uses the default deployment strategy which is: 

```
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```

this means, the new pod is scheduled and then when ready, the old one will be removed. As we are using the cloudscale-csi storage integration, we can only mount a volume once. Therefore the new pod cannot mount the volume as long as the old pod is not gone. Do you know how to solve this?

As a workaround you can scale down you deployment and then scale it up again:

```
kubectl scale deployment quiet-squirrel-hackmd --replicas=0 --namespace team1-dockerimage
# Wait for the Pod to be removed and then scale up again
kubectl scale deployment quiet-squirrel-hackmd --replicas=1 --namespace team1-dockerimage
```

or change the deployment strategy to:

```
  strategy:
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 25%
    type: RollingUpdate
```


**Note:** that when you change the deployment stategy manually, on a Helm upgrade your changes are lost!

### Cleanup

```
helm delete --purge altered-billygoat
```
