---
title: Hacks, cracks, dev culture and Armenia's development in tech 🇦🇲
tags: armenia, security, development, economics
description: security vulns.
---

###Audience
<hr/>

This post is intended for both technical and non-technical people; its
for people interested in the development of programmers, cultures of
professionalism, security, and the development of Armenia as a tech
power. Along the way I will show some security vulnerabilities of
well-known sites in Armenia, non-technical people will be able to
follow along as everything will be explained. I will also introduce
the concept of a bug bounty exchange.

A glossary of technical terms follows the post, might be worth while
to read through those terms first.

**I want to emphasis that the point of this blog post is not to shame
or make fun of any individual or company, rather it is meant to bring
attention to the tech industry, culture in Armenia and what can be
done to further it to a higher average level of excellence and
professionalism.**

### Problem
<hr/>

Specifically, with respect to Armenia, much has been said about the
tech as a shining example of an honest and rapidly growing
industry. However, in my view much of that has been pushed by people
who frankly are little connected to the tech industry writ tech
itself. There are not enough voices heard from the down in the
trenches folks who can provide a truly informed voice.

Armenia's geopolitical situation being what it is makes the tech
industry arguably of high interest to national security as:

1. A robust engine of sustained, organic economic growth. A nation
   cannot wage wars or devote high levels of resources to its military
   without a resilient economy. A strong economy lets us negotiate
   with our enemies from a position of strength.
2. As a nursery and pipeline for cyber-security. Armenians are
   disproportionality effective in every domain, the digital realm
   amplifies that 100X.

For the development of countries, it is better to see a country of
makers and not outsourcers. Why? Because making products, like Uber,
Facebook, etc, is what makes the biggest bucks of all and is best able
to utilize the Armenian people's creative talents. Working as
outsourcing tends to make decent but soulless products, and such
products are made by programmers who simply don't care about what
they're making beyond the salary paid. (Hardly eudaimonia)

With that, Armenia has a lot of outsourcing but the product scene is
coming along nicely although we haven't even seen our first largish
failure yet.

Here are two intertwined things I see in the Armenian tech scene
that need more attention & sustained improvement.

<div style="display: inline-flex;justify-content: center;width: 100%;">
| Problem | Fixable | Solution |
|-------- |:-------:|----------|
| Hacker, developer culture lagging| **Yes** | Invest more in people |
| Cyber-security | **Yes** | Invest more in people|
</div>

What do I mean by developer culture? In a way, it's like
professionalism in other fields which is characterized by devotion to
your craft, not cutting corners, doing things the right way even when
it takes longer and deeply learning the tools of your craft. Sometimes
in Armenian culture people like to cut corners, this is a
fact. Examples include cutting corners in Gyumri's apartment
construction which led to worse outcomes in the 1988 earthquake to the
elevator in my building whose buttons were always off by one. There's
an implicit acceptance of a shrugging your shoulders quality work and it
is reflected in the tech industry as well.

### Examples of shortcoming
<hr/>

Now let's look at a few examples of the manifestations of these
shortcomings. I will now show you three examples of live web
vulnerabilities of three sites in Armenia, each of which was told that
a problem existed but didn't pursue a fix initially although one has
been fixed. All three of them are the result of similar entry level
mistakes.

