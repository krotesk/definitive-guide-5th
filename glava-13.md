---
description: Состояния устройств
---

# Глава 13

Out of clutter, find simplicity.

Albert Einstein

It is often useful to be able to determine the state of the devices that are attached to a telephone system. For example, a receptionist might require the ability to see the statuses of everyone in the office in order to determine whether somebody can take a phone call. Asterisk itself needs this same information. As another example, if you were building a call queue, as discussed in [Chapter 12](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22asterisk-ACD), Asterisk needs to know when an agent is available so that another call can be delivered. This chapter discusses device state concepts in Asterisk, as well as how devices and applications use and access this information.

## Device States

There are two categories of devices that Asterisk provides state information for: channel devices \(such as PJSIP endpoints\) and virtual devices \(which are built-in services that one might wish to monitor, such as conference rooms\).

To reference the state of a channel, you do so in exactly the same way you would with Dial\(\), for example DEVICE\_STATE\(PJSIP/000f300B0B02\), whereas to reference the state of a virtual device, the format is virtual device type:identifier, for example DEVICE\_STATE\(ConfBridge:1234\).

Virtual devices include things that are inside Asterisk but provide useful state information \(see [Table 13-1](13.%20Device%20States%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22table-virtualDevices)\).

Table 13-1. Devices for which Asterisk can provide state information

| Device | Description |
| :--- | :--- |
| PJSIP/channel name | Many channels can have their state monitored, but the PJSIP channel offers by far the most amount of useful data; thus, monitoring SIP devices is the most common use of DEVICE\_STATE. |
| ConfBridge:conference bridge | The state of a MeetMe conference bridge. The state will reflect whether or not the conference bridge currently has participants called in. More information on using MeetMe\(\) for call conferencing can be found in [Chapter 11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22asterisk-SysAdmin). |
| Custom:custom name | Custom device states. These states have custom names and are modified using the DEVICE\_STATE\(\) function. Example usage can be found in [“Using Custom Device States”](13.%20Device%20States%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22usingCustomDeviceStates). |
| Park:exten@context | The state of a spot in a call parking lot. The state information will reflect whether or not a caller is currently parked at that extension. More information about call parking in Asterisk can be found in [“Call Parking”](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22parkingLots). |
| Calendar:calendar name | Calendar state. Asterisk will use the contents of the named calendar to set the state to available or busy. |

### Checking Device States

The DEVICE\_STATE\(\) dialplan function reads the current state of a device.

exten =&gt; 7012,1,Answer\(\)

 same =&gt; n,Set\(DeviceIdent=PJSIP/000f300B0B02\)

 same =&gt; n,Verbose\(3,${DeviceIdent} is ${DEVICE\_STATE\($DeviceIdent}\)}\)

 same =&gt; n,Hangup\(\)

If we call extension 7012 from the same device that we are checking the state of, the following verbose message comes up on the Asterisk console:

 -- PJSIP/000f300B0B02 is INUSE

#### Note

[Chapter 17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch17.html%22%20/l%20%22asterisk-AMI) discusses the Asterisk Manager Interface \(AMI\). The GetVar manager action can be used to retrieve device state values in an external program. You can use it to get the value of a normal variable, or to return a value from a dialplan function such as DEVICE\_STATE\(\).

Here is a list of the values that the DEVICE\_STATE\(\) function will return \(depending, of course, on what was found\):

* UNKNOWN
* NOT\_INUSE
* INUSE
* BUSY
* INVALID
* UNAVAILABLE
* RINGING
* RINGINUSE
* ONHOLD

This information can then be used in the dialplan for call-flow decisions \(for example, a local channel ringing an agent might use this information to determine that an agent phone is on a call on another line, and thus reject the call so it passes back into the queue\).

## Extension States Using the hint Directive

Extension state is a dialplan mechanism Asterisk uses to allow SIP devices to subscribe to presence information. As an example, a reception phone might have a Busy Lamp Field \(BLF\) module, containing buttons to be used to show the state of various phones in the office. The phone with the BLF will send subscription requests in order to tell Asterisk which devices it wants to receive presence information from. In the dialplan, we use the hint directive to define the mapping between an extension and one or more devices.

