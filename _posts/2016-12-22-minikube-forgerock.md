---
id: 30
title: 'OpenDJ, kubernetes and Pet Sets'
date: 2016-12-22T08:00:00+00:00
author: admin
layout: post
guid: http://www.darkedges.com/blog/?p=31
permalink: /2016/12/21/minikube-forgerock/
categories:
  - forgerock
  - minikube

---

Learning the official way to deploy `OpenDJ` on `Kubernetes` I found some issues.

__**Note:**__ This article is based on [https://forgerock.org/2016/07/opendj-pets-kubernetes/](https://forgerock.org/2016/07/opendj-pets-kubernetes/)

<!-- more -->

## Install minikube

Please follow my previous article [/2016/12/21/minikube-windows/](Deploying minikube on Windows 7) on how to deploy `minikube` on Windows.

## Install helm

Please follow my previous article [/2016/12/21/helm-windows/](Deploying Helm on Windows 7) on how to deploy `minikube` on Windows.

## Create ForgeRock Docker Images

Clone the Docker repository from ForgeRock.

``` bash
git clone https://<userid>@stash.forgerock.org/scm/docker/docker.git
```

Edit the `pom.xml` within the repository, it is currently using the
snapshot maven repository and that does not appear to be currently
available. We need to change the following values.

``` java
   <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!-- Default versions to build  -->
        <openamVersion>14.0.0-M9</openamVersion>
        <opendjVersion>4.0.0-M2</opendjVersion>
        <openidmVersion>4.0.0</openidmVersion>
        <openigVersion>4.0.0</openigVersion>
        <registry>docker-public.forgerock.io:443</registry>
        <repo>forgerock</repo>
    </properties>
```

We also need to updated `opendj\Dockerfile` and change from

```DockerFile
ADD run.sh /opt/opendj/run.sh

CMD ["/opt/opendj/run.sh"]
```

to

```DockerFile
ADD run.sh /opt/opendj/run.sh

RUN chmod 755 /opt/opendj/run.sh

CMD ["/opt/opendj/run.sh"]
```

When updated complete perform the docker build via maven.

``` bash
mvn
```

## Create OpenDJ Pod

Clone the Kubernetes repository from ForgeRock.

``` bash

git clone https://nirving@stash.forgerock.org/scm/docker/fretes.git
```

edit `fretes/helm/opendj/values.yaml` and change

``` yaml
djImage: forgerock/opendj
djImageTag: 4.0.0-M2
```

Enable minkube ingress addon

```bash
minikube addons  enable ingress
```

Start the deployment

```
cd fretes/helm
helm install  --name userstore opendj --set djInstance=userstore
```

## View deployment within Kubernetes Dashboard

Fire up the dashboard with

``` bash
minikube dashboard
```

Expand `Pods` and you should see `configstore-0`. Select it and view the logs.

The command to do  the same via `kubectl` is 

```bash
kubectl get pods
```

To view the logs

``` bash
kubectl log configstore-0
```

## Enable Forwarding

To be able to interact we can use the `kubectl port-forward` command with the correct details

```bash
kubectl port-forward opendj-configstore-0 1389:389
```

You should now be able to connect to the OpenDJ instance using

| Key | value |
|-----|-------
| host | `localhost` |
| port | `1389` |
| username | `cn=Directory Manager` |
| password | `password` |

## Removing Pod

To remove what was deployed issue

``` bash
helm list                               /development/github/darkedges.github.io
NAME                    REVISION        UPDATED                         STATUS          CHART
exhaling-chicken        1               Fri Dec 23 10:31:05 2016        DEPLOYED        opendj-0.1.0

helm delete exhaling-chicken 
```

## Conclusion

There appears to be some issues with the deployment that needs to be resolved.

* The `SNAPSHOT` repository is no longer available from ForgeRock
* There appears to be some issues with permissioning with Windows.
* The example is configured to use port forwarding instead of a `service`

All of these can be easily fixed though.