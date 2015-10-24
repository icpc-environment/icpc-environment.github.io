---
title:  DOMjudge Automatic Login
date:   2015-10-24 12:19:55
layout: post
tags: DOMjudge, squid
---

Our contest runs on [DOMjudge](http://domjudge.org) and uses php session based authentication. We have
to do this because we run multi-site and can't use anything like IP based logins
or something like that. A pain point we've had in the past is that we can't give
out the credentials to the teams otherwise there would be potential for them to
share their login with someone else.

In the past this has meant we needed volunteers to go around to the team computers
and manually log each team in at the beginning of the practice round, and again
after the computers have been cleaned at the beginning of the real contest. In
practice this also means we have to trust more people with the team passwords so
we can get everyone logged in on time.

### Solution

This year we finally came up with a solution. Each workstation runs a local squid
proxy, which combined with [SquidGuard](http://www.squidguard.org/) is what we use
to restrict access to the contestants on the internet.

Since all the team traffic passes through our local proxy, we can use it to inject
the team's credentials without them ever having access to them. Then we just have
to configure each computer once, and squid will take care of actually logging them
in after that.

From the team perspective, they click on login, see this page below, then just
have to click on the "Log in using computer credentials" and they're done.
![Autologin Page](/images/autologin-page.png)

### The technical details

The way this works is squid watches for a certain url pattern(DOMjudge pages
that require login) When it finds a pattern that matches it add a few extra
headers to the request with the team login and password. So every time the
team goes to a login page, DOMjudge will see these headers and be able to act
on them to log the team in.

The relevant part of `squid.conf` looks like this:

{% highlight squid %}
acl autologin url_regex ^http://the.domjudge.server/team/
acl autologin url_regex ^http://the.domjudge.server/public/login.php$
include /etc/squid3/autologin.conf
{% endhighlight %}

And `/etc/squid3/autologin.conf`:

{% highlight squid %}
request_header_add X-DOMjudge-Autologin true autologin
request_header_add X-DOMjudge-Login "domjudge_teamname" autologin
request_header_add X-DOMjudge-Pass "base64-encoded-password" autologin
{% endhighlight %}

What this does is set up an acl(named `autologin`) to match pages where the login
information might be needed, and then to add those 3 headers on any page that
matches. The permissions on the `/etc/squid3/autologin.conf` file are `640` so that
the team isn't able to just go in and read them.

A few changes were also required on the DOMjudge side to check for credentials
in the headers instead of the `$_POST['user']` and `$_POST['pass']` variables.
See the short patch below based off the DOMjudge 5.0 branch.

{% highlight diff %}
---
 lib/www/auth.php | 21 +++++++++++++++++++--
 1 file changed, 19 insertions(+), 2 deletions(-)

diff --git a/lib/www/auth.php b/lib/www/auth.php
index 5e72263..5acfcb6 100644
--- a/lib/www/auth.php
+++ b/lib/www/auth.php
@@ -153,6 +153,17 @@ function show_loginpage()
 Please supply your credentials below, or contact a staff member for assistance.
 </p>

+<?php
+if (isset($_SERVER['HTTP_X_DOMJUDGE_AUTOLOGIN']) && $_SERVER['HTTP_X_DOMJUDGE_AUTOLOGIN']=='true'){ ?>
+<p>You can log in using credentials that have been configured by your administrator.</p>
+<form action="<?php echo $_SERVER['PHP_SELF']; ?>" method="post">
+<input type="hidden" name="cmd" value="login" />
+<input type="hidden" name="autologin" value="true">
+<input type="submit" value="Log in using computer credentials">
+</form>
+<?php } // endif X_DOMJUDGE_AUTOLOGIN ?>
+
+
 <form action="<?php echo $_SERVER['PHP_SELF'] ?>" method="post">
 <input type="hidden" name="cmd" value="login" />
 <table>
@@ -243,8 +254,14 @@ function do_login()
 	// some specializations are handled by if-statements.
 	case 'IPADDRESS':
 	case 'PHP_SESSIONS':
-		$user = trim($_POST['login']);
-		$pass = trim($_POST['passwd']);
+		// Check for autologin headers
+		if (isset($_POST['autologin']) && isset($_SERVER['HTTP_X_DOMJUDGE_AUTOLOGIN']) && $_SERVER['HTTP_X_DOMJUDGE_AUTOLOGIN'] == 'true') {
+			$user = trim($_SERVER['HTTP_X_DOMJUDGE_LOGIN']);
+			$pass = base64_decode(trim($_SERVER['HTTP_X_DOMJUDGE_PASS']));
+		} else {
+			$user = trim($_POST['login']);
+			$pass = trim($_POST['passwd']);
+		}

 		$title = 'Authenticate user';
 		$menu = false;
{% endhighlight %}
