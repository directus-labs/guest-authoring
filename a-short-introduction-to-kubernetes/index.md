---
title: 'Introduction to Kubernetes'
description: 'A short introduction to Kubernetes in a Directus context'
author:
  name: 'Mikke Schirén'
  avatar_file_name: 'mikke.png'
---

## Introduction

When you start to run applications in docker containers it is pretty simple just fire up a container. But then you need another container, like a database, and then you need secrets, hosts, backups, updates etc. It could be quite messy to handle all the things, and you need something to orchestrate it with.

Kubernetes (K8s) is the most popular open-source container orchestration system, and it automates deployments, scaling, and management of applications. While Kubernetes is not the easiest way to run a self-hosted Directus project, there a lot of gains to be realized.

If you never used Kubernetes, you may be surprised that you very often already have it on your development computer. Kubernetes, in a simple flavor is for instance shipped with Docker Desktop for Mac. So the place to test out Kubernetes itself it is very often nearby. If you want to experiment with Kubernetes for the first time, I really recommend that you do so locally.

There are a lot of hosted Kubernetes solutions for you to choose from - like Amazon's [EKS](https://aws.amazon.com/eks/), Google's [GKE](https://cloud.google.com/kubernetes-engine), and Microsoft's [AKS](https://azure.microsoft.com/en-us/products/kubernetes-service). 

I will do a walk through of some of the basic pieces of the Kubernetes puzzle, and what they mean in a Directus context.

:::info Examples Only

The example files in this post, should not be used to create a Directus setup, they are just there to describe the different parts, please see the GitHub gist in the end of the post if you are interested in a full working example.

:::

## Yaml and API:s

All objects can be described and created with yaml, with a minimum requirement telling which api you are going to use, and what kind, like:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: directus
  labels:
    app: directus
spec:
  containers:
  - name: directus
    image: directus/directus:10.8.3
    ports:
      - name: http
        containerPort: 8055
        protocol: TCP
```

## Metadata and labels

Metadata is used to identify your Kubernetes objects, without them you soon will be lost, and even different parts of the Kubernetes puzzle requires them. Some metadata is a must, like the name - otherwise there is no identification. Labels and annotations adds metadata to your objects so you could easily find them or target them.

One of the reasons is then you create an object, like a Deployment, a random string is used for the pod it creates. So you to communicate with it, you need the metadata to identify it.

Another, extremely useful metadata is annotations, but that is out of the scope for this post.

## Containers

Containers are the Docker images you deploy to your K8s cluster. There are mainly two kind of containers, initContainers and containers. initContainers runs before your containers start, like if you need to set permissions, or do some task like updating your Directus schema etc.

## Environment variables

Environment variables could be added as part of your object, like the public url for Directus.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: directus-app
  labels:
    app: directus
spec:
  containers:
  - name: directus
    image: directus/directus:10.8.3
    ports:
    - containerPort: 80
    env:
      - name: PUBLIC_URL
        value: https://directus.app
```

You could also use Secrets and use with for setting environment variables, that I will not cover here.

## Pods

A pod is the smallest deployable compute object in K8s, like the example above. A pod is what is running one or several docker containers, a pod could be Directus, MySQL or Redis. If your pod crashes, it's dead and doesn't restart - it's like when you start a docker container with `docker run directus` and suddenly something gets weird, or your resources run out. So you need something to handle pods with, and the logic they are started, with config and volumes, there is where Deployments and StatefulSets come in.

## Deployments

A Deployment manages a set of Pods to run an application workload, usually one that doesn't maintain state (coming that later). A Deployment is a way to describe the pod(s) you want to run, if you need to mount volumes (for storing data) or config (like the config file for your Directus Deployment), also you add logic for resources is going to use (RAM, and CPU). So, with a Deployment, you create pods. Each Deployment, creates normally one pod (which, like state above, could have many containers).

A Deployment could "update" your pod - and what happens is that K8s takes down the existing pod, and replaces with a new one, like when updating your Directus instance, or adding new config or env variables.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: directus-Deployment
  labels:
    app: directus
spec:
  selector:
    matchLabels:
      app: directus
  template:
    metadata:
      labels:
        app: directus
    spec:
      containers:
      - name: directus
        image: directus:1.32.3
        ports:
          - containerPort: 80
        env:
          - name: PUBLIC_URL
            value: https://directus.app
```

## StatefulSets

When Deployments normally doesn't should be used for a pod that use a state, like a database, you need something that should be used for that, here is when you should use StatefulSet. Where a Deployment could normally just replace a pod, a StatefulSet needs either several replicas, or need to be deleted before it is replaced. Deletion of a pod, doesn't mean you loose your data, as long you are saving that data to a persistent volume.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: directus-mariadb
  labels:
    app: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  serviceName: directus-mariadb
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mariadb
    spec:
      containers:
        - name: mariadb
          image: mariadb:10.9.7
          ports:
            - name: mariadb
              containerPort: 3306
          volumeMounts:
            - name: data
              mountPath: /bitnami/mariadb
            - name: config
              mountPath: /opt/bitnami/mariadb/conf/my.cnf
              subPath: my.cnf
      volumes:
        - name: config
          configMap:
            name: directus-mariadb
  volumeClaimTemplates:
    - metadata:
        name: data
        labels:
          app.kubernetes.io/instance: directus
          app.kubernetes.io/name: mariadb
      spec:
        accessModes:
          - "ReadWriteOnce"
        resources:
          requests:
            storage: "8Gi"
```

## ReplicaSets

An application like Directus works with replicas if you set up it with external file storage, and you are not using SQLLite (there are still ways to have replicas with internal file storage and SQLite, but that is more advanced then I will cover in this post, and I don't recommend you do that).

Here we are creating 3 replicas of Directus:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: directus-Deployment
  labels:
    app: directus
spec:
  replicas: 3
  selector:
    matchLabels:
      app: directus
  template:
    metadata:
      labels:
        app: directus
    spec:
      containers:
      - name: directus
        image: directus:1.32.3
        ports:
          - containerPort: 80
```

## Volumes and StorageClasses

As you could see in the MariaDB StatefulSets above, we are mounting volumes. Volumes could be temporarily, like a `temp` directory, or they could be persistent and data stays between Deployments.

A persistent volume needs a StorageClass, so it could create the volume, and a StorageClass normally has the type `ReadWriteOnce` - which means it only could be written to by one pod (but all of the containers in the pod). Another type of StorageClass could use `ReadWriteMany` - and that is useful if you have many pods that needs to write to the same filesystem - like uploading files. You should never setup a database with a ReadWriteMany StorageClass, that could end badly.

Here is an example of a Deployment with an `emptyDir` (`/tmp`) - a non-persistent volume, when the pod restarts, it is empty again.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: directus-Deployment
  labels:
    app: directus
spec:
  selector:
    matchLabels:
      app: directus
  template:
    metadata:
      labels:
        app: directus
    spec:
      containers:
      - name: directus
        image: directus/directus:10.8.3
        ports:
          - containerPort: 80
        env:
          - name: PUBLIC_URL
            value: https://directus.app
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
```

## ConfigMaps

ConfigMaps are used to store different kind of configuration objects like environment variables or a file-based configuration, which later could be used to mount (like a volume), for an application to use.

Like config for MariaDB:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: release-name-mariadb
  labels:
    app: mariadb
data:
  my.cnf: |-
    [mysqld]
    skip-name-resolve
    max_allowed_packet=16M
```

And then mounted:

```yaml
[...]
          volumeMounts:
            - name: config
              mountPath: /opt/bitnami/mariadb/conf/my.cnf
              subPath: my.cnf
      volumes:
        - name: config
          configMap:
            name: release-name-mariadb
[...]
```

This means that you can mount your config to a docker container, so the container could be clean, without your projects config.

## Services

Service are used to expose the running pods, so you could communicate between the pods or expose them in different ways. The docker port is not exposed by default, so you need a way to tell Directus where to find MariaDB as an example. There are different kind of Services, and here we only going to cover the most common one, `ClusterIP`. Like the name says the service gets an internal IP so communication can happen between...

Example service for the MariaDB StatefulSet could be:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: directus-mariadb
  labels:
    app: mariadb
  annotations:
spec:
  type: ClusterIP
  ports:
    - name: mysql
      port: 3306
      protocol: TCP
      targetPort: mysql
  selector:
    labels:
      app.kubernetes.io/name: mariadb
```

This finds the MariaDB StatefulSet (the selector part, looks for a label named `app.kubernetes.io/name`, and value `mariadb`), and exposes port 3306 with the name `directus-mariadb` (equals hostname) for the running pod.

So in Directus, you could communicate with the MariaDB pod, setting the port and the hostname as environment variables in the Deployment, like:

```yaml
env:
  - name: DB_PORT
    value: "3306"
  - name: DB_HOST
    value: directus-mariadb
```

## Ingresses

Ingresses exposes the service (which exposes the pod) to "the world". Ingresses exists of different kinds, and one of the most common ones is Nginx Ingress. If we want someone to reach our Directus app outside of the cluster, we need an ingress, but first we need a service to expose our Directus app:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/instance: directus
    app.kubernetes.io/name: directus
  name: directus
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 8055
  selector:
    labels:
      app: directus
  type: ClusterIP
```

So, now can use the name of the Service, `directus` in the ingress, like:

```yaml
apiVersion: networking.K8s.io/v1
kind: Ingress
metadata:
  labels:
    app.kubernetes.io/instance: directus
    app.kubernetes.io/name: directus
  name: directus
spec:
  rules:
  - host: directus.app
    http:
      paths:
      - backend:
          service:
            name: directus
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
```

So, if we create a DNS-record for `directus.app`, pointing to your K8s cluster, the user would end up in our Directus app.

## Summary

You have learned some of the basic parts and lingo in Kubernetes, and also how Directus fit in that puzzle. And as promised, [here is full Directus Deployment, with Redis and MariaDB](https://gist.github.com/mikkeschiren/ab57c6bb67f57b21040215a8284d9450).