### Hints

To define a hint in the dialplan, the keyword hint is used in place of a priority.

\[hints\]

;exten = &lt;extension&gt;,hint,&lt;device state id&gt;\[& &lt;more dev state id\],&lt;presence state id&gt;

exten =&gt; 100,hint,${UserA\_DeskPhone}

exten =&gt; 221,hint,ConfBridge:221

Often, you might see hints defined in the same section of the dialplan as the normal extension. This can get a bit visually cluttered, and it also suggests that the hint is somehow associated with the dialable extension, which is not the case.

\[sets\]

exten =&gt; 100,hint,${UserA\_DeskPhone}

exten =&gt; 100,1,Gosub\(subDialUser,${EXTEN},1\(${UserA\_DeskPhone},${EXTEN},default,22\)\)

exten =&gt; 101,hint,${UserA\_SoftPhone}

exten =&gt; 101,1,Gosub\(subDialUser,${EXTEN},1\(${UserA\_SoftPhone},${EXTEN},default,23\)\)

exten =&gt; 102,hint,${UserB\_DeskPhone}

exten =&gt; 102,1,Gosub\(subDialUser,${EXTEN},1\(${UserB\_DeskPhone},${EXTEN},default,26\)\)

exten =&gt; 103,hint,${UserB\_SoftPhone}

exten =&gt; 103,1,Gosub\(subDialUser,${EXTEN},1\(${UserB\_SoftPhone},${EXTEN},default,24\)\)

exten =&gt; 110,1,Dial\(${UserA\_DeskPhone}&${UserA\_SoftPhone}&${UserB\_SoftPhone}\)

In our example we’ve made a direct correlation between the hint’s extension number and the extension number being dialed, although there is no requirement that this be the case.

### Checking Extension States

The easiest way to check the current state of the hint extensions is through the Asterisk CLI. The core show hints command will display all currently configured hints:

\*CLI&gt; core show hints

 -= Registered Asterisk Dial Plan Hints =-

100@hints : PJSIP/0000f30A0A01 State:Unavailable Presence:not\_set Watchers 0

101@hints : PJSIP/SOFTPHONE\_A State:Unavailable Presence:not\_set Watchers 0

102@hints : PJSIP/0000f30B0B02 State:Unavailable Presence:not\_set Watchers 0

103@hints : PJSIP/SOFTPHONE\_B State:Unavailable Presence:not\_set Watchers 0

110@hints : PJSIP/0000f30A0A01&P State:Unavailable Presence:not\_set Watchers 0

221@hints : ConfBridge:221 State:Unavailable Presence:not\_set Watchers 0

----------------

- 6 hints registered

In addition to showing you the state of each hint, the output of core show hints also provides a count of watchers. A watcher is an entity that has subscribed to receive updates on the state of this extension. If a SIP endpoint subscribes to the state of an extension, the watcher count will be increased.

Extension state can also be retrieved with a dialplan function, EXTENSION\_STATE\(\). This function operates much like the DEVICE\_STATE\(\) function described in the preceding section. Add the following example to your /etc/asterisk/extensions.conf file, as a new extension right after 235:

exten =&gt; 234,1,NoOp\(\)

 same =&gt; n,Set\(FEATURE\(parkingtime\)=60\)

exten =&gt; 235,1,Noop\(The state of 100@hints is ${EXTENSION\_STATE\(100@hints\)} \)

 same =&gt; n,Hangup\(\)

exten =&gt; 321,1,NoOp\(\)

When this extension is called from the endpoint assigned to 100, this is the message that shows up on the Asterisk console:

 -- The state of 100@hints is INUSE

The following list includes the possible values that may be returned from the EXTENSION\_STATE\(\) function:

* UNKNOWN
* NOT\_INUSE
* INUSE
* BUSY
* UNAVAILABLE
* RINGING
* RINGINUSE
* HOLDINUSE
* ONHOLD

## SIP Presence

