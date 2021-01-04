---
layout: inner
position: center
title: How to deploy and expose a service using Minikube?
date: 2018-01-11 14:15:00
categories: kubernetes
tags: Software Java Kubernetes Spring-Boot
lead_text: "I will create a spring-boot service and show how to deploy it in a Minikube cluster. As usual, I created a prototype using Spring starter and included the Spring web starter dependency. I have chosen the Spring-Boot version 2.3.7.RELEASE to make sure to avoid the compatibility issues raise when I try to deploy in Minikube."
project_link: 'http://steephengeorge.github.io/kubernetes/2018/01/11/deploy-expose-k8s-service.html'
---
# How to deploy and expose a service using Minikube?

Refer to the previous [article](https://soulmeadows.com/kubernetes/2017/12/20/kubernetes_build_environment.html) to find how to set up the Kubernetes local build environment in the development box.

I will create a spring-boot service and show how to deploy it in a Minikube cluster. As usual, I created a prototype using Spring starter and included the Spring web starter dependency. I have chosen the Spring-Boot version 2.3.7.RELEASE to make sure to avoid the compatibility issues raise when I try to deploy in Minikube.

{% highlight xml %}

<parent>

<groupId>org.springframework.boot</groupId>

<artifactId>spring-boot-starter-parent</artifactId>

<version>2.3.7.RELEASE</version>

<relativePath /> <!-- lookup parent from repository -->

</parent>

{% endhighlight %}

{% highlight xml %}

<build>

<plugins>

<plugin>

<groupId>org.springframework.boot</groupId>

<artifactId>spring-boot-maven-plugin</artifactId>

<configuration>

<excludes>

<exclude>

<groupId>org.projectlombok</groupId>

<artifactId>lombok</artifactId>

</exclude>

</excludes>

<image>

<name>docker-registry.localdev:5000/sm/${project.artifactId}:${project.version}</name>

</image>

</configuration>

</plugin>

</plugins>

</build>

{% endhighlight %}

Since I am using Spring-Boot version 2.3.7.RELEASE, I can't directly build against the Minikube docker environment in Windows using IDE. If we try to build the image using IDE using the Spring-boot-maven plugin goal spring-boot:build-image, IDE looks for a docker set up against localhost. I don't want to set up a redundant docker environment in my local build box. I will explain the alternative option shortly. Using Spring-boot version 2.4.1, we can configure a remote Docker environment to create the docker image.

## How to build the image locally and push it to the Docker registry?

1. Open a Windows PowerShell
2. Execute the following command

{% highlight bash %}

> minikube docker-env

$Env:DOCKER_TLS_VERIFY = "1"

$Env:DOCKER_HOST = "tcp://192.168.99.100:2376"

$Env:DOCKER_CERT_PATH = "C:\Users\steep\.minikube\certs"

$Env:MINIKUBE_ACTIVE_DOCKERD = "minikube"

# To point your shell to minikube's docker-daemon, run:

# & minikube -p minikube docker-env | Invoke-Expression

> & minikube -p minikube docker-env | Invoke-Expressionmvn

> cd <Java-project-directory>

> mvn clean spring-boot:build-image

----

------

-------

[INFO] [creator] Reusing layer 'paketo-buildpacks/ca-certificates:helper'

[INFO] [creator] Reusing layer 'paketo-buildpacks/bellsoft-liberica:helper'

[INFO] [creator] Reusing layer 'paketo-buildpacks/bellsoft-liberica:java-security-properties'

[INFO] [creator] Adding layer 'paketo-buildpacks/bellsoft-liberica:jre'

[INFO] [creator] Reusing layer 'paketo-buildpacks/bellsoft-liberica:jvmkill'

[INFO] [creator] Reusing layer 'paketo-buildpacks/executable-jar:class-path'

[INFO] [creator] Reusing layer 'paketo-buildpacks/spring-boot:helper'

[INFO] [creator] Reusing layer 'paketo-buildpacks/spring-boot:spring-cloud-bindings'

[INFO] [creator] Reusing layer 'paketo-buildpacks/spring-boot:web-application-type'

[INFO] [creator] Adding 1/1 app layer(s)

[INFO] [creator] Reusing layer 'launcher'

[INFO] [creator] Adding layer 'config'

[INFO] [creator] Adding label 'io.buildpacks.lifecycle.metadata'

[INFO] [creator] Adding label 'io.buildpacks.build.metadata'

[INFO] [creator] Adding label 'io.buildpacks.project.metadata'

[INFO] [creator] Adding label 'org.opencontainers.image.title'

[INFO] [creator] Adding label 'org.opencontainers.image.version'

[INFO] [creator] Adding label 'org.springframework.boot.spring-configuration-metadata.json'

[INFO] [creator] Adding label 'org.springframework.boot.version'

[INFO] [creator] \*\*\* Images (7b852de32b29):

[INFO] [creator] docker-registry.localdev:5000/sm/register-service:0.0.2

[INFO]

[INFO] Successfully built image 'docker-registry.localdev:5000/sm/register-service:0.0.2'

[INFO]

[INFO] ------------------------------------------------------------------------

[INFO] BUILD SUCCESS

[INFO] ------------------------------------------------------------------------

[INFO] Total time: 41.371 s

[INFO] Finished at: 2021-01-03T15:50:41-08:00

{% endhighlight %}

Now we sucessfully created a docker image named docker-registry.localdev:5000/sm/register-service:0.0.2.

The next step is to push the image to the Docker registry.

{% highlight bash %}

docker push docker-registry.localdev:5000/sm/register-service:0.0.2

The push refers to repository [docker-registry.localdev:5000/sm/register-service]

60c0b38b207a: Pushed

126dd5439845: Layer already exists

14fdaac82fa0: Pushed

6763b99e3792: Layer already exists

c2e9ddddd4ef: Layer already exists

108b6855c4a6: Layer already exists

ab39aa8fd003: Layer already exists

0b18b1f120f4: Layer already exists

cf6b3a71f979: Pushed

ec0381c8f321: Layer already exists

89d24718af18: Layer already exists

eb0f7cd0acf8: Layer already exists

8cc74c487427: Layer already exists

cd208f0ddc9d: Layer already exists

a0d609deac92: Layer already exists

fe6d8881187d: Layer already exists

23135df75b44: Layer already exists

b43408d5f11b: Layer already exists

0.0.2: digest: sha256:64956dda55ffcb24f9e5623b2846c254abb1a17c3f853bd522192f1d521623bd size: 4087

{% endhighlight %}

## How to deploy the service to the Minikube?

Let's do dry runs and generate the deployment yml files

{% highlight bash %}

kubectl create deployment register-service

--image docker-registry.localdev:5000/sm/register-service:0.0.2

-o yaml --dry-run=client >

../k8s/register-service-deployment.yml

kubectl create service clusterip register-service --tcp 9095:9095

-o yaml --dry-run=client > ../k8s/register-service.yml

{% endhighlight %}

The above steps generate the following yml files

{% highlight yml %}

apiVersion: apps/v1

kind: Deployment

metadata:

creationTimestamp: null

labels:

app: register-service

name: register-service

spec:

replicas: 1

selector:

matchLabels:

app: register-service

strategy: {}

template:

metadata:

creationTimestamp: null

labels:

app: register-service

spec:

containers:

- image: docker-registry.localdev:5000/sm/register-service:0.0.2

name: register-service

resources: {}

status: {}

{% endhighlight %}

{% highlight yaml %}

apiVersion: v1

kind: Service

metadata:

creationTimestamp: null

labels:

app: register-service

name: register-service

spec:

ports:

- name: 9095-9095

port: 9095

protocol: TCP

targetPort: 9095

selector:

app: register-service

type: ClusterIP

status:

loadBalancer: {}'

{% endhighlight %}

To deploy the service into our local Minikube execute the command:

{% highlight bash %}

> kubectl apply -f ..\k8s

deployment.apps/register-service created

service/register-service created

kubectl get po

NAME READY STATUS RESTARTS AGE

register-service-57495f79fb-z5djj 1/1 Running 0 66m

PS C:\code\java\soulMeadows\register-service> kubectl get svc

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE

kubernetes ClusterIP 10.96.0.1 <none> 443/TCP 23d

register-service ClusterIP 10.105.89.134 <none> 9095/TCP 66m

{% endhighlight %}

{% highlight bash %}

kubectl logs register-service-57495f79fb-z5djj

Setting Active Processor Count to 4

Calculating JVM memory based on 1136752K available memory

Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -Xmx517911K -XX:MaxMetaspaceSize=106840K -XX:ReservedCodeCacheSize=240M -Xss1M (Total Memory: 1136752K, Thread Count: 250, Loaded Class Count: 16449, Headroom: 0%)

Adding 138 container CA certificates to JVM truststore

Spring Cloud Bindings Enabled

Picked up JAVA_TOOL_OPTIONS: -Djava.security.properties=/layers/paketo-buildpacks_bellsoft-liberica/java-security-properties/java-security.properties -agentpath:/layers/paketo-buildpacks_bellsoft-liberica/jvmkill/jvmkill-1.16.0-RELEASE.so=printHeapHistogram=1 -XX:ActiveProcessorCount=4 -XX:MaxDirectMemorySize=10M -Xmx517911K -XX:MaxMetaspaceSize=106840K -XX:ReservedCodeCacheSize=240M -Xss1M -Dorg.springframework.cloud.bindings.boot.enable=true

----

------

2021-01-04 00:35:17.429 INFO 1 --- [main] org.apache.catalina.core.StandardEngine : Starting Servlet engine: [Apache Tomcat/9.0.41]

2021-01-04 00:35:17.591 INFO 1 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/] : Initializing Spring embedded WebApplicationContext

2021-01-04 00:35:17.591 INFO 1 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1851 ms

2021-01-04 00:35:17.957 INFO 1 --- [main] o.s.s.concurrent.ThreadPoolTaskExecutor : Initializing ExecutorService 'applicationTaskExecutor'

2021-01-04 00:35:18.502 INFO 1 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer : Tomcat started on port(s): 9095 (http) with context path ''

2021-01-04 00:35:18.519 INFO 1 --- [main] c.s.r.RegisterServiceApplication : Started RegisterServiceApplication in 6.193 seconds (JVM running for 6.955)

{% endhighlight %}

You may observe some exceptions in the starting log stack. Please ignore it for the time being. It is not a show stopper.

Time is up to port forward and validate the service up and running:

## How to expose the service from the Minikube?

{% highlight bash %}

kubectl port-forward service/register-service 9095:9095

Forwarding from 127.0.0.1:9095 -> 9095

Forwarding from [::1]:9095 -> 9095

{% endhighlight %}

try the URL: http://localhost:9095/registerSvc/api/address from Postman:

expected result

----

Address1

We are good. You can find the github repository [here](https://github.com/steephengeorge/experiments/tree/main/register-service) && [k8s files](https://github.com/steephengeorge/experiments/tree/main/k8s)
