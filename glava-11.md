---
description: 'Функции АТС, включая парковку, пейджинг и конференц-связь'
---

# Глава 11

> _Я не верю в ангелов, нет. Но у меня есть крошечный парковочный ангел. Он у меня на приборной панели, и ты его заводишь. Крылья хлопают, и это должно дать вам место для парковки. Это работало до сих пор._ 
>
> -- Билли Коннолли

This chapter discusses several peripheral features common to business telephone environments. We’ll briefly cover the features.conf file, and then spend a few sections on paging and parking, and finally do a bit of work with Asterisk’s conferencing engine, confbridge.

First up, let’s copy the features.conf file over from the installation directory, and have a look at it:

```text
$ sudo cp ~/src/asterisk-16.<TAB>/configs/samples/features.conf.sample \
/etc/asterisk/features.conf
$ sudo chown asterisk:asterisk /etc/asterisk/features.conf
```

## features.conf

Asterisk provides several features common to most PBXs, many of which have optional parameters. The features.conf file is where you can adjust or define the various feature parameters in Asterisk.

**DTMF-Based Features**

Many of the parameters in features.conf only apply when invoked on calls that have been bridged by the dialplan applications Dial\(\) or Queue\(\) using one or more of the options K, k, H, h, T, t, W, w, X, or x. Features accessed in this way are DTMF-based \(meaning they can’t be accessed via SIP messaging, but only through touch-tone signals in the audio channel triggered by users dialing the required digits on their dialpads\).[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406491144)

Transfers on SIP channels \(for example, from a SIP telephone\) can be handled using the capabilities of the phone itself and won’t be affected by anything in the features.conf file.

### The \[general\] Section

In the \[general\] section of features.conf, you can define options that fine-tune the behavior of the transfers feature in Asterisk. These have nothing to do with how SIP telephones handle call transfers. You instead access these features by using DTMF digits while on a call \(the call must be bridged, so calls ringing or in progress will not have access to these features\).

The features.conf.sample file in your ~/asterisk/ folder contains details of the various options, and examples of how they can be set.

These features are not as commonly used as they were in the past, mostly because many of these things can be handled in more advanced ways than firing DTMF from the telephone set \(for example, through an external integration of some sort, or for that matter from the telephone itself using its own internal transfer features\).

### The \[featuremap\] Section

The \[featuremap\] section, summarized in [Table 11-1](10.%20Deeper%20into%20the%20Dialplan%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AdditionalConfig_id243783), allows you to define specific DTMF sequences that will trigger features on channels that have been bridged via options in the Dial\(\) or Queue\(\) application. The two options you are most likely to use are parkcall and automixmon.

Table 11-1. features.conf \[featuremap\] section

| Option | Value/example | Notes | Dial\(\)/Queue\(\) flags |
| :--- | :--- | :--- | :--- |
| blindxfer | \#1 | Invokes a blind \(unsupervised\) transfer | T, t |
| disconnect | \*0 | Hangs up the call | H, h |
| automon | \*1 | Starts recording of the current call using the Monitor\(\) application \(pressing this key sequence a second time stops the recording\) | W, w |
| atxfer | \*2 | Performs an automated transfer | T, t |
| parkcall | \#72 | Parks a call | K, k |
| automixmon | \*3 | Starts recording of the current call using the MixMonitor\(\) application \(pressing this key sequence again stops the recording\) | X, x |

### The \[applicationmap\] Section

The \[applicationmap\] section of features.conf is arguably the most nifty, as it allows you to map DTMF codes to dialplan applications. The caller will be placed on hold until the application has completed execution.

The syntax for defining an application map is as follows \(it must appear on a single line; line breaks are not allowed\):[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406451032)

Name =&gt; DTMF\_sequence,ActivateOn\[/ActivatedBy\],App\(\[Args\]\)\[,MOH\_Class\]

What you are doing is the following:

1. Giving your map a name so that it can be enabled in the dialplan through the use of the DYNAMIC\_FEATURES channel variable \(more on this in a moment\).
2. Defining the DTMF sequence that activates this feature \(we recommend using at least two digits for this\).
3. Defining which channel the feature will be activated on, and \(optionally\) which participant is allowed to activate the feature \(the default is to allow both channels to use/activate this feature\).
4. Giving the name of the application that this map will trigger, and its arguments.
5. Providing an optional music on hold \(MOH\) class to assign to this feature \(which the opposite channel will hear when the application is executing\). If you do not define any MOH class, the caller will hear only silence.

Here is an example of an application map that will trigger an AGI script:[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406440184)

agi\_test =&gt; \*6,self/callee,AGI\(agi-test.agi\),default

You may add this to your /etc/asterisk/features.conf file if you wish.

**Note**

