---
description: Безопасность
---

# Глава 22

## Chapter 22. Security

We spend our time searching for security and hate it when we get it.

John Steinbeck

Security for your Asterisk system is critical, especially if the system is exposed to the internet. There is a lot of money to be made by attackers in exploiting systems to make free phone calls. This chapter provides advice on how to provide stronger security for your VoIP deployment.

## Scanning for Valid Accounts

If you expose your Asterisk system to the public internet, one of the things you will almost certainly see is a scan for valid accounts. [Example 22-1](22.%20Security%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22example-accountscan) contains log entries from one of the authors’ production Asterisk systems.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch22.html%22%20/l%20%22idm46178396108536) This scan began with checking various common usernames, then later went on to scan for numbered accounts. It is common for people to name SIP accounts the same as extensions on the PBX. This scan takes advantage of that fact.

**Tip**

Use non-numeric usernames for your VoIP accounts to make them harder to guess. For example, in this book we use the MAC address of a SIP phone as its account name in Asterisk.

#### Example 22-1. Log excerpts from account scanning

\[Aug 22 15:17:15\] NOTICE\[25690\] chan\_sip.c: Registration from

'"123"&lt;sip:123@127.0.0.1&gt;' failed for '203.86.167.220:5061' - No matching peer

found

\[Aug 22 15:17:15\] NOTICE\[25690\] chan\_sip.c: Registration from

'"1234"&lt;sip:1234@127.0.0.1&gt;' failed for '203.86.167.220:5061' - No matching peer

found

\[Aug 22 15:17:15\] NOTICE\[25690\] chan\_sip.c: Registration from

'"12345"&lt;sip:12345@127.0.0.1&gt;' failed for '203.86.167.220:5061' - No matching peer

found

...

\[Aug 22 15:17:17\] NOTICE\[25690\] chan\_sip.c: Registration from

'"100"&lt;sip:100@127.0.0.1&gt;' failed for '203.86.167.220:5061' - No matching peer found

\[Aug 22 15:17:17\] NOTICE\[25690\] chan\_sip.c: Registration from

'"101"&lt;sip:101@127.0.0.1&gt;' failed for '203.86.167.220:5061' - No matching peer found

The logs on any system will be full of intrusion attempts. This is simply the nature of connecting systems to the internet. In this chapter, we will discuss some of the ways to configure your system so that it will have robust mechanisms to deal with these things.

## Authentication Weaknesses

The first section of this chapter discussed scanning for usernames. Even if you have usernames that are difficult to guess, it is critical that you have strong passwords as well. If an attacker is able to obtain a valid username, they will likely attempt to brute-force the password. Strong passwords make this much more difficult.

The default authentication scheme of the SIP protocol is weak. Authentication is done using an MD5 challenge-and-response mechanism. If an attacker is able to capture any call traffic, such as a SIP call made from a laptop on an open wireless network, it will be much easier to work on brute-forcing the password, since it will not require authentication requests to the server.

**Tip**

Use strong passwords. There are countless resources available on the internet that help define what constitutes a strong password. There are also many strong password generators available. Use them!

## Fail2ban

The previous two sections discussed attacks involving scanning for valid usernames and brute-forcing passwords. [Fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page) is an application that can watch your Asterisk logs and update firewall rules to block the source of an attack in response to too many failed authentication attempts.

**Tip**

Use Fail2ban when exposing Voice over IP services on untrusted networks. It will automatically update the firewall rules to block the sources of attacks.

### Installation

Fail2ban is available as a package in many distributions. Alternatively, you can install it from source by downloading it from the Fail2ban website. To install Fail2ban on RHEL, you must have the EPEL repository enabled \(which was handled during [Chapter 3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22asterisk-Install)\). You can install Fail2ban by running the following command:

$ sudo yum install fail2ban

**Note**

The installation of Fail2ban from a package will include a startup script to ensure that it runs when the machine boots up. If you install from source, make sure that you take the necessary steps to ensure that Fail2ban is always running.

### Configuration

First up, we’ll want to configure the security log in Asterisk, which Fail2ban is able to make use of.

$ sudo vim /etc/asterisk/logger.conf

Uncomment the \(or add a\) line that reads security =&gt; security, and edit the dateformat so Fail2ban understands the logfile.

\[general\]

exec\_after\_rotate=gzip -9 ${filename}.2;

dateformat = %F %T

\[logfiles\]

;debug =&gt; debug

security =&gt; security

