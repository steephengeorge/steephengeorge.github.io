---
layout: inner
position: center
title: How to set up a Kubernetes development box?
date: 2017-12-20 14:15:00
categories: kubernetes
tags: Software Java Kubernetes Spring-Boot
lead_text: "I am going to set up a kubernetes development environment in a windows box. I am going to use the following technologies/tools for this purpose: Minikube, Docker Container registry, Spring-Boot, Java 11. I will build a simple Spring-boot application and create a docker image and push it to the container registry. Let's start !!"
project_link: 'http://localhost:4000/kubernetes/2017/12/20/kubernetes-build-environment.html'
---

# Part-1: How to set up a Kubernetes development box?

I am going to set up a kubernetes development environment in a windows box. I am going to use the following technologies/tools for this purpose.

1. Windows 10 operating system
2. Minikube
3. Spring-boot
4. Maven
5. Java 11
6. local private registry

I will build a simple Spring-boot application and create a docker image and push it to the container registry. Let's start !!

## Set up a Minikube

I am going to use the [chocolatey](https://chocolatey.org/)as my deployment tool in windows. If you don't have choco, install it using PowerShell. Please note. Windows PowerShell is not the usual command prompt.

{% highlight PowerShell %}

Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

{% endhighlight %}

There are two preconditions before installing Minikube.

- Enable virtualization in BIOS
- Install Virtual box / Enable Hyper -V

I am using Virtual box. Virtual box is the default virtualization technology using behind Minikube. You can reconfigure for a different VT if you want. We will mention it while discussing the MiniKube configurations.

To install Minikube, it is simple now. Execute on regular Windows command prompt with administrator privilege.

{% highlight PowerShell %}

choco install minikube

{% endhighlight %}

To manage the Minikube environment there are 3 commands

{% highlight PowerShell %}

- minikube start
- minikube stop
- minikube delete
- minikube status

{% endhighlight %}

Minikube needs a Virtualbox Manager Command Line Interface(CLI) to start Virtualbox. If you don't have installed VirtualBox Manager along with VirtualBox make sure to install it. VirtualBox cannot work on another VirtualBox, in a nested manner. So please note we can't install Minikube in a virtual environment. While starting minikube the first time, if it fails in between, make sure to delete minikube before retrying to start.

To start Minikube executes the command:

{% highlight PowerShell %}

minikube start

{% endhighlight %}

To check the status of Minikube executes the command:

{% highlight PowerShell %}

minikube status

--------

minikube

type: Control Plane

host: Running

kubelet: Running

apiserver: Running

kubeconfig: Configured

{% endhighlight %}

Now minikube is up and running. We can find the IP address of Minikube using the command:

{% highlight PowerShell %}

minikube ip

{% endhighlight %}

If you prefer to log in to the bash shell of minikube, use the command:

{% highlight PowerShell %}

minikube ssh

{% endhighlight %}

To access the docker environment of Minikube, you have to move to Powershell. From PowerShell execute the command and follow the instructions within output to point your shell to Minikubes docker-daemon.

{% highlight PowerShell %}

minikube docker-env

-----------------

---

# To point your shell to minikube's docker-daemon, run:

# & minikube -p minikube docker-env | Invoke-Expression

{% endhighlight %}

So we will execute the command to point our shell to docker daemon:

{% highlight PowerShell %}

& minikube -p minikube docker-env | Invoke-Expression

{% endhighlight %}

Now onwards, when we need to execute a docker command, we have to use this PowerShell. Please note, we didn't set up a docker environment in our Windows machine. But Minikube set up an internal docker environment, and we are going to use that in our build workflow.

## Set up a container registry

Minikube has several add-ons. One of them is the registry. We can enable this add on with the command.

{% highlight PowerShell %}

minikube addons enable registry

{% endhighlight %}

It will create a container registry along with the Minikube installation. You can access the registry using a browser:

{% highlight PowerShell %}

http:// <minikube-IP >:5000/v2

{% endhighlight %}

You will see blank curly braces over the screen.

To access this registry from the Minikube Docker, we need to enable the insecure container registry access from Minikube.

To enable insecure registry access(Don't worry, you are not leaving your host system!), edit the file by adding the minikube Ip and registry port number and stop and start minikube.

Update the hosts file with registry name

{% highlight PowerShell %}

C:\Windows\System32\drivers\etc\hosts

 <Minikube IP > docker-registry.localdev

 <Minikube IP > minikube

{% endhighlight PowerShell %}

{% highlight PowerShell %}

C:\Users \<username> \.minikube\machines\minikube\config.json

"InsecureRegistry": [

"10.96.0.0/12", // Add coma

"docker-registry.localdev:5000" //-\> Add this line

],

{% endhighlight %}

Stop and start Minikube, now you can access the registry using http as follows:

{% highlight PowerShell %}

http://docker-registry.localdev:5000/v2

{% endhighlight %}

## Build a Docker image for a Spring-boot app

Now we will create a simple Spring-boot application. You can go to https://start.spring.io/ and generate a simple web application. The idea is to create a simple REST endpoint using Spring-boot and deploy it within our Minikube.

I am using Spring boot version 2.4.1. that provides the capability to access our Minikube docker environment for building the image. I removed the SNAPSHOT tag from versioning, so I can reuse it to tag my image.

{% highlight xml %}

Pom.xml

 <parent >

 <groupId >org.springframework.boot </groupId >

 <artifactId >spring-boot-starter-parent </artifactId >

 <version >2.4.1 </version >

 <relativePath / >  <!-- lookup parent from repository -- >

 </parent >

 <groupId >com.sm </groupId >

 <artifactId >collection-service </artifactId >

 <version >0.0.1 </version >

{% endhighlight %}

I am using Java 11. I only added Spring-boot-starter-web and Lombok as a dependency. Lombok library helps us to avoid boilerplate coding.

{% highlight xml %}

 <properties >

 <java.version >11 </java.version >

 </properties >

 <dependencies >

 <dependency >

 <groupId >org.springframework.boot </groupId >

 <artifactId >spring-boot-starter-web </artifactId >

 </dependency >

 <dependency >

 <groupId >org.projectlombok </groupId >

 <artifactId >lombok </artifactId >

 <optional >true </optional >

 </dependency >

 <dependency >

 <groupId >org.springframework.boot </groupId >

 <artifactId >spring-boot-starter-test </artifactId >

 <scope >test </scope >

 </dependency >

 </dependencies >

{% endhighlight %}

In the build component of pom, we can set up the image name. To push the image to the docker-registry. localdev, we need to tag it with the registry name.

Image name holds details as follows:
{% highlight xml %}

 <container-registry domain name >: <port number >/ <repository >/ <image name >: <version tag >
 
 {% endhighlight %}

During the build process, the spring-boot-maven-plugin has a goal spring-boot:build-image to generate the docker image. By default, the build process tries to access the local docker environment. From Spring-boot 2.4.1 onwards we have an option to configure a remote docker set up. We are accessing the Minikube docker environment for our build process. You can see its configuration within node docker.

{% highlight xml %}

 <build >

 <plugins >

 <plugin >

 <groupId >org.springframework.boot </groupId >

 <artifactId >spring-boot-maven-plugin </artifactId >

 <configuration >

 <image >

 <name >docker-registry.localdev:5000/sm/${project.artifactId}:${project.version} </name >

 </image >

 <docker >

 <host >tcp://minikube:2376 </host >

 <tlsVerify >true </tlsVerify >

 <certPath >C:\Users\<user name>\.minikube certs</certPath >

 </docker >

 </configuration >

 </plugin >

 </plugins >

 </build >

{% endhighlight %}

Spring-boot 2.4.1 by default uses the TLS.1.3 version for secure communication. Build process access to the remote Docker environment is enabled with TLS1.3. But to use the TLS1.3 we need Java 15. Since we are using Java 11 we need to pass the TLS configuration as part of our build process

To generate the image execute the command

{% highlight bash%}

mvn clean spring-boot:build-image -Djdk.tls.client.protocols=TLSv1.2

{% endhighlight %}

This will generate an image within the Minikube Docker environment.

Now onwards use the PowerShell to interact with docker and docker-registry. Following steps to push the latest built image to our local private registry.

{% highlight bash%}

docker images | findstr collection

docker-registry.localdev:5000/sm/collection-service 0.0.1 11e993ec7143 41 years ago 278MB

docker push docker-registry.localdev:5000/sm/collection-service:0.0.1

{% endhighlight %}

Images are immutable within container registry, so if you try to push the same image with same tag version, push attempt will fail.

## Deploy the app in Minikube

Since our container registry is set up along with Minikube there are not many hurdles to deploying the image to Minikube.

{% highlight bash%}

kubectl create deployment collection-service --image=docker-registry.localdev:5000/sm/collection-service:0.0.1

deployment.apps/collection-service created

C:\Users\<user name>\>kubectl get pods

NAME READY STATUS RESTARTS AGE

collection-service-fb9f5bb84-ws5x5 1/1 Running 0 7s

{% endhighlight %}

The pod is the atomic deployment unit within Kubernetes. A pod can contain one or more containers. We can check the logs to make sure our deployment is successful or not.

{% highlight bash%}

C:\Users\<user name>\>kubectl logs collection-service-fb9f5bb84-ws5x5

Setting Active Processor Count to 4

Calculating JVM memory based on 1297600K available memory

Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -Xmx703177K -XX:MaxMetaspaceSize=82422K -XX:ReservedCodeCacheSize=240M -Xss1M (Total Memory: 1297600K, Thread Count: 250, Loaded Class Count: 12138, Headroom: 0%)

Adding 138 container CA certificates to JVM truststore

Spring Cloud Bindings Enabled

Picked up JAVA\_TOOL\_OPTIONS: -Djava.security.properties=/layers/paketo-buildpacks\_bellsoft-liberica/java-security-properties/java-security.properties -agentpath:/layers/paketo-buildpacks\_bellsoft-liberica/jvmkill/jvmkill-1.16.0-RELEASE.so=printHeapHistogram=1 -XX:ActiveProcessorCount=4 -XX:MaxDirectMemorySize=10M -Xmx703177K -XX:MaxMetaspaceSize=82422K -XX:ReservedCodeCacheSize=240M -Xss1M -Dorg.springframework.cloud.bindings.boot.enable=true

. \_\_\_\_ \_ \_\_ \_ \_

/\\ / \_\_\_'\_ \_\_ \_ \_(\_)\_ \_\_ \_\_ \_ \ \ \ \

( ( )\_\_\_ | '\_ | '\_| | '\_ \/ \_` | \ \ \ \

\\/ \_\_\_)| |\_)| | | | | || (\_| | ) ) ) )