Since applications spawned from the application map are run outside the PBX core, you cannot execute any applications that trigger the dialplan \(such as Goto\(\), Macro\(\), Background\(\), etc.\). If you wish to use the application map to spawn external processes \(including executing dialplan code\), you will need to trigger an external application through an AGI\(\) call or the System\(\) application. The point is, if you want anything complex to happen through the use of an application map, you will need to test very carefully, as not all things will work as you might expect.

To use an application map, you must declare it in the dialplan by setting the DYNAMIC\_FEATURES variable somewhere before the Dial\(\) command that connects the channels. Use the double underscore modifier on the variable name to ensure that the application map is available to both channels throughout the life of the call. Let’s toss this one in our subDialUser subroutine, so that it’ll be available whenever any of our extensions call each other:

\[subDialUser\]

exten =&gt; \_\[0-9\].,1,Noop\(Dial extension ${EXTEN},channel: ${ARG1}, mailbox: ${ARG2}\)

 same =&gt; n,Noop\(mboxcontext: ${ARG3}, timeout ${ARG4}\)

 same =&gt; n,Set\(\_\_DYNAMIC\_FEATURES=agi\_test\)

 same =&gt; n,Dial\(${ARG1},${ARG4}\)

 same =&gt; n,GotoIf\($\["${DIALSTATUS}" = "BUSY"\]?busy:unavail\)

**Note**

If you want to allow more than one application map to be available on a call, you will need to use the \# symbol as a delimiter between multiple map names:

