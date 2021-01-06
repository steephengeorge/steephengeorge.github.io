---
layout: inner
position: center
title: Inter-service communication within Kubernetes
date: 2018-01-30 14:15:00
categories: kubernetes
tags: Software Java Spring-Cloud Spring-Boot
lead_text: "In a previous article Route using Spring Cloud Gateway, I have explained how to use Spring cloud Gateway to route the endpoints to different services using 'path' as a predicate. Here I am planning to do the same within a Minikube cluster."
project_link: 'http://steephengeorge.github.io/kubernetes/2018/01/30/inter-service-k8s-communication.html'
---

## Inter-service communication within Kubernetes

In a previous article [Route using Spring Cloud Gateway](https://soulmeadows.com/kubernetes/2018/01/22/route-using-gateway.html), I have explained how to use Spring cloud Gateway to route the endpoints to different services using 'path' as a predicate. Here I am planning to do the same within a Minikube cluster.

In the article, [How to deploy and expose a service using Minikube?](https://soulmeadows.com/kubernetes/2018/01/11/deploy-expose-k8s-service.html) I explained how did I deploy register-service. I just followed the same process for another service called search-service. So we have two different services up and running in Minikube.

In search-service we have a REST endpoint to return the preferences

{% highlight java %}

@RestController

@RequestMapping("/searchSvc/api")

public class preferenceController {

@GetMapping("/preferences")

public List\<String\> getPreferences(){

return List.of("reading", "music");

}

}

{% endhighlight %}

{% highlight bash %}

\> kubectl get po

register-service-57495f79fb-z5djj 1/1 Running 5 2d5h

search-service-67fbcf546f-fkm6d 1/1 Running 5 44h

{% endhighlight %}

{% highlight bash %}

\> kubectl get svc

register-service ClusterIP 10.105.89.134 \<none\> 9095/TCP 2d5h

search-service ClusterIP 10.106.96.97 \<none\> 9096/TCP 45h

{% endhighlight %}

Even though we didn't create endpoint objects directly for these services, the Minikube cluster generated those for us. This data point is pretty interesting to watch when we change the replicas of a service.

{% highlight bash %}

\> kubectl get ep

register-service 172.17.0.15:9095 2d5h

search-service 172.17.0.17:9096 45h

{% endhighlight %}

The values of the following parameters are different for gateway service while deploying in the Minikube cluster. The parameter 'URI' holds the service name rather than the host IP address. To deploy the gateway-service, follow the same step we did for the other two services.

{% highlight bash %}

server.port=9097

spring.cloud.gateway.routes[0].uri = http://search-service:9096

spring.cloud.gateway.routes[1].uri = http://register-service:9095

spring.cloud.kubernetes.discovery.all-namespaces=true

spring.cloud.kubernetes.config.namespace=all

{% endhighlight %}

Since we are running the gateway service on port 9097 as a clusterip service, we can do a port forwarding on Minikube

{% highlight bash %}

kubectl port-forward service/gateway-service 9097:9097

Forwarding from 127.0.0.1:9097 -\> 9097

Forwarding from [::1]:9097 -\> 9097

{% endhighlight %}

Now by using Postman, we can access register-service endpoint as well as search-service endpoint through gateway-service.

{% highlight javascript %}

http://localhost:9097/registerSvc/api/address

Address1

http://localhost:9097/searchSvc/api/preferences

[

"reading",

"music"

]

{% endhighlight %}

You can find all code in GitHub:

- [search-service](https://github.com/steephengeorge/experiments/tree/main/search-service)
- [register-service](https://github.com/steephengeorge/experiments/tree/main/register-service)
- [gateway-service](https://github.com/steephengeorge/experiments/tree/main/gateway-service)