Asterisk gives devices the capability to subscribe to extension state using the SIP protocol. This functionality is often referred to as BLF \(Busy Lamp Field\); see [Figure 13-1](13.%20Device%20States%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig1301).

![](.gitbook/assets/0%20%282%29.png)

#### Figure 13-1. Busy Lamp Field aka sidecar

The configuration of the module will be slightly \(or very\) different for each manufacturer; however, the subscription information will—one way or another—need to include the following:

* The address of the Asterisk server \(this might be defined on a per-button basis, or it might apply to the whole phone\).
* The context to subscribe to \(in our sample dialplan, it’s named \[hints\]\). This setting is defined in the subscribe\_context field of the asterisk.ps\_endpoints table.
* The relevant extension \(100, 101, 102, etc.\)[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405538488)

One of the more simple and inexpensive ways we’ve found for testing presence is using the open source Windows softphone, MicroSIP.[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405536680) You’ll first need to download MicroSIP and get it registered to your Asterisk system. Then, under the contacts tab of the softphone, you can right-click in the open area to Add a contact. In the Name section you can put whatever you wish, but under the Number section, you will input extension@hints context, which in our case would be one of 100@hints, 101@hints, 102@hints, or 103@hints. If you’ve set everything up in Asterisk per the previous examples, you should see the state of your subscriptions change in response to whatever the far end set is doing. You can also monitor this from Asterisk’s perspective using a command such as:

$ watch -n 0.5 "sudo asterisk -rx 'core show hints'"

The configuration of presence on physical desk telephones is essentially the same, but it can be more difficult to make sense of the specific syntax each manufacturer requires. Our advice is to get it working with MicroSIP \(which you should be able to run on WINE under Linux or macOS\). It’s an easy setup, and from there you’ll have a known-good configuration you can trust when you’re sorting out a similar config for one of your desk phones.

## Using Custom Device States

In addition to the devices Asterisk knows internally how to monitor \(PJSIP, ConfBridge, Park, Calendar\), Asterisk also provides the ability to create custom device states, which can be very useful in the development of some interesting applications.

Custom device states are defined using a prefix of Custom:. The text that comes after the prefix can be anything you want. To set or read the value of a custom device state, use the DEVICE\_STATE\(\) dialplan function. Put this into your extensions.conf right after extension 235:

exten =&gt; 235,1,Noop\(The state of 100@hints is ${EXTENSION\_STATE\(100@hints\)} \)

 same =&gt; n,Hangup\(\)

exten =&gt; 236,1,Noop\(Set a custom status\)

 same =&gt; n\(blink\),Set\(DEVICE\_STATE\(Custom:rudolph\)=UNAVAILABLE\)

 same =&gt; n,Set\(DEVICE\_STATE\(Custom:santa\)=NOT\_INUSE\)

 same =&gt; n,Wait\(0.75\)

 same =&gt; n,Set\(DEVICE\_STATE\(Custom:rudolph\)=NOT\_INUSE\)

 same =&gt; n,Set\(DEVICE\_STATE\(Custom:santa\)=UNAVAILABLE\)

 same =&gt; n,Wait\(0.75\)

 same =&gt; n,Goto\(blink\)

Then add this to your \[hints\] context:

exten =&gt; 221,hint,ConfBridge:221

exten =&gt; santa,hint,Custom:santa

exten =&gt; rudolph,hint,Custom:rudolph

Festive, yeah?

#### Note

You will notice that when you hang up, one of the custom device states will remain “Unavailable.” This is an important point: there is nothing in the system that will update your custom device states, unless you yourself have implemented something to do that.

## Conclusion

The device states functionality in Asterisk can be used to track the state of various resources and deliver information about those states to various subscribers. Commonly \(and traditionally\) used for Busy Lamp Fields, the Custom device state allows this resource to be far more flexible than it would be in a traditional PBX.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405538488-marker) Items 2 and 3 may be formed as a single string, looking like 100@hints, or something similar.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405536680-marker) Which is written using the same PJSIP library that Asterisk uses.

