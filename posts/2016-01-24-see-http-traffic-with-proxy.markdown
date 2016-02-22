---
title: Proxy yourself
tags: infosec, proxy, mitmproxy
description: Proxy yourself
---

Often times we want to see our network traffic in an organized way,
one tool we can use is `tcpdump` but its a pretty low level tool and
often times HTTP is all we actually care about. 

# mitmproxy (man in the middle proxy)

I've been using `mitmproxy`, its an incredible Python tool that is
designed for HTTP and I especially love the command line ncurses
interface. Its designed for man in the middle attacks but you can also
use it as a debugging tool. Its installable with `pip` but I just
settle for the version that is present on the package manager. With
`mitmproxy` we'll see all the HTTP traffic in a clean and organized
way; `mitmproxy` is MUCH nicer than using squid and the icky
configurations that come along with squid.

# Example

This example was tested and works on Ubuntu 14.04 and also worked on
an Ubuntu 14.04 VM running on VMWare on OS X.

Essentially we will send all IP traffic from our local machine through
`mitmproxy` as a proxy, this is apparently called a local transparent
proxy.

First we set up some rules for the Linux kernel:

I'm going to assume you have another Unix account named mitm\_account,
yes two accounts are needed.

```shell
$ sudo iptables -t nat -A OUTPUT -p tcp \
  -m owner ! --uid-owner mitm_account \
  --dport 443 -j REDIRECT --to-port 9001
```

This looks complicated, you can read up on the `iptables` man page for
all the nitty gritty details. I will try to get a `OS X` equivalent
as well. We also do this same command over, but change `--dport` to 80
for regular HTTP traffic as well.

Note the `mitm_account`. This is the name of some other account,
you'll need to have two Unix accounts for this to work and
`mitm_account` is the Unix account we'll use that will actually run
the `mitmproxy` program.

Then we'll open another shell and change users to mitm\_account and
run:

```shell
$ mitmproxy -T -p 9001
```

And this will start the proxy interceptions.
