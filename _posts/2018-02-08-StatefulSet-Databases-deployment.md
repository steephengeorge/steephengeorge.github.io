---
layout: inner
position: center
title: SatefulSet ElasticSearch Deployment
date: 2018-02-08 14:15:00
categories: kubernetes
tags: Kubernetes StatefulSet ElasticSearch
lead_text: "Databases need to maintain the state in storage. While deploying databases on Kubernetes platform, we can decouple the storage layer with service layer and statically link the same storage layer with multiple database services."
project_link: 'http://steephengeorge.github.io/kubernetes/2018/02/08/StatefulSet-Databases-deployment.html'
---

# StatefulSet deployment of ElasticSearch

In current system, now we have stateless services like search and register services. Stateless is kind of a lauded word, in simple sense these services do not hold any specific state of the system. These services are built on top of REST(Representational State Transfer) protocol. Setting up stateless services on Kubernetes platform is already explained so far. In this article, I will try to capture what need to be done to deploy a StatefulSet component.

Databases need to maintain the state in storage. While deploying databases on Kubernetes platform, we can decouple the storage layer with service layer and statically link the same storage layer with multiple database services. This way system can manage the statefulness of database. PersistenceVolumeClaim is the Kubernetes Object to manage the storage.

{% highlight yaml %}

kind: PersistentVolumeClaim

apiVersion: v1

metadata:

name: elastic-search-volumeclaim

annotations:

volume.beta.kubernetes.io/storage-class: standard

spec:

accessModes:

- ReadWriteOnce

resources:

requests:

storage: 4Gi

{% endhighlight %}

Then, I will create a StatefulSet and link the PersistenceVolumeClaim. Since we are planning to deploy on Minikube, I created a volume mount.

{% highlight yaml %}

apiVersion: apps/v1

kind: StatefulSet

metadata:

name: elasticsearch

spec:

serviceName: "elasticsearch"

replicas: 1

selector:

matchLabels:

app: elasticsearch

template:

metadata:

labels:

app: elasticsearch

spec:

containers:

- name: elasticsearch

image: docker.elastic.co/elasticsearch/elasticsearch:7.10.1

env:

- name: discovery.type

value: single-node

ports:

- containerPort: 9200

name: client

- containerPort: 9300

name: nodes

volumeMounts:

- name: es-storage

mountPath: /var/lib/es

volumes:

- name: es-storage

persistentVolumeClaim:

claimName: elastic-search-volumeclaim

{% endhighlight %}

I created a service to expose the ElasticSearch Database.

{% highlight yaml %}

apiVersion: v1

kind: Service

metadata:

name: elasticsearch

labels:

service: elasticsearch

spec:

ports:

- port: 9200

name: client

- port: 9300

name: nodes

selector:

app: elasticsearch

{% endhighlight %}
