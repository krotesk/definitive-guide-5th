---
description: ARI
---

# Глава 19

## Chapter 19. Asterisk REST Interface

People who think they know everything are a great annoyance to those of us who do.

Isaac Asimov

The Asterisk REST Interface \(ARI\) was created to address the limitations inherent in developing external or enhanced functionality outside Asterisk. While AGI allows you to trigger external applications, and AMI allows you to externally supervise and control calls in progress, any attempt to integrate both into a complete external application quickly becomes complex and kludgy. ARI allows developers to build a stand-alone and complete application, using Asterisk as the underlying engine.

As of this writing, ARI requires a very basic dialplan in order to trigger the Stasis\(\) dialplan application, which then hands the channel over to ARI. By the time you read this, it’s very likely that this requirement has changed, as the Asterisk developer community has actively been working on allowing ARI to spawn without any dialplan in the middle.

Using an external interface such as ARI to control Asterisk is not necessarily going to make your life easier. The skills required to implement and troubleshoot applications of this type require a comprehensive skill set, in not only your language of choice, but also in Linux system administration, Asterisk administration, network troubleshooting, and fundamental telephony concepts. For the skilled developer, ARI can give you the power you want in your applications, but for someone learning, we recommend you consider mastering the dialplan before you dive into external development environments. The dialplan is peculiar, but it is also completely integrated, high-performance, and relatively easy to learn.

Having said that, let’s get you up and running with ARI.

## ARI Quick Start

This section gives you a simple working example of ARI. Later in the chapter we’ll cover things in more detail.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178403407976)

#### Warning

In this quick-start section we will be using a very simple HTTP access layer. You must be very careful about putting this sort of configuration into production. If, for example, you are going to run your application on a separate machine and connect it to Asterisk across a socket, you will want a more secure connection. What we’re doing in this section is akin to a sailing club using dinghies to teach; useful as an introduction, but foolish and dangerous to set out to sea in such a craft.

### Basic Asterisk Configuration

You should already have the Asterisk web server running, so you simply need to verify that your /etc/asterisk/http.conf file looks similar to the following:

\[general\]

enabled = yes

bindaddr = 127.0.0.1

Next, a simple /etc/asterisk/ari.conf file is needed:

\[general\]

enabled = yes

pretty = yes

\[asterisk\]

type = user

read\_only = no

password = whateveryoudodontusethispassword

OK, let’s load the ari module now:

$ sudo asterisk -rx 'module load res\_ari.so'

Loaded res\_ari.so =&gt; \(Asterisk RESTful Interface\)

Then, into our /etc/asterisk/extensions.conf file we need an extension to trigger the Stasis\(\) dialplan app:[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396697256)

exten =&gt; 242,1,Noop\(\)

 same =&gt; n,Stasis\(zarniwoop\)

 same =&gt; n,Hangup\(\)

Reload your dialplan with

$ sudo asterisk -rx 'dialplan reload'

Dialplan reloaded.

At this point it might be worthwhile to simply reload Asterisk:

$ sudo service asterisk restart

There are just a few steps left, and you’re ready to test your ARI environment.

### Testing Your Basic ARI Environment

Since ARI depends on WebSockets, we’ll need a tool to allow us to test from the command line. The Node.js package manager \(npm\) will allow us to find and install the wscat tool we’ll use for our tests.

$ sudo yum -y install npm

$ sudo npm install -g wscat

/usr/bin/wscat -&gt; /usr/lib/node\_modules/wscat/bin/wscat

/usr/lib

+-- wscat@2.2.1

 +-- commander@2.15.1

 +-- read@1.0.7

 ¦ +-- mute-stream@0.0.8

 +-- ws@5.2.2

 +-- async-limiter@1.0.0

Now let’s light it up and see what we get!

$ wscat -c "ws://localhost:8088/ari/events?api\_key= \

 asterisk:whateveryoudodontusethispassword&app=zarniwoop"

connected \(press CTRL+C to quit\)

&gt;

So far, so good. Let’s place a call to our Stasis\(\) app and see what happens.

Open up a new SSH window \(leave the other one as is so you can see what happens in the wscat session\). Connect to the Asterisk CLI in that new shell session:

$ sudo asterisk -rvvvv