' |\_\_\_\_| .\_\_|\_| |\_|\_| |\_\_\_, | / / / /

=========|\_|==============|\_\_\_/=/\_/\_/\_/

:: Spring Boot :: (v2.4.1)

2020-12-21 06:36:48.363 INFO 1 --- [main] c.s.c.CollectionServiceApplication : Starting CollectionServiceApplication v0.0.1 using Java 11.0.9.1 on collection-service-fb9f5bb84-ws5x5 with PID 1 (/workspace/BOOT-INF/classes started by cnb in /workspace)

2020-12-21 06:36:48.367 INFO 1 --- [main] c.s.c.CollectionServiceApplication : No active profile set, falling back to default profiles: default

2020-12-21 06:36:49.747 INFO 1 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer : Tomcat initialized with port(s): 8080 (http)

2020-12-21 06:36:49.766 INFO 1 --- [main] o.apache.catalina.core.StandardService : Starting service [Tomcat]

2020-12-21 06:36:49.767 INFO 1 --- [main] org.apache.catalina.core.StandardEngine : Starting Servlet engine: [Apache Tomcat/9.0.41]

2020-12-21 06:36:49.855 INFO 1 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/] : Initializing Spring embedded WebApplicationContext

2020-12-21 06:36:49.856 INFO 1 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1381 ms

2020-12-21 06:36:50.250 INFO 1 --- [main] o.s.s.concurrent.ThreadPoolTaskExecutor : Initializing ExecutorService 'applicationTaskExecutor'

2020-12-21 06:36:50.513 INFO 1 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer : Tomcat started on port(s): 8080 (http) with context path ''

2020-12-21 06:36:50.529 INFO 1 --- [main] c.s.c.CollectionServiceApplication : Started CollectionServiceApplication in 2.8 seconds (JVM running for 3.438)

{% endhighlight %}

Looks good!! Service is up and running. But it is not accessible now. To Make a service accessible, we need to expose it. We will look into that aspects in another article.

Refer code [here]: (https://github.com/steephengeorge/experiments/tree/main/collection-service)