* Set\(\_\_DYNAMIC\_FEATURES=agi\_test\#my\_other\_map\)

The reason why the \# character was chosen instead of a simple comma is that older versions of the Set\(\) application interpreted the comma differently than more recent versions, and the syntax for application maps has never been updated.

Don’t forget to reload the features module after making changes to the features.conf file:

\*CLI&gt; module reload features

You can verify that your changes have taken place through the CLI command features show.

Also, since we are introducing an AGI script here, there are some commands that must be run to make the referenced AGI script available to Asterisk.

$ sudo cp ~/src/asterisk-16.&lt;TAB&gt;/agi/agi-test.agi /var/lib/asterisk/agi-bin/

$ sudo chown asterisk:asterisk /var/lib/asterisk/agi-bin/\*

$ sudo chmod 755 /var/lib/asterisk/agi-bin/\*

Make sure you test out your application map before you turn it over to your users!

**Dynamic Feature-Map Creation from the Dialplan**

You can create feature maps from the dialplan directly, making the definition of a feature \(and its DTMF mapping\) dynamic, on a per-channel basis. This is done with the FEATURE\(\) and FEATUREMAP\(\) dialplan functions. Valid values for FEATUREMAP\(\) include the following, which set or retrieve the DTMF sequence used to trigger the functionality:

atxfer

Attended transfer

blindxfer

Blind transfer

automon

Auto Monitor\(\) \(call recording\)

disconnect

Call disconnect

parkcall

Call parking

automixmon

Auto MixMonitor\(\) \(call recording\)

With FEATUREMAP\(\), the function can be used to retrieve the current DTMF sequence for that functionality:

exten =&gt; 232,1,Noop\(Current DTMF for parkcall: ${FEATUREMAP\(parkcall\)}\)

Or you can use the DTMF sequence for a feature function on the current channel:

exten =&gt; 233,1,NoOp\(\)

 same =&gt; n,Set\(FEATUREMAP\(parkcall\)=\*9\)

 same =&gt; n,Noop\(DTMF for parkcall now: ${FEATUREMAP\(parkcall\)}\)

If you want to set the parking timeout for a channel, you can do so with the FEATURE\(\) function. It contains a single argument, parkingtime, which is a value in seconds before the parked call is returned to the caller \(or destination, depending on how you’ve configured parking\):

exten =&gt; 234,1,NoOp\(\)

 same =&gt; n,Set\(FEATURE\(parkingtime\)=60\)

### Application Map Grouping

If you have a lot of features that you need to activate for a particular context or extension, you can group several features together in an application map grouping, so that one assignment of the DYNAMIC\_FEATURES variable will assign all of the designated features of that map.

The application map groupings are added at the end of the features.conf file. Each grouping is given a name, and then the relevant features are listed:

\[shifteight\]

unpauseMonitor =&gt; \*1 ; custom key mapping

pauseMonitor =&gt; \*2 ; custom key mapping

agi\_test =&gt; ; no custom key mapping

**Note**

If you want to specify a custom key mapping to a feature in an application map grouping, simply follow the =&gt; with the key mapping you want. If you do not specify a key mapping, the default key map for that feature will be used \(as found in the \[featuremap\] section\). Regardless of whether you want to assign a custom key mapping or not, the =&gt; operator is required.

In the dialplan, you would assign this application map grouping with the Set\(\) application:

 same =&gt; Set\(\_\_DYNAMIC\_FEATURES=shifteight\) ; use the double underscore if you want

 ; to ensure both call legs have the

 ; variable assigned.

## Parking and Paging

Although these two features are completely separate from each other, they are so commonly used together we might as well treat them as one single feature.

Call parking allows calls to be placed on hold and then retrieved from a location different from where they were originally answered. Paging uses a public address system to allow announcements to be sent from the telephone system \(for example, to announce who the parked call is for and how it can be retrieved\).

Some businesses, perhaps with large warehouses, outdoor areas, or employees who move around the office a lot, utilize the paging and parking functionality of their systems to direct calls around the office. In this chapter we’ll show you how to use both parking and paging in the traditional setting \(park ’n’ page\), along with a couple of more modern takes on these commonly used functions.

### Call Parking

A parking lot allows a call to be held in the system without being associated with a particular extension. The call can then be retrieved by anyone who knows the park code for that call. This feature is often used in conjunction with a public address \(PA\), or “paging” system. For this reason, it is often referred to as “park-and-page.” It should be noted that parking and paging are in fact separate. We’ll cover paging momentarily, but first, let’s talk about call parking.

Let’s grab a copy of the sample file that we’ll use to configure call parking:

$ sudo cp ~/src/asterisk-16.&lt;TAB&gt;/configs/samples/res\_parking.conf.sample \

/etc/asterisk/res\_parking.conf

$ sudo chown asterisk:asterisk /etc/asterisk/res\_parking.conf

$ sudo asterisk -rx 'module load res\_parking.so'

To park a call in Asterisk, you need to transfer the caller to the feature code assigned to parking, which is assigned in the res\_parking.conf file with the parkext directive. By default, this is 700:

parkext =&gt; 700 ; What extension to dial to park \(all parking lots\)

You have to wait to complete the transfer until you get the number of the parking retrieval slot from the system, or you will have no way of retrieving the call. By default the retrieval slots, assigned with the parkpos directive in res\_parking.conf, are numbered from 701 to 720:

parkpos =&gt; 701-720 ; What extensions to park calls on \(defafult parking lot\)

Once the call is parked, anyone on the system can retrieve it by dialing the number of the retrieval slot \(parkpos\) assigned to that call. The call will then be bridged to the channel that dialed the retrieval code.

There are two common ways to define how retrieval slots are assigned. This is done with the findslot directive in the res\_parking.conf file. The default method \(findslot =&gt; first\) always uses the lowest-numbered slot if it is available, and only assigns higher-numbered codes if required. The second method \(findslot =&gt; next\) will rotate through the retrieval codes with each successive park, returning to the first retrieval code after the last one has been used. Which method you choose will depend on how busy your parking lots are. If you use parking rarely, the default findslot of first will be best \(people will be used to their parked calls always being in the same slot\). If you constantly use the parking feature \(for example, in an automobile dealership\), it is far better for each successive page to assign the next slot, since you will often have more than one call parked at a time. Your users will get used to listening carefully to the actual parking lot number \(instead of just always dialing 701\), and this will minimize the chance of people accidentally retrieving the wrong call on a busy system.

**Handling Timed-Out Parked Calls with the comebacktoorigin Option**

This option configures the behavior of call parking when the parked call times out \(see the parkingtime option\). comebacktoorigin can have one of two values:

yes \(default\)

When the parked call timeout is exceeded, Asterisk will attempt to send the call back to the peer that parked this call. If the channel is no longer available to Asterisk, the caller will be disconnected.

no

This option would be used when you want to perform custom dialplan functionality on parked calls that have exceeded their timeouts. The caller will be sent into a specific area of the dialplan where logic can be applied to gracefully handle the remainder of the call \(this may involve simply returning the call to a different extension, or performing a lookup of some sort\).

You also may need to take into account calls where the originating channel cannot handle a returned parked call. If, for example, the call was parked by a channel that is also a trunk to another system, there would not be enough information to send the call back to the correct person on that other system. The actions following a timeout would be more complex than comebacktoorigin=yes could handle gracefully.

Parked calls that timeout with comebacktoorigin=no will always be sent into the parkedcallstimeout context.

**Note**

The dialplan \(and contexts\) were discussed in detail in [Chapter 6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics).

The extension they will be sent to will be built from the name of the channel that parked the call. For example, if a SIP peer named 0004F2040808 parked this call, the extension will be SIP\_0004F2040808.

If this extension does not exist, the call will be sent to the s extension in the parkedcallstimeout context instead. Finally, if the s extension of parkedcallstimeout does not exist, the call will be sent to the s extension of the default context.

Additionally, for any calls where comebacktoorigin=no, there will be an extension of SIP\_0004F2040808 created in the park-dial context. This extension will be set up to do a Dial\(\) to SIP/0004F2040808.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406331512)

If you are using parking, you are also going to need a way to announce the parked calls so users know how to retrieve them. While you could just run down the hall yelling “Bob, there’s a call for you on 701!,” the more professional method is to use a paging system \(more formally known as a public address system\), which we will discuss now.