Using one of your lab telephones, place a call to 242.

On the Asterisk CLI, you should see this:

\*CLI&gt;

 == Setting global variable 'SIPDOMAIN' to '172.29.1.57'

 -- Executing \[242@sets:1\] NoOp\("PJSIP/SOFTPHONE\_A-00000001", ""\) in new stack

 -- Executing \[242@sets:2\] Stasis\("PJSIP/SOFTPHONE\_A-00000001", "zarniwoop"\) in new stack

And on the wscat session, you should see this:

&gt;

&lt; {

 "type": "StasisStart",

 "timestamp": "2019-01-27T21:43:43.720-0500",

 "args": \[\],

 "channel": {

 "id": "1548643423.2",

 "name": "PJSIP/SOFTPHONE\_A-00000002",

 "state": "Ring",

 "caller": {

 "name": "101",

 "number": "SOFTPHONE\_A"

 },

 "connected": {

 "name": "",

 "number": ""

 },

 "accountcode": "",

 "dialplan": {

 "context": "sets",

 "exten": "242",

 "priority": 2

 },

 "creationtime": "2019-01-27T21:43:43.709-0500",

 "language": "en"

 },

 "asterisk\_id": "08:00:27:27:bf:0e",

 "application": "zarniwoop"

}

&gt;

OK, now we’re going to open yet another shell session[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396682696) so we can interact with this connection we’ve created. From this new shell, issue the following command:

$ curl -v -u asterisk:whateveryoudodontusethispassword -X POST \

 "http://localhost:8088/ari/channels/1548643423.2/play?media=sound:believe-its-free" sd

Note the "id" from the JSON returned on the wscat session must be used following the 'channels/' portion of the curl command. In other words, you must match the channel identifier in your command to the channel identifier associated with your call. In this manner, you can of course wrangle many calls simultaneously.

### Working with Your ARI Environment Using Swagger

Asterisk’s ARI has been developed to be compatible with the OpenAPI Specification \(aka Swagger\), which means that many tools compatible with this spec will work with ARI. As an example, you can interact with your ARI installation using Swagger-UI, which will be useful both for debugging and as a documentation source.

First up, we’ll need to expose our Asterisk HTTP server to the local network \(currently it’s only allowing connections from 127.0.0.1\). In your /etc/asterisk/http.conf file you’ll bind the HTTP server to the local IP address of your Asterisk machine:

$ sudo vim /etc/asterisk/http.conf

; Enable the built-in HTTP server, and only listen for connections on localhost.

\[general\]

enabled = yes

;bindaddr = 127.0.0.1 ; comment this out

bindaddr = 172.29.1.57 ; LAN IP OF YOUR ASTERISK SERVER

Next, we’ll need to add a line to your /etc/asterisk/ari.conf file:

$ sudo vim /etc/asterisk/ari.conf

\[general\]

enabled = yes

pretty = yes

allowed\_origins=http://ari.asterisk.org

...

Save and reload the http and ari modules in Asterisk:

$ sudo asterisk -rx 'module reload http' ; sudo asterisk -rx 'module reload ari'

