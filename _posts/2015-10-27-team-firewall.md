---
title:  Configuring the Team Firewall
date:   2015-10-27 16:22:00
layout: post
---

One thing that needs to be done during the programming contest is restrict teams
from accessing the network. We run DOMjudge on the internet however so we can't
just cut the team workstations off entirely. We also need some network access
for teams to be able to print.

The simple solution is to just use iptables to restrict outbound traffic to a
set of trusted ip addresses and/or ports. This works as long as you can prepare
a list of IPs ahead of time and trust them not to change. We have multiple sites
and never prepare that early so it is difficult for us to to do this effectively.
We also moved our DOMjudge installation into Amazon Web Services and we don't
have a well known static ip address anymore.

The solution we've come up with solves these problems and results in a more secure
set up than before. First we add an iptables rule to block all traffic coming from
the contestant user id to the outside world. This effectively stops them from
talking to anything on the network except for the local machine.

{% highlight bash %}
# block all traffic from the contestant user
iptables -t filter -A output -m owner --uid-owner contestant -j REJECT
{% endhighlight %}


Next we install and configure [squid](http://www.squid-cache.org/) with
[squidguard](http://www.squidguard.org/). SquidGuard allows us to set up a
whitelist of domains or urls that the proxy is allowed to access. Anything that
isn't on the whitelist we get to redirect to our own custom block page. Since we
can use dns names here we don't run into the problem of having to know ip addresses
ahead of time.

The proxy runs on localhost, so the teams are allowed to contact it. Printing
still works because the cups service is what actually communicates with the
printer and it runs as a different user than the contestant.

We also use squid to handle automatic login of our teams into domjudge. You can
read more about that here: <a href="{% post_url 2015-10-24-domjudge-autologin %}">DOMjudge Automatic Login</a>

### Technical Details

For the technically inclined you'll want to take a look at the following files:

* [ansible firewall playbook](https://github.com/icpc-environment/icpc-env/blob/master/playbooks/firewall.yml)
* [squid config](https://github.com/icpc-environment/icpc-env/blob/master/files/squid/squid.conf.j2)
* [squidguard config](https://github.com/icpc-environment/icpc-env/blob/master/files/squid/squidGuard.conf.j2)
