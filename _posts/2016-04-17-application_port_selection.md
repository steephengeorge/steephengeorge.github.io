---
layout: inner
position: center
title: 'Be careful while choosing an application port!!!'
date: 2020-04-17 14:15:00
categories: Troubleshooting
tags: Software Troubleshooting OperatingSystem Networking
lead_text: 'When we choose a port for our application, how far are we diligent in
finding an appropriate port serves our purpose? When we configure an application, we configure a port for server-side
communication, but what is happening in the client socket. If you observe intermittently in production, a port conflict is
happening. There are three possibilities. We will go through one after
another...'
project_link: 'http://steephengeorge.github.com/troubleshooting/2020/04/17/application_port_selection.html'

---


Be careful while choosing an application port!!!
================================================

 
 
For most software-developers, a port is a four-digit number. Oh no,
system administrators say shame on you. For most well-known protocols
like 'Secure Shell'(ssh) and 'File Transfer Protocol' (FTP), ports
are less than three-digit numbers. For ssh, the standard protocol is 22.
For FTP, it is 20. Let be precise port can hold values between 1 to
65535.
 
 


 
 

Software developers usually fall into a trap, are very familiar with
default ports used by their tools. FTP and ssh ports are behind some
development tools and abstractions they are using in daily life. For
Apache Tomcat we know the default port is 8443, Oracle database it
is 1521, Apache Kafka it is 9092, PostgreSQL it is 5432. The <a href="https://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority" title="Internet
Assigned Numbers Authority(IANA)"> Internet
Assigned Numbers Authority(IANA) </a> is managing the official assignments of the port to use. The
Wikipedia page <a href= "https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers" title ="List of TCP and UDP port Numbers" > List of TCP and UDP port Numbers </a>
provides a list of well-known ports and registered ports.
 
 


 
 
When we choose a port for our application, how far are we diligent in
finding an appropriate port serves our purpose? If Apache Tomcat is in
8443 and the Oracle database in 1521, we may go for 9443 or some other
port number for our application port. If we get a port conflict, we are
sure that service is up and running in the background even though IDE
does not signal the same way. Is it good enough? Let's discuss further.
 
 

Ephemeral ports


 
 
Before moving forward, let us take two steps backward and analyze some
fundamentals of network communication. Most of you are aware a socket is
nothing but an IP address and port number. If you use TCP or UDP, there
are two sockets in the game. We usually name them server-side socket and
client-side sockets. UDP and TCP are duplex protocols, so once
established a connection, data transfer is possible in both directions.
 
 
 
 
When we configure an application, we configure a port for server-side
communication, but what is happening in the client socket. The client
application chooses a port from a reserved list available for an
operating system. That reserved list is known as ephemeral ports. Once a
client closed the connection, that port will be reused by any other
client application as per need.
 
 
How to resolve a port conflict?


 
 
On most occasions, when we hit a port conflict, the same application is
kicking in the background. In short, we are trying to run two instances
of the application on the same servers on the same port. A socket is an
Operating System resource and can not share among processes.
 
 
 
 
We can resolve this issue in a few steps across all operating systems.
 
 


<div align="left">
<ol>
 <li> Identify the background process holding the port

 <li> Terminate the process and release the port
</ol>
 

 
 
In the case of the Windows Operating System do these 2 steps:
 
 

{% highlight powershell linenos %}
	netstat - aon | findstr <port number>
	Ex: netstat -aon \| findstr 443
	taskkill /F /PID \<PID\>
	Ex: taskkill /F /PID 4452
{% endhighlight %}

In a similar way, for the Linux Operating system do these 2 steps:

{% highlight powershell linenos %}
	netstat -aon \| grep \<port number\>
	Ex: netstat -aon \| grep 443
	kill -9 \<PID\>
	Ex: kill -9 443
{% endhighlight %}
 
 
Looks pretty easy!!!
 
 
 
 
This is not always pretty easy as you think.
 
 


<h4> Port conflict scenarios</h4>


 
 
