---
description: Внешние подключения
---

# Глава 7

> _Вы не всегда можете контролировать то, что происходит снаружи. Но вы всегда можете контролировать то, что происходит внутри._ 
>
> -- Уэйн Дайер

В предыдущих главах мы рассмотрели много важной информации, которая необходима для работы системы Asterisk. Однако нам еще предстоит обсудить то, что жизненно важно для любой АТС: а именно, подключение ее к внешнему миру. В этой главе мы обсудим внешние подключения.

Новаторская архитектура Asterisk была знаменатлеьной в значительной степени из-за того, что она рассматривает все типы каналов как равные. Это отличается от традиционной АТС, где транки \(которые соединяют с внешним миром\) и расширения \(которые соединяются с пользователями и ресурсами\) логически разделены. Тот факт, что диалплан Asterisk обрабатывает все каналы аналогичным образом означает, что в системе Asterisk вы можете очень легко выполнить то, что гораздо сложнее \(или невозможно\) достичь на традиционной АТС.

Однако, эта гибкость имеет цену. Поскольку система по своей сути не знает разницы между внутренним ресурсом \(например, телефонным аппаратом\) и внешним \(например, каналом телефонной связи\), вы должны убедиться, что ваш диалплан обрабатывает каждый тип ресурса соответствующим образом.

## The Basics of Trunking

The purpose of trunking is to provide a shared connection between two entities. Trees have trunks, and everything that passes between the roots and the leaves happens through the trunk. Railroads use the term “trunk” to refer to a major line that connects feeder lines together.

In telecommunications, trunking connects two systems together. Carriers use trunks to connect their networks to each other. In a PBX, the circuits that connect the PBX to the outside world are \(from the perspective of the PBX\) usually referred to as trunks \(although the carriers themselves do not generally consider these to be trunks\). From a technical perspective, the definition of a trunk is not as clear as it used to be \(PBX trunks used totally different technology from station circuits, but now both are usually SIP\), but as a concept, trunks are still important. With SIP, everything is technically peer-to-peer, so from a technology perspective there isn’t really such a thing as a trunk anymore \(or perhaps it’s more accurate to say that everything is a trunk\). From a functional perspective it is still useful to be able to differentiate between VoIP resources that connect to the outside world \(trunks, lines, circuits, etc.\) and VoIP resources that connect to user endpoints \(stations, sets, extensions, handsets, telephones, etc.\).

In an Asterisk PBX, you might have trunks that go to your VoIP provider for in-country long-distance calls, trunks for your overseas calls, and trunks that connect your various offices together. These trunks might actually run across the same network connection, but in your dialplan you could treat them quite differently. You can even have a trunk in Asterisk that simply loops back in on itself \(which is usually some kludgy hack that solves some namespace or CDR problem that wasn’t getting solved any other way\).

## Fundamental Dialplan for Outside Connectivity

In a traditional PBX, external lines are generally accessed by way of an access code that must be dialed before the number.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407857960) It is common to use the digit 9 for this purpose.

In Asterisk, it is similarly possible to assign 9 for routing of external calls, but since the Asterisk dialplan is so much more intelligent, it is not really necessary to force your users to dial 9 before placing a call. Typically, you will have an extension range for your system \(say, 100–199\), and a feature code range \(\*00 to \*99\). Anything outside those ranges that matches the dialing pattern for your country or region can be treated as an external call.

If you have one carrier providing all of your external routing, you can handle your external dialing through a few simple pattern matches. The example in this section is valid for the North American Numbering Plan \(NANP\). If your country is not within the NANP \(which serves Canada, the US, and many Caribbean countries\), you will need a different pattern match.

The \[globals\] section contains two variables, named LOCAL and TOLL.[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407852440) The purpose of these variables is to simplify management of your dialplan should you ever need to change carriers. They allow you to make one change to the dialplan that will affect all places where the specified channel is referenced:

```text
[globals]
; These channels are the same to asterisk as
; any PJSIP enpoint, so they'll be configured
; similar to telephone sets.
; Each carrier will have their own configuration
; requirements (although they'll all be similar)
LOCAL=PJSIP/my-itsp
TOLL=PJSIP/my-other-itsp
```

The \[external\] section contains the actual dialplan code that will recognize the numbers dialed and pass them to the Dial\(\) application:[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407848792)

