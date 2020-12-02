---
layout: inner
position: center
title: 'Integrating, Applications using Domain Language'
date: 2019-07-23 15:15:00
categories: Software Design
tags: Software Architecture Configuration
lead_text: 'Integrating domain language processing systems with external systems
will be a challenge. We have two options in hand. Either build a tightly
coupled layer in the external system which enables the communication. In
another way, being arrogant and command, if you want to communicate with
me learn my language...'
project_link: 'http://steephengeorge.github.io/software/design/2019/07/23/application-integration.html'

---

**Integrating, Applications using Domain Language**
![application_integration](/img/posts/integration-1280-853.jpg)
<p>
<div align="justify">
Application integration is a challenge. It will be a great challenge if
engineers have only an overview of the existing systems. There are
several options to get that overview, you can scan through code or read
through existing documentation, or interact with engineers working on
the system a while. In the end, you will be confused over the blind
man\'s elephant. Setting expectations on each above-mentioned avenue may
help you to get a realization as early as possible. Let us go through
each of them one by one.
</div>
</p>
**Scan code**
<p>
<div align="justify">
Read it through buddy!!! You can become an archeological nerd while
running your eyes over those old lines. This is the history of a company
or a product. Do not miss the comments. You may see, it is spilled with
blood and sweat. You can find the handwriting of a sleepless junkie.
Sometimes a "bloody coder" may be laughing at a boss who is stealing his
ideas and climbing the ladder.
</div>
</p>
<p>
<div align="justify">
You can find pieces of a logical puzzle. Play around it, you may find
some pretty lines passed through generations. Keep it in your pocket,
may chance to reuse it on another occasion with a slight adaptation. Oh,
do not copy it as it is. Now you may be flooded with a lot of details.
Probably now you may know why it is doing it. But you cannot find the
tradeoffs engineers applied during the implementations. Why did they
choose this route rather than other multiple possible routes?
</div>
</p>
**Read documentation**
<p>
<div align="justify">
Lol, read the documentation. You can find out UML diagrams, sequence
diagrams, and more. Is it possible to identify which was first? Code or
documentations? Chicken or egg?
</div>
</p>
<p>
<div align="justify">
Is there any trace of how can I integrate this system? Oh, yes there is
a big name for a library. It is not bad to have an option to open the
window/door. Do it always through it!! Build a tunnel. Not bad. Data
will flow through it if you play the game as per the rule. What is this
data? Can I categorize it? There is a logical data model in place.
Follow the ruling buddy!! If you did not get, go back and read the
system design document once again. Everything is there. If you want this
data, do it my way, that is the highway.
</div>
</p>
**Interact with an old hat**
<p>
<div align="justify">

"See, I have this priority list. And you are sitting on the bottom.
I have five minutes. And my manager asked me to explain this. You are
asking great questions, but I don't have time right now". Are you
familiar with this conversation? This engineer is riding a mystical
horse using a magic wand. You can give a smile and go back to your seat.
Unfortunately, the magical horse is counting its days in an enterprise.
This is the hardest avenue. You will end up addressing the people\'s
problems rather than system problems. On top, you are unknowingly
borrowing his glasses and expecting to see a new universe.
</div>
</p>

<p>
<div align="justify">
Do not imagine this avenue is always ending up in people\'s problems. I
met a few compassionate senior engineers ready to answer your questions
during boarding. And always ready to stretch their hands to reach the
next aisle and enabling the junior ones and shed some light into the
darkest corners.
</div>
</p>

**Identify data**
<p>
<div align="justify">
In a nutshell, every application, receiving the input data, massaging
it, and persisting or generating output data. How it is receiving,
massaging, and generating depends on the specific domain. One of the
pitfalls, while processing domain-specific data, systems may handle
everything as domain data.
</div>
</p>
<p>
<div align="justify">
Some of the domain-specific data processing required domain-specific
language for several reasons. In case of low latency and high-volume
data processing, the system may be enabled with domain specific
serialization. This decision will enable us to attain high performance
because the source and destination can use a common language. And that
language can build on top of low-level system abstractions.
</div>
</p>
<p>
<div align="justify">
Integrating domain language processing systems with external systems
will be a challenge. We have two options in hand. Either build a tightly
coupled layer in the external system which enables the communication. In
another way, being arrogant and command, if you want to communicate with
me learn my language.
</div>
</p>
<p>
<div align="justify">
The second option is to build another layer around domain specific
systems. And expose a common standard language to communicate with all
external systems.
</div>
</p>
<p>
<div align="justify">
This itself can attain multiple ways. If it is low-frequency data, as
per request, the data hand over mechanism will be good enough. Or
low-frequency event trigger can invoke a web service call and share the
data with the external system. If it is a high frequency, high volume
data transfer scenario, we should depend on a messaging system for
reliability. Or in certain scenarios a hybrid version, like send the
request over web service and pass data through messaging systems will
work.
</div>
</p>