;console =&gt; notice,warning,error,verbose

console =&gt; notice,warning,error,debug

messages =&gt; notice,warning,error

full =&gt; notice,warning,error,debug,verbose,dtmf,fax

Then reload the Asterisk logger:

$ sudo asterisk -rx 'logger reload'

Since current versions of Fail2ban already come with an Asterisk jail definition, all we need to do is enable it:

The current best practice is to create a file /etc/fail2ban/jail.local for this purpose \(technically you can put it in /etc/fail2ban/jail.conf, but this is more likely to be overwritten\):

$ sudo vim /etc/fail2ban/jail.local

\[asterisk\]

enabled = true

filter = asterisk

action = iptables-allports\[name=ASTERISK, protocol=all\]

 sendmail\[name=ASTERISK, dest=me@shifteight.org, sender=fail2ban@shifteight.org\]

logpath = /var/log/asterisk/messages

 /var/log/asterisk/security

maxretry = 5

findtime = 21600

bantime = 86400

We’ve set up the ban for 24 hours, but you can do longer or shorter times as well if you prefer \(the bantime is defined in seconds, so calculate accordingly\). Since most attacking hosts move on after a few hours, there’s no harm in unblocking an IP after 24 hours. If the host attacks again, they’ll be blocked again.

Oh, you might also want to tell it to ignore your IP \(or any other IP addresses that are OK to receive connection attempts from\). If you haven’t yet accidentally gotten yourself blocked because you were doing some lab work and misregistering, don’t worry, you will eventually do this to yourself \(unless, of course, you create an ignore list for appropriate IPs\).

\[DEFAULT\]

ignoreip = &lt;ip address\(es\), separated by commas&gt;

\[asterisk\]

enabled = true

filter = asterisk

action = iptables-allports\[name=ASTERISK, protocol=all\]

 sendmail\[name=ASTERISK, dest=me@shifteight.org, sender=fail2ban@shifteight.org\]

logpath = /var/log/asterisk/messages

 /var/log/asterisk/security

maxretry = 5

findtime = 21600

bantime = 86400

Restart Fail2ban and you’re good to go.

$ sudo systemctl reload fail2ban

Test it out if you can, from an IP address you don’t mind being blocked \(for example, an extra computer in your lab that can be the test subject for this\). Attempt to register using bad credentials, and after five attempts \(or whatever you set maxretry to\), that IP should be blocked.

You can see what addresses the Asterisk jail is blocking with the command:

$ sudo fail2ban-client status asterisk

And if you want to unblock an IP,[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch22.html%22%20/l%20%22idm46178396052456) the following command should do so.

$ sudo fail2ban-client set asterisk unbanip ip to unban