If you observe intermittently in production, a port conflict is
happening. There are three possibilities. We will go through one after
another.
 
 
<ul>
  <li> Multiple instances of the same application are up and running </li>
 
 
If that is the case, and if they are supposed to run simultaneously for
scalability, configure them with different port numbers while running in
the same Operating systems. If they are not supposed to run in parallel,
we need to terminate one of them.
 
 
<li>  The same port number configured with the two different applications
    in the same operating system</li>

 
 
If that is the case, the answer is straight forward. We need to use two
different port numbers of two applications. We can update the
configuration files of applications using two separate port numbers.
 
 
<li>  A port number from the ephemeral port range of the Operating system
    is using to configure an application  </li>

 
 
This scenario is a little complicated to identify. One reason, different
Operating Systems are using the distinct ephemeral port range. <a href="https://en.wikipedia.org/wiki/Ephemeral_port"> Wikipedia </a>
says as follows:
 
 
 
 

"The <a href="https://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority"> Internet Assigned Numbers
Authority (IANA) </a>
 suggests the range 49152 to 65535 (215+214 to 216−1) for dynamic
or private ports. Many <a href= "https://en.wikipedia.org/wiki/Linux_kernel"> Linux
kernels </a> use the port
range 32768 to 60999.<a href="https://en.wikipedia.org/wiki/FreeBSD"> 
FreeBSD </a> has used the
IANA port range since release 4.6. Previous versions, including the
<a href ="https://en.wikipedia.org/wiki/Berkeley_Software_Distribution"> Berkeley Software
Distribution BSD </a>
use ports 1024 to 5000 as ephemeral ports.
 
 
 
 
<a href="https://en.wikipedia.org/wiki/Microsoft_Windows"> Microsoft
Windows </a> operating systems through <a href="https://en.wikipedia.org/wiki/Windows_XP" >Windows
XP </a> use the range
1025--5000 as ephemeral ports by default. <a href="https://en.wikipedia.org/wiki/Windows_Vista" >Windows
Vista </a>, <a href="https://en.wikipedia.org/wiki/Windows_7>Windows
7</a> and <a href="https://en.wikipedia.org/wiki/Server_2008"> Server
2008</a> use the IANA
range by default. <a href="https://en.wikipedia.org/wiki/Windows_Server_2003"> Windows Server
2003</a> uses the
range 1025--5000 by default, until Microsoft security update MS08-037
from 2008 is installed, after which it uses the IANA range by default.
Windows Server 2008 with Exchange Server 2007 installed has a default
port range of 1025--60000. In addition to the default range, all
versions of Windows since Windows 2000 have the option of specifying a
custom range anywhere within 1025--65535."
 
 
 
 
So if your application is targeting multiple Operating Systems, it will
be a challenge. one easy step to diagnose this scenario is identifying
the port held by the client or server process. If the client process
keeps the port, this will be the case.
 
 


<h4>How to change ephemeral port Range in Linux?</h4>


 
 
In the case of Linux, let's go beyond what did we see so far. As you
know every setting in the Linux Operating system is ultimately somewhere
in a file. Is it possible to change the ephemeral port range of a Linux
Operating System?
 
 
 
 
In an ideal scenario, the Linux boot process loads the configurations
within /etc/sysctl.conf file. But there is no guarantee, runtime
environment reflected with the configuration present within sysctl.conf
file. To verify the present ephemeral port range in a Linux Operating
System use the following command:
 
 
{% highlight powershell linenos %}
    cat /proc/sys/net/ipv4/ip_local_port_range
{% endhighlight %}
 
 
In case if we need to modify the port range, edit the /etc/sysctl.conf
with the specified range.

{% highlight powershell linenos %}
Ex: net.ipv4.ip_local_port_range = 9000 65500
{% endhighlight %}

 
 
If you don't prefer to reboot the server and would like to update the
new settings in the runtime environment, execute the following command:
{% highlight powershell linenos %}
sysctl -p /etc/sysctl.conf
{% endhighlight %}
 
 
 
 

<h5>Note:</h5>

One golden rule, we should avoid a port from the ephemeral port range of
an operating system for an application port configuration.
 
 