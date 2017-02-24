---
layout: post
title:  "Working With Webservers (www)"
date:   2016-06-08 12:00:00
categories: blog
---
I work with webservers a lot in my day-to-day life. From helping people figure out why their SSL certs aren't working, to solving problems with specific modules, and even assisting folks that use Apache httpd as a load balancer for their backend servers via mod_cluster.

Sometimes logs don't cut it, and it's extremely helpful to see what's going on between the client and server directly. For that, you can use tcpdump to capture traffic and view it later. Below are just some snippets you may find useful when working with web traffic.

Use this to generate a pcap file with *all* traffic on a particular host. Generally, you'll run this on your server to see incoming and outgoing traffic.

{% highlight bash %}
tcpdump -s 0 -n -i any -w /tmp/$(hostname)-$(date +"%Y-%m-%d-%H-%M-%S").pcap
{% endhighlight %}

Once you have some packet captured, it's helpful to analyze what you have. This command will show you all of the traffic, content included. You'll want to change the `/path/to/file.pcap` and `tcp port 80` to the correct values.

{% highlight bash %}
tcpdump -A -r /path/to/file.pcap 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
{% endhighlight %}

To show just the HTTP headers, use this variant of the command above.

{% highlight bash %}
tcpdump -A -r /path/to/file.pcap 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)' | \
egrep --line-buffered "^........(GET |HTTP\/|POST |HEAD )|^[A-Za-z0-9-]+: " | \
sed -r 's/^........(GET |HTTP\/|POST |HEAD )/\n\1/g'
{% endhighlight %}

Sometimes you'll want to remove the extra filters, that's the stuff that looks like: `(((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)`