```text
[external]
exten => _NXXNXXXXXX,1,Dial(${LOCAL}/${EXTEN}) ; 10-digit pattern match for NANP
exten => _NXXXXXX,1,Dial(${LOCAL}/${EXTEN}) ; 7-digit pattern match for NANP
exten => _1NXXNXXXXXX,1,Dial(${TOLL}/${EXTEN}) ; Long-distance pattern match
 ; for NANP
exten => _011.,1,Dial(${TOLL}/${EXTEN}) ; International pattern match for
 ; calls made from NANP
; This section is functionally the same as the above section.
; It is for people who like to dial '9' before their calls
exten => _9NXXNXXXXXX,1,Dial(${LOCAL}/${EXTEN:1})
exten => _9NXXXXXX,1,Dial(${LOCAL}/${EXTEN:1})
exten => _91NXXNXXXXXX,1,Dial(${TOLL}/${EXTEN:1})
exten => _9011.,1,Dial(${TOLL}/${EXTEN:1})
In any context that would be used by sets or user devices, you would use an include => directive to allow access to the external context:
[sets]
include => external
```

{% hint style="warning" %}
**Warning**

It is critically important that you do not include access to the external lines in any context that might process an incoming call. The risk here is that a phishing bot could eventually gain access to your outgoing trunks \(you’d be surprised at how common these phishing bots are\).

We cannot stress enough how important it is that you ensure that no external resource can access your toll lines.
{% endhint %}

## The PSTN

The public switched telephone network \(PSTN\) has existed for over a century. It is the precursor to many of the technologies that shape our world today, from the internet to MP3 players.

The use of old-school PSTN circuits in Asterisk systems is no longer common. The technical complexities, costs, and limitations of obsolete technology are only justified in situations where a reliable internet connection is not available \(and even then, traditional circuits will often be a problematic choice\). Even the carriers themselves have largely switched to VoIP for their internal backbones.

{% hint style="info" %}
**The PSTN Has Retired**

More than any technical factor, perhaps the most significant nail in the PSTNs coffin is the fact that most of the technical experts in the field of traditional telephony are near or past retirement age, and the new kids have no interest in this sort of thing. Point being: you will increasingly find that carriers no longer have the skilled staff required to deploy traditional PSTN services. All the cool kids are learning VoIP \(which is ultimately just a networking technology\), and all the carriers put their best and brightest on the VoIP/SIP side of the business.

So, while it used to be true that you couldn’t beat a PRI circuit for reliability, that is no longer the case. In fact, many companies deliver PRI circuits across a SIP connection, which is a kludge Asterisk has no use for.
{% endhint %}

Where the PSTN might still hold sway for a few years more is in telephone numbers. If VoIP had been invented without the PSTN preceding it, it’s unlikely that something like a phone number would have ever been invented. Still, we’ve got them, and we use them, and the reason we do so is perhaps not so much due to any usefulness they provide, but rather due to the fact that they are managed by a complex, multinational consortium of standards bodies and curators who ensure the integrity of the global call routing plan.

To put the value of this in perspective, it might be worth considering that if the internet had designed the telephone network \(and phone calls were as free as email\), all our SIP phones would likely be ringing all day long with one spammy call after another. That still happens, but it’s greatly reduced by the fact that a phone call costs money, and even if it costs just pennies, that’s enough to keep much of the exceedingly mindless spam out of the game.

Another feature the PSTN offers is standards compliance and interoperability. If you look at any internet-based voice product, they are either proprietary walled gardens, or they are community-driven and have failed to gain any useful traction. It is our belief that this will not change until some sort of trust mechanism exists that ensures the identities of incoming callers have been verified by some widely recognized authority.

### Traditional PSTN Trunks

{% hint style="info" %}
**Note**

This section has been written as a nod to the telecommunications industry, and to the history of Asterisk itself. It is in part because Asterisk could talk to so many different sorts of old-school circuits that it achieved the early success it did. These days, the use of these old circuits has for the most part faded into history.
{% endhint %}

There are two types of fundamental technology that PSTN carriers have used to deliver telephone circuits: analog and digital.

#### Analog telephony

The first telephone networks were all analog. The audio signal that you generated with your voice was used to generate an electrical signal, which was carried to the other end. The electrical signal had the same characteristics as the sound being produced.

Analog circuits have several characteristics that differentiate them from other circuits you might wish to connect to Asterisk:

* No signaling channel exists—line state signaling is electromechanical, and addressing is done using in-band audio tones.
* Disconnect supervision is usually delayed by several seconds, and is not completely reliable.
* Far-end supervision is minimal \(for example, answer supervision is lacking\).
* Differences in circuits mean that audio characteristics will vary from circuit to circuit and will require tuning.

Incoming analog circuits that you wish to connect to your Asterisk system will need to connect to a Foreign eXchange Office \(FXO\) port. Since there is no such thing as an FXO port in any standard computer, an FXO port of some sort must be provided to the system before you can connect traditional analog lines. Companies such as Digium and Sangoma offer such cards, but you can also purchase a SIP device that provides such ports.

{% hint style="danger" %}
**FXO and FXS**

For any analog circuit, there are two ends: the office \(typically the central office of the PSTN\) and the station \(typically a phone, but could also be a card such as a modem or line card in a PBX\).

The central office is responsible for:

* Power on the line \(nominally 48 volts DC\)
* Ringing voltage \(nominally 90 volts AC\)
* Providing dialtone
* Detecting hook state \(off-hook and on-hook\)
* Sending supplementary signaling such as caller ID

The station is responsible for:

* Providing a ringer \(or at least being able to handle ringing voltage in some manner\)
* Providing a dialpad \(or some way of sending DTMF\)
* Providing a hook switch to indicate the status of the line

A Foreign eXchange \(FX\) port is named by what it connects to, not by what it does. So, for example, a Foreign eXchange Office \(FXO\) port is actually a station: it connects to the central office. A Foreign eXchange Station \(FXS\) port is actually a port that provides the services of a central office \(in other words, you would plug an analog set into an FXS port\).

Note that changing from FXO to FXS is not something you can simply do with a settings change. FXO and FXS ports require completely different electronics.

This stuff is old-school, folks. You can run old phones from 100 years ago off an FXS port!
{% endhint %}

We do not recommend the use of analog trunks in an Asterisk system. Their configuration and use is outside of the scope of this book.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407792344)

#### Digital telephony

Digital telephony was developed in order to overcome many of the limitations of analog. Some of the benefits of digital circuits include:

* No loss of amplitude over long distances
* Reduced noise on circuits \(especially long-distance circuits\)
* Ability to carry more than one call per physical circuit
* Faster call setup and teardown
* Richer signaling information \(especially if using ISDN\)
* Lower cost for carriers
* Lower cost for customers \(at higher densities\)

There were several fundamental digital circuits that gained wide acceptance in the telecommunications industry:

T1 \(24 channels\)

Used in Canada and the United States \(mostly for ISDN-PRI\)[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407778472)

E1 \(32 channels\)

Used in the rest of the world \(ISDN-PRI or MFC/R2\)

BRI \(2 channels\)

Used for ISDN-BRI circuits \(Euro-ISDN\)

Note that the physical circuit can be further defined by the protocol running on the circuit. For example, a T1 could be used for either ISDN-PRI or CAS, and an E1 could be used for ISDN-PRI, CAS, or MFC/R2.

It is difficult to justify these circuit types anymore. Relative to VoIP protocols, they have become expensive, complex, and somewhat inflexible. If you need to connect such circuits to an Asterisk system, we recommend some sort of gateway device to convert the circuit into SIP, and then connect via SIP to your Asterisk system. If you want a single-chassis system, companies such as Digium and Sangoma offer digital PSTN cards that can be installed directly into your Asterisk server; they are connected to Asterisk by way of the DAHDI channel driver. The use of this technology is outside of the scope of this book.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407773192)

## VoIP

Compared to the lengthy history of the telecommunications industry,[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407767752) VoIP is still a relatively new concept. For the century or so prior to VoIP, the only way to connect your site to the PSTN was through the use of circuits provided for that purpose by your local telephone company. VoIP now allows for connections between endpoints without the PSTN having to be involved at all \(although in most VoIP scenarios, there will still be a PSTN component at some point, especially if there is a traditional E.164 phone number involved\). The PSTN still controls the phone numbers, and we’ll be using those until somebody comes up with an internet-based addressing mechanism that is not subject to abuse the way email has been.[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407766216)

### Network Address Translation

If you are going to be using VoIP across any sort of wide-area network \(such as the internet\), you will be dealing with firewalls, and quite frequently network address translation \(NAT\) as well.[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407758680) A basic understanding of how the SIP and RTP protocols work together to create a VoIP call can be helpful in understanding and debugging functional problems \(such as the “one-way audio” issue NAT configuration errors can often create\). NAT allows a single external IP address to be shared by multiple devices behind a router. Since NAT is typically handled in the firewall, it also forms part of the security layer between a private network and the internet.

A VoIP call using SIP doesn’t consist just of the signaling messages to set up the call \(the SIP part of the connection\). It also requires the RTP streams \(the media\), which carry the actual audio connection,[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407755912) as shown in [Figure 7-1](7.%20Outside%20Connectivity%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22idp220832_ch07).

![Figure 7-1. SIP and RTP](.gitbook/assets/0%20%283%29.png)

The use of separate protocols to carry the audio is what can make NAT traversal troublesome for VoIP connections, especially if the remote phones are behind one NAT, and the PBX is behind a different NAT. The problem is caused by the fact that while the SIP signaling will typically be allowed to pass through the firewalls at both ends, the RTP streams may not be recognized as part of the SIP session taking place, and thus will be ignored or blocked, as shown in [Figure 7-2](7.%20Outside%20Connectivity%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22idp224128_ch07). The effect of one or both of the RTP streams being blocked is that users will complain that they are seeing their calls happen, and can answer them, but cannot hear \(or cannot be heard\).

