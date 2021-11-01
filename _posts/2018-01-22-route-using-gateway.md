---
layout: inner
position: center
title: Route using Spring Cloud Gateway!
date: 2018-01-22 14:15:00
categories: kubernetes
tags: Software Java Spring-Cloud Spring-Boot
lead_text: "Spring Cloud Gateway is one of the components in the Spring Cloud framework. Here we will figure out how to route the REST traffic to different service endpoints using Spring Cloud Gateway. In most examples, I have seen following the same practice of using the application.yml file to set up configurations as shown in Spring documentation. I will initially set it up using YAML then I will convert it into old fashion application. properties. This may be useful to generate a Kubernetes configMap because configMap generation utilities are available to generate it from a conventional application.properties file."
project_link: 'http://steephengeorge.github.io/kubernetes/2018/01/22/route-using-gateway.html'
---

# Part-3: Route using Spring Cloud Gateway

Spring Cloud Gateway is one of the components in the Spring Cloud framework. Here we will figure out how to route the REST traffic to different service endpoints using Spring Cloud Gateway. In most examples, I have seen following the same practice of using the application.yml file to set up configurations as shown in Spring documentation. I will initially set it up using YAML then I will convert it into old fashion application. properties. This may be useful to generate a Kubernetes configMap because configMap generation utilities are available to generate it from a conventional application.properties file.

I have 3 services in place:

1. registerSvc
2. searchSvc
3. gatewaySvc

## RegisterSvc

I have a REST endpoint in this service configured with port number 9095.

{% highlight java %}

@RestController

@RequestMapping("/registerSvc/api")

public class AddressController {

@GetMapping("/address")

public String getAddress() {

return "Address1";

}

}

{% endhighlight %}

## SearchSvc

In searchSvc, I have a REST endpoint set up using port number 9096

{% highlight java %}

@RestController

@RequestMapping("/searchSvc/api")

public class preferenceController {

@GetMapping("/preferences")

public List<String> getPreferences(){

return List.of("reading", "music");

}

}

{% endhighlight %}

## GatewaySvc

It has a Spring cloud gateway as a dependency.

{% highlight xml %}

<dependency>

<groupId>org.springframework.cloud</groupId>

<artifactId>spring-cloud-starter-gateway</artifactId>

</dependency>

{% endhighlight %}

I didn't write a single line of code in this service other than this application.yml file. All magic is happening here.

{% highlight yaml %}

server:

port: 9097

spring:

cloud:

gateway:

routes:

- id: registerSvc

uri: http://localhost:9095/

predicates:

- Path=/registerSvc/api/**

- id: searchSvc

uri: http://localhost:9096/

predicates:

- Path=/searchSvc/api/**

{% endhighlight %}

Once all three services are up and running, I can test these endpoints using Postman using the following URLs.

{% highlight javascript %}

URL: http://localhost:9097/registerSvc/api/address

Response: Address1

URL: http://localhost:9097/searchSvc/api/preferences

Response:

[

"reading",

"music"

]

{% endhighlight %}

## How to set up Application. properties

When I started working on it, I felt it will be pretty easy. I need to expand the YAML tree. But Gateway services spewing trash on my attempts. In the YAML file, we generated a short form of configuration. To attain a corresponding application.properties file, the trick is to go after detailed recommendation under the Fully Expanded Arguments approach

detailed [here](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/index.html#fully-expanded-arguments).

So it is a two-step process, first, change the YAML format to a fully expanded format. Second, flatten the YAML format and write down the application.properties

{% highlight javascript %}

server.port=9098

spring.cloud.gateway.routes[0].id = searchSvc

spring.cloud.gateway.routes[0].uri = http://localhost:9096/

spring.cloud.gateway.routes[0].predicates[0].name=Path

spring.cloud.gateway.routes[0].predicates[0].args.name = Path

spring.cloud.gateway.routes[0].predicates[0].args.regexp =/searchSvc/api/**

spring.cloud.gateway.routes[1].id = registerSvc

spring.cloud.gateway.routes[1].uri = http://localhost:9095/

spring.cloud.gateway.routes[1].predicates[0].name=Path

spring.cloud.gateway.routes[1].predicates[0].args.name = Path

spring.cloud.gateway.routes[1].predicates[0].args.regexp =/registerSvc/api/**

{% endhighlight %}

You can delete or rename the application.yml and restart the service. You can test the REST endpoints through gatewaySvc as follows:

{% highlight javascript %}

URL: http://localhost:9098/registerSvc/api/address

Response: Address1

URL: http://localhost:9098/searchSvc/api/preferences

Response:

[

"reading",

"music"

]

{% endhighlight %}

You can find all code in GitHub:

- [searchSvc](https://github.com/steephengeorge/experiments/tree/main/searchSvc)
- [registerSvc](https://github.com/steephengeorge/experiments/tree/main/registerSvc)
- [gatewaySvc](https://github.com/steephengeorge/experiments/tree/main/gatewaySvc)