### Paging \(aka Public Address\)

In many PBX systems, it is useful to connect the telephone system to some sort of public address system. This involves dialing a feature code or extension that makes a connection to a public address resource of some kind, and then making an announcement through the handset of the telephone that is broadcast to all devices associated with that paging resource \(perhaps you’ve heard the clerk at your grocery store request a price check through the telephone\). Often, this will be an external paging system consisting of an amplifier connected to overhead speakers; however, paging through the speakers of office telephones is also popular \(mainly for cost reasons\). If you have the budget \(or an existing overhead paging system\), overhead paging is generally better, but set-based paging can work well in many environments. It is also possible to have a mix of set-based and overhead paging, where, for example, set-based paging might be in use for the administrative offices, but overhead paging would be used for warehouse, hallway, parking lot, and public areas \(cafeteria, reception, etc.\).

In Asterisk, the Page\(\) application is used for paging. This application simply takes a list of channels as its argument, calls all of the listed channels simultaneously, and, as they are answered, puts each one into a conference room. With this in mind, it becomes obvious that one requirement for paging to work is that each destination channel must be able to automatically answer the incoming connection and place the call onto a speaker of some sort \(in other words, Page\(\) won’t work if all the phones just ring\).

So, while the Page\(\) application itself is painless and simple to use, getting all the destination channels to handle the incoming pages correctly is a bit trickier. We’ll get to that shortly.

The Page\(\) application takes three arguments: 1\) the group of channels the page is to be connected to, 2\) the options, and 3\) the timeout:

exten =&gt; \*724,1,Noop\(Page\)

 same =&gt; n,Set\(ChannelsToPage=${UserA\_DeskPhone}&${UserA\_SoftPhone}}&${UserB\_DeskPhone}\)

 same =&gt; n,Page\(${ChannelsToPage},i,120\)

The options \(outlined in [Table 11-2](10.%20Deeper%20into%20the%20Dialplan%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AdditionalConfig_id257235)\) give you some flexibility with respect to how Page\(\) works, but the majority of the configuration is going to have to do with how the target devices handle the incoming connection. We’ll dive into the various ways you can configure devices to receive pages in the next section.

Table 11-2. Page\(\) options

| Option | Description | Discussion |
| :--- | :--- | :--- |
| d | Enables full-duplex audio | Sometimes referred to as “talkback paging,” the use of this option implies that the equipment that receives the page has the ability to transmit audio back at the same time as it is receiving audio. Generally, you don’t want to use this unless you have a specific need for it. |
| i | Ignores attempts to forward the call | You would normally want this option enabled, because a call-forwarded set could go pretty much anywhere, and that’s not where your page needs to go. |
| q | Does not play beep to caller \(quiet mode\) | Normally you won’t use this, since it’s good for paging to make a sound to alert people that a page is about to happen. However, if you have an external amplifier that provides its own tone, you may want to set this option. |
| r | Records the page into a file | If you intended to use the same page multiple times in the future, you could record the page and then use it again later by triggering it using Originate\(\) or using the A\(x\) option to Page\(\). |
| s | Dials a channel only if the device state is NOT\_INUSE | This option is likely only useful \(and reliable\) on SIP-bound channels, and even so may not work if a single line is allowed to host multiple calls simultaneously \(quite common with SIP phones\). Therefore, don’t rely on this option in all cases. |
| A\(x\) | Plays announcement x to all participants | You could use a previously recorded file to be played over the paging system. If you combined this with Originate\(\) and Record\(\), you could implement a delayed paging system. |
| n | Does not play announcement simultaneously to caller \(implies A\(x\)\) | By default, the system will play the paged audio to both the caller and the callee. If this option is enabled, the paged audio will not be played to the caller \(the person paging\). |

**Warning**

Because of how Page\(\) works, it is very resource-intensive. We cannot stress this enough. Carefully read on, and we’ll cover how to ensure that paging does not cause performance problems in a production environment \(which it is almost certain to do if not designed correctly\).

### Places to Send Your Pages

As we stated before, Page\(\) is, in and of itself, very simple. The trick is how to bring it all together. Pages can be sent to different kinds of channels, and they all require different configuration.

#### External paging

If a public address system is installed in the building, it is common to connect the telephone system to an external amplifier and send pages to it through a call to a channel. The best way we know of doing this is to use an FXS device of some kind \(such as an ATA\), which is connected through a paging interface such as a Bogen UTI1,[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406290408) which then connects to the paging amplifier.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406289064)

Another popular way to connect to a paging system is to plug the output of the sound card of your Asterisk server into the paging amplifier, and send calls to the channel named Console/DSP. We don’t like this method because while it may seem inexpensive and simple, in practice it can be time-consuming. It asumes that the sound drivers on your server are working correctly, the audio levels are normalized correctly on that channel, your server has a decent on-board audio card, the grounding is good, and...well, in our opinion, this way is not recommended.[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406286520)