I found out about these exploits after first posting in the Facebook
iterate chatroom about bots
attacking [silicondzor.com](https://silicondzor.com)

<div style="display: inline-flex;justify-content: center;width: 100%;">
![](/images/bots.png)
</div>

This image is from a server log for `silicondzor.com`. See the line
`GET /.git/ HTTP/1.1`? That's a bot checking if it can get our entire
git repo, all our source code. After posting that, `Sparik
Hayrapetyan` reached out to me and reported some Armenian sites that
were vulnerable to this very exploit!

I verified some of those site and here are how damning it is...

Say we have three websites each ending in a `.am` TLD:

_(the creator field is from attributes given on the public facing
sites)_

<div style="display: inline-flex;justify-content: center;width: 100%;">

| Site | Subject matter | exploit | creator |
|-------- |:-------:|-------|-------|
| siteA | auction | Full git repo, found credentials | [http://voodoo.pro/en](http://voodoo.pro/en)
| siteB | car rental | Full git repo, found credentials | sitemax
| siteC | reading materials | Full git repo | [https://www.studio-one.am](https://www.studio-one.am)
</div>


What does it mean to have the full git repo? It means to have the
entire history of the source code from beginning to end.

Verification pics:

### SiteA -- the auction site

First about the programmer culture, this particular code base uses
PHP. A programming language known to be inherently defective in
security and one that is usally avoided for new projects in Silicon
Valley.

<div style="display: inline-flex;justify-content: center;width: 100%;">
![](/images/generated-code-site-a.png)
</div>

Not only are they using PHP, they are also using an IDE to
**generate** PHP code.

Another bad practice they have their passwords in the source
code. Hint: this is never a good idea. Since as an auction site they
are handling all kinds of credit cards and other sensitive information
and now any attacker, including me potentially have access to it
all. (Programmers: A better solution is to use environment variables)

<div style="display: inline-flex;justify-content: center;width: 100%;">
![](/images/credentials-site-a.png)
</div>

SiteA also exposed their **SQL** on the public website
<div style="display: inline-flex;justify-content: center;width: 100%;">
![](/images/sql-dumped.png)
</div>

### SiteB -- Car rental site

The car rental site required a little bit more work to reconstruct the
original git repo. Thankfully using tools
like [GitTools](https://github.com/internetwache/GitTools) makes the
process painless and after some searching through the source code I
found this goodie. These are credentials to the database which being a
car rental site also probably contains some nice credit card numbers,
accounts.

<div style="display: inline-flex;justify-content: center;width: 100%;">
![](/images/site-b-mysql-creds.png)
</div>

### SiteC -- Reading materials

I didn't dig too deeply in this repo but here's an example
structure. The indentation means hierarchy of directories and files,
things ending in `.php` are source code files.

<div style="display: inline-flex;justify-content: center;width: 100%;">
![](/images/site-c.png)
</div>

Studio-One proudly boasts of its clients including `AmeriaBank` and
the `National Assembly of RA`. How much do you want to bet that they
do similar sloppy coding across all these projects?

<div style="display: inline-flex;justify-content: center;width: 100%;">
![](/images/site-c-boast.png)
</div>

Again this exploit was rather simple, it was a rookie mistake.

### Bug Bounties

Now as I mentioned Sparik first reported exploits to the respective
site owners but amazingly some didn't even reply or simply asked him
leave his email address, the de facto equivalent of "Don't call us,
we'll call you". Amazingly Sparik wasn't compensated at all or
recognized! I reached out as well but only one replied and has since
fixed the mistakes.

Many other companies would have paid him under a system called a `bug
bounty`. This is when companies pay whoever finds exploits on their
website/app/program under a structured disclosure method. The idea is
that it is better for the company to pay to know about the exploit,
fix it, and move on rather than have the exploit end up on the black
market and then hit them out in the wild. An example of this is the
Target hacking, that cost over $100 Million in damages. Facebook has
been running their bug bounty since 2011 with great success, some
payouts reach $40,000 which is still substantially less than what can
happen when someone malicious literally has all the passwords to your
databases and computers. To my knowledge, there are no companies in
Armenia that run a bug bounty.

### Mitigation

How can we mitigate these kinds of exploits? Well in a way its simple
but also hard & vague; we do it by:

1. Promoting a culture of people open about knowledge, about promoting
   collerboration. This particular exploit is talked about in just
   about every other InfoSec meetup in San Francisco and is well known
   but in Armenia a seemingly prominent firm is repeatedly making
   it. Practically speaking this means more meetups. A check
   on [silicondzor.com](https://silicondzor.com) shows 53 events in
   Feburary for Armenia, that number needs to be higher and a higher
   percentage needs to be programmers talking to other programmers,
   not fluff sessions about Marketing/Startups. The meetups should
   lean toward being workshops with hands on examples and live coding.

2. Sponsoring bug bounties, especially starting with Government &
   military websites.

3. Having more events like the
   recent
   [Capture the Flag](https://www.facebook.com/events/642338025891387)
   which literally included the exploit used in this blog post.

4. Promoting a culture of professionalism & respect for
   programmers. Programmers are not respected as crafts people in
   Armenia.

5. Collective funding of talent. The entire Armenian tech industry
   needs to be willing to spend some money on the collective pool of
   talent. This means like non-trivial prizes for hackathon, paid bug
   bounties, paid trainings Armenian culture in general does not
   promote doing things for free, or helping someone without expecting
   something in return implicitly.

6. Promoting and funding projects that protect critical internet
   infrastructure for Armenia,
   like
   [CERT-AM](https://www.trusted-introducer.org/directory/teams/cert-am.html)

Keep in mind that many of these are happening one way or another, I am
merely enumerating some for record. Because I believe in **doing** and
not merely speaking, I will be creating a public bug bounty exchange
for all of Armenia, it will be listed
on [silicondzor.com](https://silicondzor.com). Once it is up, I
encourage all companies, not just tech companies, to post on the bug
bounty exchange with offers of payment for successful examples of
exploits.

### Parting

(While writing this post a new story broke out of crackers attacking
Armenian
banks,
[Hackers Infiltrate Computer System of Bank in Armenia; Steal $273,000](http://hetq.am/eng/news/75680/hackers-infiltrate-computer-system-of-bank-in-armenia-steal-$273000.html),
I think you could see the connections of what I'm trying to say;
forget criminals for a moment and now imagine a nation-state
determined to destroy us.)

There is a lot of hype and excitement for tech in Armenia and it holds
a huge promise, but we must grow our industry, talent correctly. In
addition, because the topic is Armenia, we must keep in mind about
security as cyber-security is increasingly the first line of attack
and defense in war. Would our enemies be so kind as to tell us about
exploits on our banking sites? Or perhaps military sites or other
critical government? What about the electrical system or other
critical infrastructure that is increasingly connected to the
internet. How effective are `Iskanders` when someone hacks and
reprograms them mid-flight? These questions show the level to which
tech, economy and national security are all interrelated in Armenia
and the level of responsibility the tech industry has to Armenia.

### Glossary

Note:

1. Non-technical people, you should probably start learning the jargon.
2. Technical people, these are loose definition meant for intuition
rather than accuracy.

**source code**: The original code of a program, written by a
programmer. This can be publicably available or private, usually when
its private then it has things like passwords written in it.

**exploit**: A mistake in the source code which then lets another
person use that mistake to cause a program to do something other than
originally intended. I.E. if I find an exploit in the ATM machine,
then maybe I can get it to spit out money.

**InfoSec**: Short for information security, its a subfield of
programming culture and include a focus on security.

**cracker**: A professional hacker who attacks programs, products for
money usually.

**pentest**: Short for penetration testing, its when you pay a
infosec professional to attack your product as if they were a cracker.

**DDoS**: This is when someone abuses whatever service you are
providing by overwhelming you with too many requests for that service
thereby bringing the whole system down.

**git**: A program used by programmers to help them keep track of all
the source code they write, down to the detail of which line of code
was written by whom, when.

**repo**: A directory on the hard drive of a computer created by the
git program. It contains all the source code and whatever else the
programmers decided to keep track of in the project.

**commit**: Something created by git that is like a collection of
saves of source code. Think of it like a record in time of how the
source code look like.

**TLD**: Top level domain, basically the `.com` or `.am` part of a
website name.

**IDE**: A program that some programmers use to help them make new
programs, think of it like a fancy word processor with auto
completion.

**SQL**: A programming language designed for databases. If I know your
SQL then I know how your data is organized, stored, and what I should
be looking for once I get into your system.

**bots**: Computer programs designed to do boring, repetitive
things. In this post it refers to programs designed to search the web
for simple, rookie level exploits.