![Figure 7-2. RTP blocked by firewall](.gitbook/assets/1%20%282%29.png)

In this section we will discuss some of the methods you may employ to alleviate issues caused by NAT. There are two different scenarios that need to be considered, each requiring you to define parameters within the pjsip.conf file. NAT issues can be annoying to troubleshoot, as there are many different types of firewalls in production, and many different ways to configure them.

In general, you’re going to want to add the following to the transport sections of your /etc/asterisk/pjsip.conf file:

```text
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0
local_net=x.x.x.x/xx ; IP/CIDR of your internal network
external_media_address=x.x.x.x ; External IP address to use in RTP handling
external_signaling_address=x.x.x.x ; External address for SIP signalling
```

{% hint style="info" %}
**Note**

If you want to find the external address of your PBX, run the following from the shell:

`$ dig +short myip.opendns.com @resolver1.opendns.com`
{% endhint %}

\*\*\*\*

It is probably safe to set these parameters for all scenarios, but be prepared to experiment by commenting out settings and reloading PJSIP to test different scenarios.

#### Endpoints behind NAT

If your telephone sets are behind a remote NAT, there may be options in the ps\_endpoints table of your database that should be adjusted from the default settings. You will need to experiment with changing the following values between 'yes' and 'no'. There may be many different combinations possible.

```text
MySQL> update ps_endpoints set rtp_symmetric='yes',force_rport='yes',rewrite_contact='yes'
```

Other parameters you may wish to look at include media\_address and direct\_media.

Keep in mind the defaults when you are making changes. If in doubt, set the value of a field you’ve changed to NULL, as that will effectively set it back to the default.

{% hint style="success" %}
**Keeping a Remote Firewall Open**

Sometimes a problem with a SIP telephone will surface wherein the phone will register and function when it is first booted, but then it will suddenly become unreachable. What is often happening here is that the remote firewall, seeing no activity coming from the set, will close the external connection to the telephone, and thus the PBX will lose the ability to signal to the set. The effect is that if the PBX tries to send a call to the phone, it will fail to connect \(the remote firewall will reject the connection\). If, on the other hand, the user makes a call out, for a few minutes the set will again be able to accept incoming calls. Naturally this can cause a lot of confusion for the users.

A relatively simple solution to this problem[11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407725320) involves setting the registration timer on the remote phone to a low enough value that it will stimulate the connection every minute or so, and thus convince the firewall that this connection can be allowed to exist for a little while longer. It’s a bit of a hack, but it has proven successful. The challenge with proposing a universal solution is that there are many different models of firewalls, from inexpensive consumer-grade units to complex session border controllers, and this is one of the few solutions that seems to address the problem reliably in almost all cases.

This approach is best on smaller systems \(fewer than 100 telephones\). A larger system with hundreds or thousands of phones will not be well served by this solution, as there will be an increased load on the system due to a near-constant flood of registrations from remote phones. In such a case, some more careful thought will need to be given to the overall design \(for example, a dedicated registrar server could be used in place of Asterisk to handle the registration traffic\).

In a perfect world, you would be able to specify a particular model of firewall, and devise a configuration for those firewalls that would ensure your SIP traffic was properly handled. In reality, you will come up against not just different models of firewall, but even different firmware versions for the same model firewall.
{% endhint %}

#### Asterisk behind NAT

First up, we need to tell you that putting your PBX behind a NAT is not recommended. It’s far better to ensure it is firewalled without a NAT layer \(especially if you have endpoints that are not on the same network as the PBX\).

If you are stuck working with a PBX behind a NAT, you will need to work with your network team to ensure that the NAT components of the network are being correctly handled by your NAT device \(typically your firewall\). If they do not have sufficient skill in this regard, you may require the services of an outside consultant who is skilled in NAT traversal for SIP/RTP traffic. As we said, having your PBX behind NAT is not recommended.

Typically, the endpoints will also be behind NAT, and thus you will have a double-NAT scenario, which is likely to require a few hours of wrangling with various settings, not only in Asterisk but also in the firewall, in order to achieve success. Remember that it is vital that you test audio in both directions; it’s not enough to simply verify that calls can be dialed and answered.

In a scenario where there is no choice but to use a double-NAT, we would recommend finding out whether a VPN can be used between the PBX and the remote endpoints. In many cases this will end up being easier to configure.

We wish we could say there’s a simple, reliable way to ensure NAT works in all cases, but unfortunately there is not.

You could also look into using NAT assisting technologies such as STUN, TURN, and ICE. The details of these are beyond the scope of this book, since they require external servers, but many folks have had success with those protocols where other methods failed.

### PSTN Termination and Origination

Passing calls between a VoIP environment and the PSTN requires some sort of gateway to convert the VoIP \(typically SIP\) signaling into something compatible with PSTN protocols. These processes are referred to as origination and termination \([Figure 7-3](7.%20Outside%20Connectivity%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig0703)\).