Now, from your development desktop, open up your browser and navigate to [http://ari.asterisk.org](http://ari.asterisk.org/).

You’ll see a web page similar to [Figure 19-1](19.%20Asterisk%20REST%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig1901).

![](.gitbook/assets/0%20%285%29.png)

#### Figure 19-1. Swagger UI for ARI

Replace localhost with the LAN IP address of your Asterisk server, and in the api\_key field, put your ARI user:password from /etc/asterisk/ari.conf \(for example, asterisk:whateveryoudodontusethispassword\). If you’ve got all the configuration correct, you will be rewarded with the results in [Figure 19-2](19.%20Asterisk%20REST%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig1902).

![](.gitbook/assets/1%20%285%29.png)

#### Figure 19-2. ARI Swagger

You are looking at comprehensive documentation for your ARI module, and you can actually pass queries to it as well. This is a massively useful debugging aid, and kudos to the Digium folks for it.

As an example of what this is good for, select the endpoints:Endpoint resources item, press the GET button beside /endpoints, and you will see the screen shown in [Figure 19-3](19.%20Asterisk%20REST%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig1903).

![](.gitbook/assets/2%20%284%29.png)

#### Figure 19-3. Get endpoints

Well, go ahead—press the “Try it out!” button.

Note the "id" of the channel in the wscat session, which you’ll want to copy for use in the Swagger UI \(you’ll see several lines of JSON output relating to the call\).

Perform the following actions on the channel using the Swagger UI interface: POST: Answer \(answer the channel\), POST: hold \(place the call on hold\), DELETE: hold \(take the call off hold\). Note what happens to the channel in each case.

Use of this Swagger UI is also documented over at the [Asterisk wiki](https://wiki.asterisk.org/wiki/display/AST/Using+Swagger+to+Drive+ARI).

This will greatly simplify your development and testing process.

OK, that’s the quick start. Let’s dive in deeper to ARI.

## The Building Blocks of ARI

There are three components that work together to deliver ARI:

* The RESTful interface, through which the external application communicates with Asterisk.
* A WebSocket that passes information back to the external application from Asterisk \(in JSON format\).
* The Stasis\(\) dialplan application, which connects control of a channel to the external application.

### REST

The term RESTful stems from Representational State Transfer \(REST\), which is an architectural model for web services \(as opposed to, say, a protocol\). The term RESTful has commonly come to refer to any API that provides interaction through URLs, with data represented in JSON format.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396640584) So, anything that is “RESTful” is supposed to adhere to the constraints of REST, but in practice may be implemented as a looser interpretation \(which, if it gets the job done, may indeed be good enough\).

### WebSocket

The WebSocket connection is the mechanism that performs the communication between the internals of Asterisk and the RESTful interface. In Asterisk, events may happen that the client did not initiate, and the WebSocket allows Asterisk to signal those changes to the client.

Asterisk’s built-in HTTP server potentially provides other services across a web interface. For example, WebRTC also connects through the web server. If you are making changes or adding new services, make sure you not only test the item you’re working on, but also other services running through the same server, to ensure you haven’t inadvertently misconfigured something else.

### Stasis

The Stasis Message Bus allows the core of Asterisk to communicate events with other modules and components. It is mostly internal to Asterisk; however, in the case of ARI, a dialplan application named Stasis\(\) allows the dialplan to pass call control to your external ARI application.

The Stasis\(\) application itself is required in order to signal to the dialplan that call control is to be passed to the external program via ARI.

As of Asterisk 16, it is no longer necessary to write dialplan code to define a connection from an incoming channel to your ARI client application. Many developers in the Asterisk community write all their call control logic in external applications, and having to code up a few lines of dialplan just to pass channels to their app was seen as kludgy and confusing. They requested \(and developed\) a mechanism whereby Asterisk will create automatic dialplan to handle this function.

When the API is instantiated, the application reference in the URL—for example, our app zarniwoop—will trigger the automatic creation of a dialplan context named according to the app name \(in this case, \[stasis-zarniwoop\]\), including an extension that pattern matches everything. This extension will then pass all calls arriving in that context to Stasis\(zarniwoop\). You will need to associate your channels with the correct context \(context=stasis-zarniwoop\) in your PJSIP \(or other channel\) configuration tables, at which point calls to those channels will automatically be connected through Stasis\(\) to the client application.

If all this seems confusing, there’s no reason you need to stop using actual dialplan to handle this, as we did earlier in our quick-start example.

Understanding the workings of Stasis\(\) is generally not necessary unless you are going to be developing the Asterisk product itself \(i.e., joining the Asterisk development team and coding new capabilities into Asterisk\).

Typically, after your initial experimentation with ARI, you will want to implement a framework to help ease the work of developing your external application.

## Frameworks

A production-grade application using ARI will benefit from the implementation of a framework to simplify development effort, add a layer of security, and provide a control environment.

There are several such libraries available. Which one you choose will in part be dictated by which language you prefer to use, and should also take into account whether the framework you’re interested in has an active community and is still being actively maintained.

The ones described next are listed in the Asterisk wiki. We examined the code repository for each, and while some projects are still actively maintained, others have not been updated in quite some time. If you are planning to implement one of these frameworks, you will need to do your own due diligence to ensure you can get support for it. In many cases, it may be worthwhile to reach out to the developers, and determine their consulting rates so you can ensure priority access to their time should you need it.

### ari-py \(and aioari\) for Python

The [ari-py framework](https://github.com/asterisk/ari-py) was written by Digium in 2013–2014, and as of this writing had not been updated since then. This framework builds on Asterisk’s Swagger.py client.

Shortly after the relase of ari-py, it was forked into the [aioari project](https://pypi.org/project/aioari/), which delivers an asynchronous version of ari-py. This code has been more steadily updated since then \(although as of this writing had not been updated since early 2018\). This framework should be included in your evaluation of a Python framework for ARI.

If you are looking to develop ARI applications in Python, one of these two frameworks may be what you are looking for. If you are looking to build a large ARI application, you will need to ensure that you have carefully tested the performance implications of using Python for what you are doing.

Digium has provided samples for this framework \(and others\) at [https://github.com/asterisk/ari-examples](https://github.com/asterisk/ari-examples).

### node-ari-client

For the JavaScript folks, there is a Node.js-based ARI framework that was first released in early 2014, and as of this writing is still being updated. It is based on the automatically generated API that comes from swagger-js.

For JavaScript/Node developers, this is where you’ll want to start: [https://github.com/asterisk/node-ari-client](https://github.com/asterisk/node-ari-client).

Digium has provided samples for this framework \(and others\) at [https://github.com/asterisk/ari-examples](https://github.com/asterisk/ari-examples).

### AsterNET.ARI

The Windows folks are not left out. The AsterNET.ARI project delivers a framework for .NET that augments the AsterNET project \(which also includes integration with Asterisk’s FastAGI and AMI interfaces\).

You can find the repository for AsterNET.ARI here: [https://github.com/skrusty/AsterNET.ARI](https://github.com/skrusty/AsterNET.ARI).

Digium has provided samples for this framework \(and others\) at [https://github.com/asterisk/ari-examples](https://github.com/asterisk/ari-examples).

### ari4java

The ari4java project is one of the most actively developed ARI frameworks we have found. It has been developed since 2013, and the repository was receiving commits at the same time as this writing.

If Java is your language, you will want to check out the ari4java repository at [https://github.com/l3nz/ari4java](https://github.com/l3nz/ari4java).

### phpari

The phpari project delivers an ARI framework for the PHP community. It has been developed since 2014, and the repository was still being updated as of this writing.

For the PHP fans, you’ll find the repository at [https://github.com/greenfieldtech-nirs/phpari](https://github.com/greenfieldtech-nirs/phpari).

### aricpp

If you’re used to writing in C++, there’s even an ARI project for you. The aricpp framework consists of header files only, so you can build its functions right into whatever you’re developing. This library has also been performance tested with SIPp, and while we don’t have any numbers on that, it seems to us that a compiled framework that has been performance tested is very much worth taking for a spin if you have the right skills.

One of the newer of the ARI frameworks, this project benefits from regular updates. Check it out at [https://github.com/daniele77/aricpp](https://github.com/daniele77/aricpp).

### asterisk-ari-client

Yes, Ruby also has an ARI framework.

You can find it at [https://github.com/svoboda-jan/asterisk-ari](https://github.com/svoboda-jan/asterisk-ari).

## Conclusion

ARI provides a current-generation RESTful API that can be used to develop communications applications using popular development languages. Through it, an experienced developer can harness the power of the most successful PBX platform in history. This allows next-generation communications applications to interact with legacy telecommunications protocols and applications, which could prove very useful as we are increasingly called to bridge the gap between past, present, and future communications technologies.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178403407976-marker) Although, to be honest, there’s really nothing more to the configuration unless you’re implementing one of the frameworks, which is strongly recommended if you’re going to put this into a production environment, and which we will explore later.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396697256-marker) We named the app “zarniwoop” because “hello-world” was used in the Digium wiki on ARI, and it seemed best to avoid overlap. You can of course name it anything you wish.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396682696-marker) If your computer has only one screen, now is probably the point where you’re thinking what a good idea it would be to have more of them.

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396640584-marker) Strictly speaking, REST is far more than that, but as a practical matter, these days it seems not uncommon to assume that a REST API will be URL and JSON-based, simply because so many such services are presented in those formats.