In your dialplan, paging to an external amplifier would look similar to a simple Dial\(\) to the device that is connected to the paging equipment. You would need to configure the ATA the same as any SIP telephone \(through ps\_endpoints, ps\_auth, etc., in the database\), named something like PagingATA. You then plug the ATA into a Bogen UTI1, and to page you have this dialplan code:

exten =&gt; \*725,1,Verbose\(2,Paging to external amplifier\) ; The '\*' is part of what you dial

 same =&gt; n,Set\(PageDevice=PJSIP/PagingATA\) ; This probably belongs in \[globals\]

 same =&gt; n,Page\(${PageDevice},i,120\)

You can name this device anything you want \(for example, we often use the MAC address as the name of a SIP device\), but for anything that is not a user telephone, it can be helpful to use a name that makes it stand out from other devices.

There are also many SIP-based paging devices on the market \(SIP paging speakers are popular, but—we think—rather expensive for what you get, especially in a large deployment\).

#### Set paging

Set-based paging first became popular in key telephone systems, where the speakers of the office telephones are used as a pauper’s public address system. Most SIP telephones have the ability to auto-answer a call on handsfree, which accomplishes what is required on a per-telephone basis. In addition to this, however, it is necessary to pass the audio to more than one set at the same time. Asterisk uses its built-in conferencing engine to handle the under-the-hood details. You use the Page\(\) application to make it happen.

Like Dial\(\), the Page\(\) application can handle several channels. Since you will generally want Page\(\) to signal several sets at once \(perhaps even all the sets on your system\), you may end up with lengthy device strings that look something like this:

Page\(PJSIP/SET1&PJSIP/SET2&PJSIP/SET3&PJSIP/SET4&PJSIP/SET5&PJSIP/SET6&PJSIP/SET7&...

**Warning**

Beyond a certain size, your Asterisk system will be unable to page multiple sets. For example, in an office with 200 telephones, using SIP to page every set would not be possible; the traffic and CPU load on your Asterisk server would simply be too much. In cases like this, you should be looking at either multicast paging or external paging.

Perhaps the trickiest part of SIP-based paging is the fact that you usually have to tell each set that it must auto-answer, but different manufacturers of SIP telephones use different SIP messages for this purpose. So, depending on the telephone model you are using, the commands needed to accomplish SIP-based set paging will be different. Here are some examples:

* For Mitel \(FKA Aastra\):

exten =&gt; \*726,1,Verbose\(2,Paging to Aastra sets\)

 same =&gt; n,SIPAddHeader\(Alert-Info: info=alert-autoanswer\)

 same =&gt; n,Set\(PageDevice=SIP/00085D000000\)

 same =&gt; n,Page\(${PageDevice},i\)

* For Polycom:

exten =&gt; \*727,1,Verbose\(2,Paging to Polycom sets\)

 same =&gt; n,SIPAddHeader\(Alert-Info: Ring Answer\)

 same =&gt; n,Set\(PageDevice=SIP/0004F2000000\)

 same =&gt; n,Page\(${PageDevice},i\)

* For Snom:

exten =&gt; \*728,1,Verbose\(2,Paging to Snom sets\)

 same =&gt; n,Set\(VXML\_URL=intercom=true\)

; replace 'domain.com' with the domain of your system

 same =&gt; n,SIPAddHeader\(Call-Info: sip:domain.com\;answer-after=0\)

 same =&gt; n,Set\(PageDevice=SIP/000413000000\)

 same =&gt; n,Page\(${PageDevice},i\)

* For Cisco SPA \(the former Linksys phones, not the 79XX series\):

exten =&gt; \*729,1,Verbose\(2,Paging to Cisco SPA sets, but not Cisco 79XX sets\)

 same =&gt; n,SIPAddHeader\(Call-Info:\;answer-after=0\) ; Cisco SPA phones

 same =&gt; n,Set\(PageDevice=SIP/0004F2000000\)

 same =&gt; n,Page\(${PageDevice},i\)

Assuming you’ve figured that out, what happens if you have a mix of phones in your environment? How do you control which headers to send to which phones?[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406263240)

Any way you slice it, it’s not pretty.

Fortunately, many of these sets support IP multicast, which is a far better way to send a page to multiple sets \(read on for details\). Still, if you only have a few phones on your system and they are all from the same manufacturer, SIP-based paging could be the simplest method, so we don’t want to scare you off it completely.

#### Multicast paging via the MulticastRTP channel

If you are serious about paging through the sets on your system, and you have more than a handful of phones, you will need to look at using IP multicast. The concept of IP multicast has been around for a long time,[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406256840) but it has not been widely used. Nevertheless, it is ideal for paging within a single location.

Asterisk has a channel \(chan\_multicast\_rtp\) that is designed to create an RTP multicast. This stream is then subscribed to by the various phones, and the result is that whenever media appears on the multicast stream, the phones will pass that media to their speakers.

Since MulticastRTP is a channel driver, it does not have an application, but instead will work anywhere in the dialplan that you might otherwise use a channel. In our case, we’ll be using the Page\(\) application to initiate our multicast.

To use the multicast channel, you simply send a call to it the same as you would to any other channel. The syntax for the channel is as follows:

MulticastRTP/type/ip address:port\[/linksys address:port\]

The type can be either basic or linksys. The basic syntax of the MulticastRTP channel looks like this:

exten =&gt; \*730,1,Page\(MulticastRTP/basic/239.0.0.1:1234\)

Not all sets support IP multicast, but we have tested it out on Snom,[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406245224) Linksys/Cisco, Polycom \(firmware 4.x or later\), and Aastra, and it works very well.

**Multicast Paging on Cisco SPA Telephones**

The multicast paging feature on Cisco SPA phones is a bit strange, but once configured it works fine. The trick of it is that the address you put into the phone is not the multicast address that the page is sent across, but rather a sort of signaling channel.

What we have found is that you can make this address the same as the multicast address, but simply use a different port number.

The dialplan looks like this:

exten =&gt; \*724,1,Page\(MulticastRTP/linksys/239.0.0.1:1234/239.0.0.1:6061\)

In the SPA phone, you need to log into the Administration interface and navigate to the SIP tab. At the very bottom of the page you will find the section called Linksys Key System Parameters. You need to set the following parameters:

* Linksys Key System: Yes
* Multicast Address: 239.0.0.1:6061

Note that the multicast address you assign to the phone is the one that comes second in the channel definition \(in our example, the one using port 6061\).

Note that you can write the Page\(\) command in this format in an environment where there is a mix of SPA \(f.k.a. Linksys, now Cisco\) phones and other types of phones. The other phones will use the first address and will work the same as if you had used basic instead of linksys.

#### SIP-based paging adapters

There are many SIP-based paging speakers on the market. These devices are addressed in the dialplan in the exact same way as a SIP ATA connected to a UTI1 \(in other words, to the system they’re just a telephone set\), but physically they are similar to external paging speakers. Since they auto-answer, there is often no need to pass them any extra information, the way you would need to with a SIP telephone set.

For smaller installations \(where no more than perhaps a half-dozen speakers are required\), these devices might be cost-effective because no other hardware is required. However, for anything larger than that \(or for installation in a complex environment such as a warehouse or parking lot\), you will get better performance at far less cost with a traditional analog paging system connected to the phone system by an analog \(FXS\) interface.

We haven’t had any experience with these types of devices, but it is hoped that they would support multicast as standard. Keep this in mind if you are planning to use a large number of them. It’s usually best to order one, test it out in a prototypical configuration, and then only commit to a quantity once you’ve verified that it does what you need.

#### Combination paging

In many organizations, there may be a need for both set-based and external paging. As an example, a manufacturing facility might want to use set-based paging for the office area but overhead paging for the plant and warehouse. From Asterisk’s perspective, this is fairly simple to accomplish. When you call the Page\(\) application, you simply specify the various resources you want to page, separated by the & character, and they will all be included in the conference that the Page\(\) application creates.

#### Bringing it all together

At this point you should have a list of the various channel types that you want to page. Since Page\(\) will nearly always want to signal more than one channel, we recommend setting a global variable in the \[globals\] section of your extensions.conf file that defines the list of channels to include, and then calling the Page\(\) application with that string:

```text
[globals]
MULTICAST=MulticastRTP/linksys/239.0.0.1:1234
;MULTICAST=MulticastRTP/linksys/239.0.0.1:1234/239.0.0.1:6061 ; if you have SPA phones
BOGEN=PJSIP/ATAforPaging ; Assumes an ATA named [ATAforPaging]
PAGELIST=${MULTICAST}&${BOGEN} ; Variable names are arbitrary.
;...
[sets]
; ...
exten => *731,1,Page(${PAGELIST},i,120)
```

This example offers several possible configurations, depending on the hardware. While it is not strictly required to have a PAGELIST variable defined, we have found that this will tend to simplify the management of multiple paging resources, especially during the configuration and testing process.

### Zone Paging

Zone paging is popular in places such as automobile dealerships, where the parts department, the sales department, and perhaps the used car department all require paging, but do not want to hear each other’s pages.

In zone paging, the person sending the page needs to select which zone to page into. A zone paging controller such as a Bogen PCM2000 is generally used to allow signaling of the different zones: the Page\(\) application signals the zone controller, the zone controller answers, and then an additional digit is sent to select to which zone the page is to be sent. Most zone controllers will allow for a page to all zones, in addition to combining zones \(for example, a page to both the new- and used-car sales departments\).

You could also have separate extensions in the dialplan going to separate ATAs \(or groups of telephones\), but this may prove more complicated and expensive than simply purchasing a paging controller that is designed to handle this. Zone paging doesn’t require any significantly different technology, but it does require a little more thought and planning with respect to both the dialplan and the hardware.

And that’s parking and paging. It’s a ton of information to digest, but once you get the hang of it, it’s quite logical.

## Advanced Conferencing

The ConfBridge\(\) application is an enhanced conferencing application in Asterisk that delivers high-definition audio and basic video conferencing. We previously introduced a basic working setup for ConfBridge\(\). If you’ve been building out your dialplan as you read along, you’ll find a basic conference bridge in your extensions.conf file that looks something like this:

exten =&gt; 221,1,NoOp\(\)

 same =&gt; n,ConfBridge\(${EXTEN}\)

In a traditional Asterisk configuration, there would be a confbridge.conf file where we could configure parameters to apply to various scenarios. That is still possible, but it no longer makes much sense to do it that way. So, we’re going to skip right over the whole configuration file, except to say that the sample file \(found at ~/src/asterisk-15.&lt;TAB&gt;/configs/samples/confbridge.conf.sample\) now becomes an excellent reference document, but no more than that. Read on and this should start to make sense.

First up, we need to explain that there are three types of items that can be configured for a conference, namely bridge, menu, and user.

The bridge type defines the conference rooms themselves, the menu type defines menus that can be accessed from the conferences, and the user type allows different participants in the conference to have specific configuration applied to them. For example, a large conference call might have a speaker \(who will do most of the talking\), an administrator \(to assist the speaker\), and dozens of participants \(who might not be allowed to speak\).

Let’s lay down a subroutine to get us started:

```text
[subConference]
exten => _[0-9].,1,Noop(Creating conference room for ${EXTEN})
 same => n,Goto(${ARG1})
 same => n,Noop(INVALID ARGUMENT ARG1: ${ARG1})
 same => n(admin),Noop()
 same => n,Authenticate(${ARG2}) ; Could also use ,Set(CONFBRIDGE(user,pin)=${ARG2})
 same => n,Set(ConfNum=$[${EXTEN} - 1]) ; Hack: Subtract 1 to get the conf number
 same => n,Set(CONFBRIDGE(bridge,record_conference)=yes) ; Record when admin arrives
 same => n,Set(RecordingFileName=${ConfNum}-${STRFTIME(,,%Y-%m-%d %H:%m:%S)})
 same => n,Set(CONFBRIDGE(bridge,record_file)=${RecordingFileName}) ; unique name
 same => n,Set(CONFBRIDGE(user,admin)=yes) ; Admin
 same => n,Set(CONFBRIDGE(user,marked)=yes) ; Mark this user
 same => n,Set(CONFBRIDGE(menu,7)=decrease_talking_volume) ; Decrease gain
 same => n,Set(CONFBRIDGE(menu,9)=increase_talking_volume) ; Increase gain
 same => n,Set(CONFBRIDGE(menu,4)=set_as_single_video_src) ; Lock video on me
 same => n,Set(CONFBRIDGE(menu,5)=release_as_single_video_src) ; Return to talker
 same => n,Set(CONFBRIDGE(menu,6)=admin_toggle_mute_participants); Mute all but admins
 same => n,Set(CONFBRIDGE(menu,2)=participant_count) ; How many participants?
 same => n,ConfBridge(${ConfNum})
 same => n,Return()
 same => n(participant),Noop()
 same => n,Set(ConfNum=${EXTEN})
 same => n,Set(CONFBRIDGE(user,wait_marked)=yes) ; Wait for a marked user
 same => n,Set(CONFBRIDGE(user,announce_only_user)=no) ; Wait for a marked user
 same => n,Set(CONFBRIDGE(user,music_on_hold_when_empty)=yes) ; Wait for a marked user
 same => n,Set(CONFBRIDGE(menu,7)=decrease_talking_volume) ; Decrease gain
 same => n,Set(CONFBRIDGE(menu,9)=increase_talking_volume) ; Increase gain
 same => n,ConfBridge(${ConfNum})
 same => n,Return()
```

We can set bridge, user, and menu parameters as in the preceding example. All of the parameters you might wish to use are documented in the ~/src/asterisk-15.&lt;TAB&gt;/configs/samples/confbridge.conf.sample file.

When we call the subroutine, we can pass the user as an argument. Place the following new code in your \[sets\] context after \_55512XX and before \*724:

```text
exten => _55512XX,1,Answer()
 same => n,Playback(tt-monkeys)
; ConfBridge
exten => *600,1,GoSub(subConference,${EXTEN:1},1(participant)) ;
exten => *601,1,GoSub(subConference,${EXTEN:1},1(admin,4242)) ;
exten => *724,1,Noop(Page)
 same => n,Set(ChannelsToPage=${UserA_DeskPhone}&${UserA_SoftPhone}&${UserB_DeskPhone})
 same => n,Page(${ChannelsToPage},i,120)
```

If you dial \*600, you will be joined as a participant. If you dial \*601, you will be asked for the PIN \(4242\), and will join as an administrator. We used dialplan labels to control the call flow into the subroutine. It’s easy to read, and easy to administer.

In [Chapter 15](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22asterisk-DB) we’ll explore how to use an external database to store and retrieve these parameters, rather than hardcoding them in the dialplan.

### Video Conferencing

The conference engine in Asterisk can handle video, but it is a very simplified offering, and you should evaluate it carefully to ensure it meets your needs. Some of the more serious limitations include:

* All video participants must be using the same video codec; no video transcoding is available in Asterisk.
* There is no video multiplexing in Asterisk; only one video source can be shown at a time to a participant.

For a user to be able to use video \(whether in a conference, or just for normal calls\), they need to have the video codecs enabled. This can be done by modifying the allow field in the asterisk.endpoints table, and adding 'h264,vp8' to the allow field. Make sure you don’t remove the codecs that are already there \(for example, the ulaw audio codec\). A functional entry in that field might look like:

ulaw,h264,vp8

Before attempting to use video with your conferences, make sure your sets are able to use it with direct desk-to-desk calls. If you can use video conferencing between your sets, it’s likely it’ll also work in your conference rooms.

In [Chapter 20](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch20.html%22%20/l%20%22webrtc_ch), we’ll dive into WebRTC, which is where we’ll explore more powerful concepts in the delivery of multimedia communication, including conferencing.

## Conclusion

In this chapter we explored the features.conf file, which contains the functionality for enabling DTMF-based transfers, enabling the recording of calls during a call, and configuring parking lots for one or more companies. We also looked at various ways of announcing calls and information to people in the office using a multitude of paging methods, including traditional overhead paging systems and multicast paging to the phone sets on employees’ desks. After that we delved into the ConfBridge\(\) application, which is extremely flexible in configuration and rich in available features. This exploration of the various methods of implementing the traditional parking, paging, and conferencing features in a modern way hopefully shows you the flexibility Asterisk can offer.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406491144-marker) Yes, we realize that a SIP INFO message is in fact a SIP message and not technically part of the audio channel, but the point is that you can’t use the “transfer” or “park” button on your SIP phone to access these features while on a call. You’ll have to send DTMF.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406451032-marker) There is some flexibility in the syntax \(you can look at the sample file for details\), but our example uses the style we recommend, since it’s the most consistent with typical dialplan syntax.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406440184-marker) We’ll cover AGI in [Chapter 18](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch18.html%22%20/l%20%22AGI), but briefly, AGI scripts are external programs you can trigger from the dialplan. Handy? Very!

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406331512-marker) We hope you realize that the actual extension will be related to the channel name that parked the call, and will not be SIP\_0004F2040808 \(unless Leif sells you the Polycom phone from his lab\).

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406290408-marker) The Bogen UTI1 is useful because it can handle all manner of incoming and outgoing connections, which pretty nearly guarantees that you’ll be able to painlessly connect your telephone system to any sort of external paging equipment, no matter how old or obscure. The cost of the unit can be offset against the hours of time saved by having a purpose-built, fully featured, Swiss-army-knife-like interface to an existing public address system \(or, for that matter, as part of a new PA system\).

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406289064-marker) In this book we’re assuming that the external paging equipment is already installed and was working with the old phone system, but there’s nothing stopping you from installing a brand-new public address system and connecting it to your Asterisk system. You might feel that we’re plugging Bogen a bit here, but it’s simply because they have been doing telephone system paging for a very long time. We’ve been using them for nearly 30 years, and they’ve been doing it for longer, so as long as you’re comfortable putting all the pieces together, you can get the job done right the first time.

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406286520-marker) If you’re at all curious about this, we want to encourage you to try this out in your lab. It might prove to work very well for you, and it does potentially save some hardware costs. We’ve just found that hardware is cheaper than labor, so we’d rather drop a couple of hundred bucks on known-good gear than have some poor technician mucking about onsite for eight hours with an upset customer demanding to know when the paging problem will be solved.

[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406263240-marker) Hint: the local channel will be your friend here.

[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406256840-marker) It even has its own Class D reserved IP address space, from 224.0.0.0 to 239.255.255.255 \(but read up on IP multicast before you just grab one of these and assign it\). Parts of this address space are private, parts are public, and parts are designated for purposes other than what you might want to use them for. For information about multicast addressing, see [its Wikipedia page](https://en.wikipedia.org/wiki/Multicast_address).

[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22idm46178406245224-marker) Very loud, and no way to adjust gain.