![Figure 7-3. PSTN origination and termination](.gitbook/assets/2%20%282%29.png)

People often confuse the terms origination and termination as to which is which. For us, it’s useful to remember that since the PSTN was already there when VoIP came along, the terms evolved in relation to it. Ideally, the processes should probably be called PSTN origination, and PSTN termination, and we encourage you to remember them that way.[12](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407704584)

#### PSTN termination

Until VoIP totally replaces the PSTN, there will be a need to connect calls from VoIP networks to the public telephone network. This process is referred to as termination \([Figure 7-4](7.%20Outside%20Connectivity%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig0704)\).

![Figure 7-4. PSTN termination](.gitbook/assets/3.png)

Although you can engineer an Asterisk system to act as a termination gateway \(using some sort of PSTN interfaces\), in practice you’re more likely to use an Internet Telephony Service Provider \(ITSP, also sometimes called a VoIP carrier\) to terminate your phone calls. ITSPs typically have a massive investment in infrastructure, and you’d be hard-pressed to do better without spending a ton of money. ITSPs have made termination inexpensive, so it’s tough to make a business case for doing it yourself.

{% hint style="success" %}
If you really need to connect your Asterisk system directly to the PSTN, you’ll need the following:

* Appropriate circuit\(s\) from a PSTN telco \(analog, BRI, PRI, SS7, MFC/R2, etc.\)
* Suitable hardware to connect to that circuit \(FXO, BRI, T1, E1, etc.\)
* Echo cancellation \(hardware or software\)
* The skills necessary to properly configure your equipment for the carrier you’re dealing with \(there are many flavors of each of these circuit types, and this can be difficult to get right even for those who know the technology well\)[13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407692712)

Beyond that, you will often have to handle a far more complex routing logic, which takes into consideration things like geography, corporate policy, cost, available resources, and so forth.
{% endhint %}

In order to send your calls to an ITSP, your dialplan needs to look something like this:

```text
; NANP-based systems
[to-pstn] ; Yes, we're going through an ITSP, but the PSTN is our destination
exten => _1NXXNXXXXXX.,1,Dial(${TOLL}/${EXTEN}) ; Country code plus phone number
; Add a '1' and send
exten => _NXXNXXXXXX.,1,Dial(${LOCAL}/1${EXTEN}) ; Country code plus phone number
; Strip off the '011' and send
exten => _011X.,1,Dial(${TOLL}/${EXTEN:3}) ; Country code plus phone number
; Emergency dialing
exten => 911,1,Dial(${LOCAL}/911) ; Defining this will require info from your carrier

; Most of the rest of the world
[to-pstn]
; Strip off NDD prefix, add country code, and send
exten => _0X.,1,Dial(${TOLL}/<add your country code here>${EXTEN:1})
; Strip off IDD prefix and send
exten => _00X,1,Dial(${LOCAL}/${EXTEN:2}) ; Country code plus phone number
; Emergency dialing (and other services)
exten => 11X,1,Dial(${LOCAL}/${EXTEN}) ; Defining this will require info from your carrier
```

{% hint style="warning" %}
**Warning**

Given that most PSTN circuits will allow you to dial any number, anywhere in the world, and given that you will be expected to pay for all incurred charges, we cannot stress enough the importance of security on any gateway machine that is providing PSTN termination. Criminals put a lot of effort into cracking phone systems \(especially poorly secured Asterisk systems\), and if you do not pay careful attention to all aspects of security, you will be the victim of toll fraud. It’s only a matter of time.

_Do not allow any unsecured VoIP connections into any context that contains PSTN termination._
{% endhint %}

Termination will tend to be more complex than we’ve outlined here—even if you’re using an ITSP as your carrier—but the basic concept is fairly straightforward: match a pattern that your users might dial, prepare it for the carrier by removing or adding necessary digits, and send the call out the appropriate PJSIP endpoint \(trunk\). We’ve only discussed the dialplan here; in a later section we’ll discuss how to configure the SIP trunks to carry this traffic.

#### PSTN origination

You might also want to be able to accept calls from the PSTN into your VoIP network. The process of doing this is commonly referred to as origination. This simply means that the call originated in the PSTN \([Figure 7-5](7.%20Outside%20Connectivity%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig0705)\).

![Figure 7-5. PSTN origination](.gitbook/assets/4.png)

In order to provide origination, a phone number is required.

In the good old days, when VoIP and Asterisk were new, it was quite common for people to handle the circuit connection to the PSTN themselves, using analog or digital trunks provided by the local phone company. For the most part this type of connection is now handled by ITSPs, and you simply need to connect your system to your VoIP carrier across a SIP trunk.