More information about Fail2ban can be found at the [Fail2ban wiki](http://www.fail2ban.org/wiki/index.php/Main_Page).

## Encrypted Media

While we gave examples in this book that used encryption, be aware that you can configure SIP so that media will be sent unencrypted. In that case, anyone intercepting the RTP traffic between two SIP peers will be able to use fairly simple tools to extract the audio from those calls.

## Dialplan Vulnerabilities

The Asterisk dialplan is another area where taking security into consideration is critical. The dialplan can be broken down into multiple contexts to provide access control to extensions. For example, you may want to allow your office phones to make calls out through your service provider. However, you do not want to allow anonymous callers that come into your main company menu to be able to then dial out through your service provider. Use contexts to ensure that only the callers you intend have access to services that cost you money.

**Tip**

Build dialplan contexts with great care. Also, avoid putting any extensions that could cost you money in the \[default\] context.

One of the more recent Asterisk dialplan vulnerabilities to have been discovered and published is the idea of dialplan injection. A dialplan injection vulnerability begins with an extension that has a pattern that ends with the match-all character, a period. Take this extension as an example:

exten =&gt; \_X.,1,Dial\(PJSIP/otherserver/${EXTEN},30\)

The pattern for this extension matches all extensions \(of any length\) that begin with a digit. Patterns like this are pretty common and convenient. The extension then sends this call over to another server using the IAX2 protocol, with a dial timeout of 30 seconds. Note the usage of the ${EXTEN} variable here. That’s where the vulnerability exists.

In the world of Voice over IP, there is no reason that a dialed extension must be numeric. In fact, it is quite common using SIP to be able to dial someone by name. Since it is possible for non-numeric characters to be a part of a dialed extension, what would happen if someone sent a call to this extension?

1234&DAHDI/g1/12565551212

A call like this is an attempt at exploiting a dialplan injection vulnerability. In the previous extension definition, once ${EXTEN} has been evaluated, the actual Dial\(\) statement that will be executed is:

exten =&gt; \_X.,1,Dial\(PJSIP/otherserver/1234&DAHDI/g1/12565551212,30\)

If the system has a PRI configured, this call will cause a call to go out on the PRI to a number chosen by the attacker, even though you did not explicitly grant access to the PRI to that caller. This problem can quickly cost you a whole lot of money.

There are several approaches to avoiding this problem. The first and easiest approach is to always use strict pattern matching. If you know the length of extensions you are expecting and expect only numeric extensions, use a strict numeric pattern match. For example, this would work if you are expecting four-digit numeric extensions only:

exten =&gt; \_XXXX,1,Dial\(PJSIP/otherserver/${EXTEN},30\)

Another approach to mitigating dialplan injection vulnerabilities is by using the FILTER\(\) dialplan function. Perhaps you would like to allow numeric extensions of any length. FILTER\(\) makes that easy to achieve safely:

exten =&gt; \_X.,1,Set\(SAFE\_EXTEN=${FILTER\(0-9A-F,${EXTEN}\)}\)

 same =&gt; n,Dial\(PJSIP/otherserver/${SAFE\_EXTEN},30\)

For more information about the syntax for the FILTER\(\) dialplan function, see the output of the core show function FILTER command at the Asterisk CLI.

A more comprehensive \(but also complex\) approach might be to have all dialed digits validated by functions outside of your dialplan \(for example, database queries that validate the dialed string against user permissions, routing patterns, restriction tables, and so forth\). This is a powerful concept, but beyond the scope of this book.

**Tip**

Be wary of dialplan injection vulnerabilities. Use strict pattern matching or use the FILTER\(\) dialplan function to avoid these problems.

## Securing Asterisk Network APIs

To secure AGI, AMI, and ARI, you will need to carefully consider the following recommended practices:

* Only allow connections directly to the API from localhost/127.0.0.1.
* Use an appropriate framework in between the Asterisk API and your client application, and handle connection security through the framework.
* Control access to the framework and the system through strict firewall rules.

Beyond that, the same sort of security rules and best practices apply that you would follow in any mission-critical web application.

## Other Risk Mitigation

There are other useful features in Asterisk that can be used to mitigate the risk of attacks. The first is to use the permit and deny options to build access control lists \(ACLs\) for privileged accounts. Consider a PBX that has SIP phones on a local network, but also accepts SIP calls from the public internet. Calls coming in over the internet are only granted access to the main company menu, while local SIP phones have the ability to make outbound calls that cost you money. In this case, it is a very good idea to set ACLs to ensure that only devices on your local network can use the accounts for the phones.

In your ps\_endpoints table, the permit and deny options allow you to specify IP addresses, but you can also point to a label in the /etc/asterisk/acl.conf file. In fact, ACLs are accepted almost everywhere that connections to IP services are configured. For example, another useful place for ACLs is in /etc/asterisk/manager.conf, to restrict AMI accounts to the single host that is supposed to be using the manager interface.

ACLs can be defined in /etc/asterisk/acl.conf.

\[named\_acl\_1\]

deny=0.0.0.0/0.0.0.0

permit=10.1.1.50

permit=10.1.1.55

\[named\_acl\_2\] ; Named ACLs support IPv6, as well.

deny=::

permit=::1/128

\[local\_phones\]

deny=0.0.0.0/0.0.0.0

permit=192.168.0.0/255.255.0.0

Once named ACLs have been defined in acl.conf, have Asterisk load them using the reload acl command. Once loaded, they should be available via the Asterisk CLI:

\*CLI&gt; module reload acl

\*CLI&gt; acl show

acl

---

named\_acl\_1

named\_acl\_2

local\_phones

\*CLI&gt; acl show named\_acl\_1

ACL: named\_acl\_1

---------------------------------------------

 0: deny - 0.0.0.0/0.0.0.0

 1: allow - 10.1.1.50/255.255.255.255

 2: allow - 10.1.1.55/255.255.255.255

Now, instead of having to potentially repeat the same permit and deny entries in multiple places, you can apply an ACL by its name. You will find an acl field in the ps\_endpoints table, which you can use to point to a named ACL in the acl.conf file.

mysql&gt; select id,transport,aors,context,disallow,allow,acl from ps\_endpoints;

\|id \|transport \|aors \|context\|disallow\|allow \|acl \|

\|0000f30A0A01\|transport-udp\|0000f30A0A01\|sets \|all \|ulaw \|NULL\|

\|0000f30B0B02\|transport-udp\|0000f30B0B02\|sets \|all \|ulaw \|NULL\|

\|SOFTPHONE\_A \|transport-udp\|SOFTPHONE\_A \|sets \|all \|ulaw,h264,vp8\|NULL\|

\|SOFTPHONE\_B \|transport-udp\|SOFTPHONE\_B \|sets \|all \|ulaw,h264,vp8\|NULL\|

mysql&gt; update ps\_endpoints

 set acl='local\_phones'

 where id in \('0000f30A0A01','0000f30B0B02','SOFTPHONE\_A','SOFTPHONE\_B'\)

 ;

mysql&gt; select id,transport,aors,context,disallow,allow,acl from ps\_endpoints;

\|id \|transport \|aors \|context\|disallow\|allow \|acl \|

\|0000f30A0A01\|transport-udp\|0000f30A0A01\|sets \|all \|ulaw \|local\_phones\|

\|0000f30B0B02\|transport-udp\|0000f30B0B02\|sets \|all \|ulaw \|local\_phones\|

\|SOFTPHONE\_A \|transport-udp\|SOFTPHONE\_A \|sets \|all \|ulaw,h264,vp8\|local\_phones\|

\|SOFTPHONE\_B \|transport-udp\|SOFTPHONE\_B \|sets \|all \|ulaw,h264,vp8\|local\_phones\|

**Tip**

Use ACLs when possible on all privileged accounts for network services.

Another way you can mitigate security risk is by configuring call limits. The recommended method for implementing call limits is to use the GROUP\(\) and GROUP\_COUNT\(\) dialplan functions. Here is an example that limits the number of calls from each SIP peer to no more than two at a time:

exten =&gt; \_X.,1,Set\(GROUP\(users\)=${CHANNEL\(endpoint\)}\)

 same =&gt; n,NoOp\(${CHANNEL\(endpoint\)} : ${GROUP\_COUNT\(${CHANNEL\(endpoint\)}\)} calls\)

 same =&gt; n,GotoIf\($\[${GROUP\_COUNT\(${CHANNEL\(endpoint\)}\)} &gt; 2\]?denied:continue\)

 same =&gt; n\(denied\),NoOp\(There are too many calls up already. Hang up.\)

 same =&gt; n,HangUp\(\)

 same =&gt; n\(continue\),NoOp\(continue processing call as normal here ...\)

**Tip**

Use call limits to ensure that if an account is compromised, it cannot be used to make hundreds of phone calls at a time.

## Resources

Some security vulnerabilities require modifications to the Asterisk source code to resolve. When those issues are discovered, the Asterisk development team puts out new releases that contain only fixes for the security issues, to allow for quick and easy upgrades. When this occurs, the Asterisk development team also publishes a security advisory document that discusses the details of the vulnerability. We recommend that you subscribe to the [asterisk-announce mailing list](http://lists.digium.com/mailman/listinfo/asterisk-announce) to make sure that you know about these issues when they come up.

**Tip**

Subscribe to the asterisk-announce list to stay up to date on Asterisk security vulnerabilities.

One of the most popular tools for SIP account scanning and password cracking is [SIPVicious](http://sipvicious.org/). We strongly encourage that you take a look at it and use it to audit your own systems. If your system is exposed to the internet, others will likely run SIPVicious against it, so make sure that you do that first.

## Conclusion—A Better Idiot

There is a maxim in the technology industry that states, “As soon as something is made idiot-proof, nature will invent a better idiot.” The point of this statement is that no development effort can be considered complete. There is always room for improvement.

When it comes to security, you must always bear in mind that the people who are looking to take advantage of your system are highly motivated. No matter how secure your system is, somebody will always be looking to crack it.

We’re not advocating paranoia, but we are suggesting that what we have written here is by no means the final word on VoIP security. While we have tried to be as comprehensive as we can be in this book, you must accept responsibility for the security of your system.

The criminals are working hard to find weaknesses and exploit them.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch22.html%22%20/l%20%22idm46178396108536-marker) The real IP address has been replaced with 127.0.0.1 in the log entries.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch22.html%22%20/l%20%22idm46178396052456-marker) For example, yourself, because you forgot to define ignoreip...