Phone numbers—when used for the purpose of origination—are commonly called DIDs \(Direct Inward Dialing numbers\). Your carrier will send a call down the circuit to your system, and pass the DID \(or special received digits in some cases[14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407677720)\), which the Asterisk dialplan will interpret. In other words, you will need a dialplan context that accepts incoming calls from your carrier, with extensions or patterns that will correlate to your DIDs.

In order to accept a call from a VoIP circuit, you will need to handle the digits the carrier will send you \(the DID or phone number\). The DNIS number and the DID do not have to match, but typically they will. In days gone by, the carrier would usually ask you in what format you wish to receive the digits. Nowadays, a VoIP carrier will typically tell you what format they will send, and you are expected to accommodate. Two common formats are: DNIS \(which is essentially the digits of the DID that was called\) or E.164, which means that they’ll be including the country code with the number.

In the dialplan, you associate the incoming circuit with a context that will know how to handle the incoming digits. As an example, it could look something like this:

```text
[from-pstn]
exten => 4165550100,1,Goto(sets,100,1)
exten => 4165550101,1,Goto(sets,101,1)
exten => 4165550102,1,Goto(sets,102,1)
exten => 4165550103,1,Goto(sets,103,1)
exten => 4165554321,1,Goto(main-menu,${EXTEN},1)
exten => 4165559876,1,VoiceMailMain() ; a handy back door for listening
 ; to voice messages
exten => i,1,Verbose(2,Incoming call to invalid number)
```

In the number-mapping context, you explicitly list all of the DIDs that you expect to handle, plus an invalid handler for any DIDs that are not listed \(you could send invalid numbers to reception, or to an automated attendant, or to some context that plays an invalid prompt\).

Now we’re ready to discuss how to configure trunks to carry your external traffic.

### Configuring SIP Trunks

SIP is far and away the most popular of the VoIP protocols—so much so that the terms VoIP and SIP have almost come to mean the same thing. In previous editions of this book, we’ve looked at some of the other protocols that were popular at the time \(primarily IAX2 and H.323\), but for this edition there’s no real reason anymore to discuss anything but SIP. The channel drivers for those older protocols are still available in Asterisk, but they’re no longer supported.

The SIP protocol is peer-to-peer and does not really have a formal trunk specification. This means that whether you are connecting a single phone to your server or connecting two servers together, the SIP connections will be similar. Having said that, there are some differences in the style of how these resources can be configured, and there will definitely be a difference in how your dialplan handles routing across trunks.

#### Connecting an Asterisk system to a SIP provider

It is quite common to use the same ITSP carrier for termination and origination, but be aware that the two processes are unrelated to each other. If calls going in one direction pass your testing, that doesn’t mean calls in the other direction are OK. If you change configuration, test routing both in and out, every time.

Many carriers will provide sample configurations for Asterisk. Unfortunately, these documents generally refer to the older chan\_sip driver, which has been deprecated. Digium has designed a PJSIP wizard that is intended to greatly simplify carrier configuration. You can still configure ITSP trunks using the exact same methods we’ve shown before for configuring other endpoints \(creating records in ps\_endpoint, ps\_aors, ps\_auths, and so forth\), but rather than hash over all that again, we are going to take a look at the wizard, since it consolidates several components into a single configuration file. We have found that since user endpoints change often, and carrier endpoints seldom do, it’s often useful to configure carriers in a configuration file rather than in the database.

Before any config can be created, however, it’s important to determine how the carrier will interact with your system. There are two fundamental models we have seen:

Password-based authentication, including registration[15](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407656840)

This is common in smaller carriers focused on the small-business market. This is also the type of service you would get if you were simply registering a SIP phone directly to a service.

IP-based authentication

No password; no registration. This is more common with carriers that provide bulk trunking services to larger enterprises and resellers. \(Typically these will also come with some sort of minimum commitment in terms of volume.\) You will be expected to have solid SIP and networking skills.

These are not hard-and-fast rules, but they are the most common in our experience.

So there are two ways we might configure an ITSP in the /etc/asterisk/pjsip\_wizard.conf file.

First, if the carrier uses an IP address–based authentication, they will expect you to send your traffic from a static IP address \(and should your address change, you will need to inform them so they can reconfigure their equipment\). Your pjsip\_wizard.conf file could then look something like this:

```text
; ITSP uses IP address-based authentication
[itsp-no-auth]
type=wizard
remote_hosts=itsp.example.com
endpoint/context=pstn-in
endpoint/allow = !all,ulaw,g722
sends_registrations=no
accepts_registrations=no
sends_auth=no
accepts_auth=no
```

Alternatively, if your IP address changes frequently \(or your carrier requires this method\), you can have your system register to the carrier \(which will require you to send authentication credentials to prove it’s really you\). Your calls will typically also be required to authenticate:

```text
[itsp-with-auth]
type=wizard
remote_hosts=itsp.example.com
endpoint/context=pstn-in
endpoint/allow = !all,ulaw,g722
sends_registration=yes
accepts_registrations=no
sends_auth=yes
accepts_auth=no
outbound_auth/username=itsp_provided_username
outbound_auth/password= itsp_provided_password
```

Note that the names \[itsp-no-auth\] and \[itsp-with-auth\] have no built-in meaning to Asterisk. They become the PJSIP channel names to which you send your calls.

**Configure trunks for termination**

The PJSIP wizard has created the channel definitions we require for our carrier. To send a call, we only need to make a minor change to the \[globals\] section of our extensions.conf file, as follows:

```text
[globals]
UserA_DeskPhone=PJSIP/0000f30A0A01
UserA_SoftPhone=PJSIP/SOFTPHONE_A
UserB_DeskPhone=PJSIP/0000f30B0B02
UserB_SoftPhone=PJSIP/SOFTPHONE_B
TOLL=PJSIP/itsp-no-auth
LOCAL=${TOLL}
;OR
;TOLL=PJSIP/itsp-with-auth
;LOCAL=${TOLL}
```

**Configuring trunks for origination**

For your incoming calls, you’ll need a context in your /etc/asterisk/extensions.conf file that matches the context specified for the ITSP channel. Let’s assume we have two NANP DIDs, 4169671111 and 4167363636, and place the required code above the \[sets\] context:

```text
TOLL=PJSIP/itsp-no-auth
LOCAL=${TOLL}
;OR
;TOLL=PJSIP/itsp-with-auth
;LOCAL=${TOLL}
[pstn-in]
exten => 4169671111,1,Dial(sets,100,1)
exten => 4167363636,1,Dial(sets,101,1)
[sets]
exten => 100,1,Dial(${UserA_DeskPhone})
```

In a small system, this is fairly easy to administer. In a larger system, you’d want to put the DIDs into a table in your database, and have the dialplan look up the required target. We’ll be diving into databases a bit more later in the book.[16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407641272)

That’s the gist of it as far as carrier interconnection is concerned. It can seem very complicated to set this up because there are a lot of options, but at a high level it’s fairly straightforward. Problems are usually found to be minor configuration mismatches. Be methodical, and please, please, please be paranoid about security!

## Emergency Dialing

In North America, people are used to being able to dial 911 in order to reach emergency services. Outside of North America, well-known emergency numbers are 112 and 999. If you make your Asterisk system available to people, you are obligated \(in many cases regulated\) to ensure that calls can be made to emergency services from any telephone connected to the system \(even from phones that otherwise are restricted from making calls\).

One of the essential pieces of information the emergency response organization needs to know is where the emergency is \(e.g., where to send the fire trucks\). In a traditional PSTN trunk, this information is already known by the carrier and is subsequently passed along to whatever local authority handles these tasks \(in Canada and the US, these are called Public Safety Answering Points, or PSAP\). With VoIP circuits things can get a bit more complicated, by virtue of the fact that they are not physically tied to any geographical location.

You need to ensure that your system will properly handle emergency calls from any phone connected to it, and you need to communicate what is available to your users. As an example, if you allow users to register to the system from softphones on their laptops, what happens if they are in a hotel room in another country, and somebody dials 911?[17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407626504)

The dialplan for handling emergency calls does not need to be complicated. In fact, it’s far better to keep it simple. People are often tempted to implement all sorts of fancy functionality in the emergency services portions of their dialplans, but if a bug in one of your fancy features causes an emergency call to fail, lives could be at risk. This is no place for playing around. The \[emergency-services\] section of your dialplan might look something like this:

```text
[emergency-services]
exten => 911,1,Goto(dialpsap,1)
exten => 9911,1,Goto(dialpsap,1) ; some people will dial '9' because
 ; they're used to doing that from the PBX
exten => 999,1,Goto(dialpsap,1)
exten => 112,1,Goto(dialpsap,1)
exten => dialpsap,1,Verbose(1,Call initiated to PSAP!)
 same => n,Dial(${LOCAL}/911) ; REPLACE 911 HERE WITH WHATEVER
 ; IS APPROPRIATE TO YOUR AREA
[internal]
include => emergency-services ; you have to have this in any context
 ; that has users in it
```

In contexts where you know the users are not on-site \(for example, remote users with their laptops\), something like this might be best instead:

```text
[no-emergency-services]
exten => 911,1,Goto(nopsap,1)
exten => 9911,1,Goto(nopsap,1) ; for people who dial '9' before external calls
exten => 999,1,Goto(nopsap,1)
exten => 112,1,Goto(nopsap,1)
exten => nopsap,1,Verbose(1,Call initiated to PSAP!)
 same => n,Playback(no-emerg-service) ; you'll need to record this prompt
[remote-users]
include => no-emergency-services
```

In North America, regulations have obligated many VoIP carriers to offer what is popularly known as E911.[18](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407618072) When you sign up for their services, they will require address information for each DID that you wish to associate with outgoing calls. This address information will then be sent to the PSAP appropriate to that address, and your emergency calls should be handled the same way they would be if they were dialed on a traditional PSTN circuit.

The bottom line is that you need to make sure that the phone system you create handles emergency calls in accordance with local regulations and user expectations.

## Вывод

Обычно прогнозируется, что ТфОП в конечном итоге полностью исчезнет. Однако, прежде чем это произойдет, потребуется широко используемый и надежный распределенный механизм, который позволит организациям и отдельным лицам публиковать адресную информацию, чтобы их можно было найти. Любая голосовая технология, которая не использует PSTN, в настоящее время является либо запатентованным продуктом стен-сада, либо игровой площадкой спамеров и преступников. Мы подозреваем, что ТфОП будет сущесвтвовать еще некоторое время, и если это так, то возникновение и завершение должны быть частью вашей системы Asterisk.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407857960-marker) In a key system, each line has a corresponding button on each telephone, and lines are accessed by pressing the desired line key.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407852440-marker) You can name these anything you wish.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407848792-marker) For more information on pattern matches, see [Chapter 6](glava-06.md).

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407792344-marker) We should note, however, that we’ve written extensively on the subject in the past, and that body of work has been released under Creative Commons licensing, and is freely available.

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407778472-marker) There’s also a circuit used in Japan called a J-1, which is most simply described as a 24-channel E1.

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407773192-marker) We would again like to note that we’ve written extensively about digital circuits and DAHDI in previous editions, and that body of work has been released under Creative Commons licensing, and is freely available. Also, Sangoma/Digium provide detailed instructions on how to install and configure their PSTN cards. If you are looking to deploy this technology, please enlist the services of a professional technical resource. This stuff is complex and nuanced, and is not something you’re going to enjoy playing with if you haven’t had some sort of previous experience. It is not necessary to learning or understanding Asterisk.

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407767752-marker) When we say “lengthy” we mean that in relation to other electronic technologies.

[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407766216-marker) Just try to imagine if your telephone number could be spammed the way your email address is. The fact that the PSTN is regulated, costs money to use, and controls the telephone numbers has served to limit the plague of spam that email has suffered.

[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407758680-marker) [The Wikipedia page on network address translation](http://bit.ly/2InpK6S) is comprehensive and useful. For more information about different types of NAT, and how NAT operates in general, start there.

[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407755912-marker) SIP is not the only protocol to use RTP to carry media streams.

[11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407725320-marker) Whether it is the best solution is still up for debate.

[12](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407704584-marker) And perhaps even use them that way in conversation, since many people are confused by these terms, and few will admit it when you’re talking to them.

[13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407692712-marker) Trust us, Jim Van Meggelen worked with this stuff for many years before getting into VoIP.

[14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407677720-marker) In traditional PBXs, the purpose of DIDs was to allow connection directly to an extension in the office. Many PBXs could not support concepts such as number translation or flexible digit lengths, and thus the carrier had to pass the extension number, rather than the number that was dialed \(which was also referred to as the DNIS number, from Directory Number Information Service\). For example, the phone number 416-555-1234 might have been mapped to extension 100, and thus the carrier would have sent the digits 100 to the PBX instead of the DNIS of 4165551234. If you ever replace an old PBX with an Asterisk system, you may find this translation in place, and you’ll need to obtain a list of mappings between the numbers that the caller dials and the numbers that are sent to the PBX. It was also common to see the carrier only pass the last four digits of the DNIS number, which the PBX then translates into an internal number. With VoIP trunks this will seldom be the case, but be aware that it is possible.

[15](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407656840-marker) Remember that registration is simply a mechanism whereby a SIP endpoint tells a registrar server where it is located. This is useful if your IP address changes, as might be the case on a consumer or small-business type of internet connection \(such as DSL or cable\).

[16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407641272-marker) A table to handle this would simply need a field for the DID, and three more for the target context, extension, and priority.

[17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407626504-marker) Don’t assume this can’t happen. When somebody calls 911, it’s because they have an emergency, and it’s not safe to assume that they’re going to be in a rational state of mind. A recording that tells your softphone users what address the system is going to be sending to the PSAP may clue them in to the fact that the fire trucks aren’t going to be sent to where they’re needed. \(“This telephone is registered to our Toronto system. Emergency services will be sent to our office at 301 Front St W. Press 1 to proceed with this call.”\).

[18](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22idm46178407618072-marker) It’s not actually the carrier that’s offering this; rather it’s a capability of the PSAP. E911 is also used on PSTN trunks, but since that happens without any involvement on your part \(the PSTN carriers handle the paperwork for you\), you are generally not aware that you have E911 on your local lines.

