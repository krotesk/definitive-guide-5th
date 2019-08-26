---
description: Основы диалплана
---

# Глава 6

> _Всё должно быть изложено так просто, как только возможно, но не более того._ 
>
> — Альберт Эйнштейн

The dialplan is the heart of your Asterisk system. It defines how calls flow into and out of the system. The dialplan is written in a scripting language, which specifies instructions that Asterisk follows in response to calls arriving from channels. In contrast to traditional phone systems, Asterisk’s dialplan is fully customizable.

Experienced software developers find Asterisk dialplan code archaic, and often prefer to control call flow using Asterisk APIs such as AMI and ARI \(which we will discuss in later chapters\). Regardless of your plans in this regard, learning how Asterisk behaves is far easier if you understand dialplan first. It is perhaps also worth noting that Asterisk dialplan is performance-tuned, and is therefore the fastest way to execute call flow in terms of responsiveness and minimal load on the system. Dialplan is fast.

This chapter introduces the essential concepts of the dialplan, which will form the basis of any dialplan you write. Do not skip too much of this chapter, since the examples build on each other, and it is so fundamentally important to Asterisk. Please also note that this chapter is by no means an exhaustive survey of all the possible things dialplans can do; our aim is to cover just the essentials. We’ll cover more advanced dialplan topics in later chapters. You are encouraged to experiment.

## Dialplan Syntax

The Asterisk dialplan is specified in the configuration file named extensions.conf, located in the /etc/asterisk directory.

The dialplan structure is built on four hierarchical components: Contexts, Extensions, Priorities, and Applications \(see [Figure 6-1](6.%20Dialplan%20Basics%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig0601)\).

![](.gitbook/assets/0%20%281%29.png)

**Figure 6-1. Dialplan hierarchy**

Let’s dive right in.

**Sample Configuration Files**

A basic extensions.conf file was created as part of the install process earlier in this book. We are going to build on that file in this chapter.

Asterisk also comes with a detailed extensions.conf file that can be installed with the sample configuration files \(the installation command make samples will do this\), and if you ran that command \(which we don’t recommend during the install, but is suggested by the installer\), you will most likely have an /etc/asterisk/extensions.conf file that is chock-full of information. Instead of starting with the sample file, we suggest that you build your extensions.conf file from scratch with a blank file \(you can rename it or move it somewhere if you wish to keep it as a reference\).

That being said, the extensions.conf.sample file is a fantastic resource, full of examples and ideas that you can use after you’ve learned the basic concepts. If you followed our installation instructions, you will find the file extensions.conf.sample in the folder ~/src/asterisk-15.&lt;TAB&gt;/configs/samples \(along with many other sample config files\).

### Contexts

Dialplans are divided into sections called contexts, which serve to separate different parts of the dialplan. An extension that is defined in one context is completely isolated from extensions in any other context, unless interaction is specifically allowed.

As a simple example, let’s imagine we have two companies sharing an Asterisk server. If we place each company’s automated attendant \(IVR\) in its own context, the two companies will be completely separated from each other. This allows us to independently define what happens when, say, extension 0 is dialed:

* Callers dialing 0 from Company A’s voice menu need to be transferred to Company A’s receptionist.
* Callers dialing 0 at Company B’s voice menu will be sent to Company B’s customer service department.

Both callers are on the same system, interacting with the same dialplan, but because they arrived in different contexts, they experience totally separate call flow. What happens to each incoming call is determined by the dialplan code in each context.

**Note**

This is a very important consideration. With traditional PBXs, there is generally a set of defaults for things like reception, which means that if you forget to define them, they will probably work anyway. In Asterisk, the opposite is true. If you do not tell Asterisk how to handle every situation, and it comes across something it cannot handle, the call will typically be disconnected.

Contexts are defined in the extensions.conf file by placing the name of the context inside square brackets \(\[\]\). The name can be made up of the letters A through Z \(upper- and lowercase\), the numbers 0 through 9, and the hyphen and underscore.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-1) A context for incoming calls from a carrier might simply be named this:

\[incoming\]

**Note**

Context names have a maximum length of 79 characters \(80 characters minus 1 terminating null\).

Or perhaps:

\[incoming\_company\_A\]

Which then of course might require something like:

\[incoming\_company\_B\]

All of the instructions placed after a context definition are part of that context, until the next context is defined.

At the beginning of the dialplan, there are two special sections named \[general\] and \[globals\]. The \[general\] section contains a list of general dialplan settings \(which you’ll probably never have to worry about\), and we will discuss the \[globals\] context shortly. For now, it’s only important to know that these two labels are not contexts, despite using context syntax. Do not use \[general\], \[globals\], and \[default\][2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408408152) as context names, but otherwise name your contexts anything you wish.

The contexts in a typical extensions.conf file might be structured something like this:

\[general\] ; This always has to be here

\[globals\] ; Global variables \(we'll discuss these later\)

\[incoming\] ; Calls from the carriers could arrive here

\[sets\] ; on a simple system, we can use this

\[sets1\] ; Multi-tenanted perhaps needs this \(sets from one company enter dialplan here\)

\[sets2\] ; ... and this \(another group of sets might enter the dialplan here\)

\[services\] ; Special services such as conferencing could be defined here

When you define a channel \(which is not done in the extensions.conf file\), one of the required parameters in each channel definition is context. The context is the point in the dialplan where connections coming from that channel will arrive. So the way you plug a channel into the dialplan is through the context \(see [Figure 6-2](6.%20Dialplan%20Basics%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22DialplanBasics_id36055184)\).

![](.gitbook/assets/1.png)

**Figure 6-2. Relation between channel configuration \(on the left\) and contexts in the dialplan \(on the right\)**

**Note**

This is one of the most important concepts to understand when dealing with channels and dialplans. Troubleshooting call flow is much easier once you understand the relationship between the channel context \(which tells the channel where to plug into the dialplan\) and the dialplan context \(which is where we create the call flow that happens when the call arrives\).

An important \(perhaps the most important\) use of contexts is to provide privacy and security. By using contexts correctly, you can give some channels access to features \(such as long-distance calling\) that aren’t made available to others. If you do not design your dialplan carefully, you may inadvertently allow others to fraudulently use your system. Please keep this in mind as you build your Asterisk system; there are many bots on the internet that were specifically written to identify and exploit poorly secured Asterisk systems.

**Warning**

The [Asterisk wiki](https://wiki.asterisk.org/wiki/display/AST/Important+Security+Considerations) outlines several steps you should take to keep your Asterisk system secure. It is vitally important that you read and understand this page. If you ignore the security precautions outlined there, you may end up allowing anyone and everyone to make long-distance or toll calls at your expense!

If you don’t take the security of your Asterisk system seriously, you may end up paying—literally. Please take the time and effort to secure your system from toll fraud.

### Extensions

In the telecommunications industry the word extension typically has referred to a numeric identifier that, when dialed, will ring a phone \(or system resource such as voicemail or a queue\). In Asterisk, an extension is far more powerful, as it defines the unique series of steps \(each step containing an application\) through which Asterisk will take that call.

Within each context, we can define as many \(or few\) extensions as required. When a particular extension is triggered \(by an incoming channel\), Asterisk will follow the steps defined for that extension. It is the extensions, therefore, that specify what happens to calls as they make their way through the dialplan. Although extensions can, of course, be used to specify phone extensions in the traditional sense \(i.e., extension 153 will cause the SIP telephone set on John’s desk to ring\), in an Asterisk dialplan, they can be used for much more.

The syntax for an extension is the word exten, followed by an arrow formed by the equals sign and the greater-than sign, like this:

exten =&gt;

This is followed by the name \(or number\) of the extension.

When dealing with traditional telephone systems, we tend to think of extensions as the numbers you would dial to make another phone ring. In Asterisk, extension names can be any combination of numbers and letters. Over the course of this chapter and the next, we’ll use both numeric and alphanumeric extensions.

**Tip**

Assigning names to extensions may seem like an unusual concept, but when you realize that SIP supports dialing by all sorts of character combinations \(anything that is a valid URI, strictly speaking\), it makes perfect sense. This is one of the features that makes Asterisk so flexible and powerful.

Each step in an extension has three components:

* The name \(or number\) of the extension
* The priority \(each extension can include multiple steps; the step number is called the “priority”\)
* The application \(or command\) that will take place at that step

These three components are separated by commas, like this:

exten =&gt; name,priority,application\(\)

Here’s a simple example:

exten =&gt; 123,1,Answer\(\)

The extension name is 123, the priority is 1, and the application is Answer\(\).

### Priorities

Each extension can have multiple steps, called priorities. The priorities are numbered sequentially, starting with 1, and each executes one specific application. As an example, the following extension would answer the phone in priority number 1, and then hang it up in priority number 2. The steps in an extension take place one after the other.

exten =&gt; 123,1,Answer\(\)

exten =&gt; 123,2,Hangup\(\)

It’s pretty obvious that this code doesn’t really do anything useful. We’ll get there. The key point to note here is that for a particular extension, Asterisk follows the priorities in order.

exten =&gt; 123,1,Answer\(\)

exten =&gt; 123,2,do something

exten =&gt; 123,3,do something else

exten =&gt; 123,4,do one last thing

exten =&gt; 123,5,Hangup\(\)

This style of dialplan syntax is still seen from time to time, although \(as you’ll see momentarily\) it is not generally used anymore for new code. Newer syntax is similar, but simplified.

#### Unnumbered priorities

In older releases of Asterisk, the numbering of priorities caused a lot of problems. Imagine having an extension that had 15 priorities, and then needing to add something at step 2: all of the subsequent priorities would have to be manually renumbered. Asterisk does not handle missing steps or misnumbered priorities, and debugging these types of errors was frustrating.

Beginning with version 1.2, Asterisk addressed this problem: it introduced the use of the n priority, which stands for “next.” Each time Asterisk encounters a priority named n, it takes the number of the previous priority and adds 1. This makes it easier to make changes to your dialplan, as you don’t have to keep renumbering all your steps. For example, your dialplan might look something like this:

exten =&gt; 123,1,Answer\(\)

exten =&gt; 123,n,do something

exten =&gt; 123,n,do something else

exten =&gt; 123,n,do one last thing

exten =&gt; 123,n,Hangup\(\)

Internally, Asterisk will calculate the next priority number every time it encounters an n.[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408333272) Now, if we want to add a new item at priority 3, we just input the new line where we need it, and no renumbering is required.

exten =&gt; 123,1,Answer\(\)

exten =&gt; 123,n,do something

exten =&gt; 123,n,SOME NEW THING

exten =&gt; 123,n,do something else

exten =&gt; 123,n,do one last thing

exten =&gt; 123,n,Hangup\(\)

Bear in mind that you must always specify priority number 1. If you accidentally put an n instead of 1 for the first priority \(a common mistake even among experienced dialplan coders\), you’ll find after reloading the dialplan that the extension will not exist.

#### The same =&gt; operator

In order to further simplify dialplan writing, a new syntax was created. As long as the extension remains the same, you can simply type same =&gt; followed by the priority and application rather than having to type the full extension on each line:

exten =&gt; 123,1,Answer\(\)

 same =&gt; n,do something

 same =&gt; n,do something

 same =&gt; n,do one last thing

 same =&gt; n,Hangup\(\)

This style of dialplan will also make it easier to copy code from one extension to another. This is the preferred and recommended style. The only reason for the discussion of previous styles is to help understand how we got here.

Make no mistake, the Asterisk dialplan is peculiar. Many folks avoid it altogether, and use AGI and ARI to write their dialplan.

While there’s certainly something to be said for writing dialplan in an external language \(and we’ll cover it in later chapters\), the Asterisk dialplan is native to it, and you will not get better performance than this. Dialplan code executes fast.

Also, if you want to understand how Asterisk thinks, you need to understand its dialplan.

#### Priority labels

Priority labels allow you to assign a name to a priority within an extension. This is to ensure that you can refer to a priority by something other than its number \(which probably isn’t known, given that dialplans now generally use unnumbered priorities\). Later you will learn that it’s often necessary to send calls from other parts of the dialplan to a particular priority in a particular extension. To assign a text label to a priority, simply add the label inside parentheses after the priority, like this:

exten =&gt; 123,n\(label\),application\(\)

Later, we’ll cover how to jump between different priorities based on dialplan logic. You’ll see a lot more of priority labels, and you’ll use them often in your dialplans.

**Warning**

A very common mistake when writing labels is to insert a comma between the n and the \(, like this:

exten =&gt; 555,n,\(label\),application\(\) ;&lt;-- THIS WON'T WORK

exten =&gt; 556,n\(label\),application\(\) :&lt;-- This is what we want

This mistake will break that part of your dialplan, and you will get an error stating that the application cannot be found.

### Applications

Applications are the workhorses of the dialplan. Each application performs a specific action on the current channel, such as playing a sound, accepting touch-tone input, looking something up in a database, dialing a channel, hanging up the call, feeding the cat, and so forth.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408295288) In the previous example, you were introduced to two simple applications: Answer\(\) and Hangup\(\). It’s obvious what they do, but it’s also obvious that on their own they aren’t terribly useful.

Some applications, including Answer\(\) and Hangup\(\), need no other instructions to do their jobs. Most applications, however, require more information. These additional elements, or arguments, are passed on to the applications to affect how they perform their actions. To pass arguments to an application, place them between the parentheses that follow the application name, separated by commas.

### The Answer\(\), Playback\(\), and Hangup\(\) Applications

The Answer\(\) application is used to answer a channel that is ringing. It seems a simple thing, but a lot of things happen on the channel with this one command. Answer\(\) tells the channel to send a message back to the far end that the call has been answered, and also to enable the media paths \(the network streams that will carry the sound between the caller and the system\). As we mentioned earlier, Answer\(\) takes no arguments. Answer\(\) is not always required \(in fact, in some cases it may not be desirable at all\), but it is an effective way to ensure a channel is connected before performing further actions.

**The Progress\(\) Application**

Sometimes it is useful to be able to pass information back to the network before answering a call. The Progress\(\) application attempts to provide call progress information to the originating channel. Some carriers expect this, and thus you may be able to resolve strange signaling problems by inserting Progress\(\) into the dialplan where your incoming calls arrive. In terms of billing, the use of Progress\(\) lets the carrier know you’re handling the call, without starting the billing meter.

The Playback\(\) application is used for playing a previously recorded sound file over a channel. Input from the user is ignored, which means that you would not use Playback\(\) in an auto attendant, for example, unless you did not want to accept input at that point.[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408274728)

**Tip**

Asterisk comes with many professionally recorded sound files, which should be found in the default sounds directory \(usually /var/lib/asterisk/sounds\). When you compile Asterisk, you can choose to install various sets of sample sounds that have been recorded in a variety of languages and file formats. We’ll be using these files in many of our examples. Several of the files in our examples come from the Extra Sound Package, which we installed in [Chapter 3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22asterisk-Install). You can also have your own sound prompts recorded in the same voices as the stock prompts by visiting [www.theivrvoice.com](http://www.theivrvoice.com/). Later in the book, we’ll talk more about how you can use a telephone and the dialplan to create and manage your own system recordings \(or import .wav files\).

To use Playback\(\), specify a filename as the argument. For example, Playback\(filename\) would play a sound file called filename.wav, assuming it was located in the default sounds directory. Note that you can include the full path to a file if you want, like this:

Playback\(/home/john/sounds/filename\)

The previous example would play filename.wav from the /home/john/sounds directory. This can be problematic, however, due to potential file permissions problems. If you’re planning on having a lot of custom sounds on your system, you’ll likely want a dedicated directory for them, and you’ll need to test to ensure Asterisk can find and play the files.

You can also use relative paths from the Asterisk sounds directory, as follows:

Playback\(custom/filename\)

This example would play filename.wav from the custom subdirectory of the default sounds directory \(probably /var/lib/asterisk/sounds/en/custom/filename.wav\). If the specified directory contains more than one file with that filename but with different file extensions, Asterisk automatically plays the best file.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-2)

The Hangup\(\) application does exactly as its name implies: it hangs up the active channel. You should use this application at the end of a context when you want to end the current call, to ensure that callers don’t continue on in the dialplan in a way you might not have anticipated. The Hangup\(\) application does not require any arguments, but you can pass an ISDN cause code if you want, such as Hangup\(16\), and it will be translated into a comparable SIP message and sent to the far end.

As we work through the book, we will be introducing you to many more Asterisk applications, but that’s enough theory for now; let’s write some dialplan!

### A Basic Dialplan Prototype

To reiterate, then, the form of all dialplans is built from those four concepts: Context, Extension, Priority, and Application \([Figure 6-3](6.%20Dialplan%20Basics%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig0603)\).

![](.gitbook/assets/2.png)

**Figure 6-3. Dialplan prototype**

## A Simple Dialplan

OK, enough theory. Open up the file /etc/asterisk/extensions.conf in your favorite editor, and let’s take a look at your first dialplan \(which was created in [Chapter 5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch05.html%22%20/l%20%22asterisk-DeviceConfig)\). We’re going to add to that.

### Hello World

As is typical in many technology books \(especially computer programming books\), our first example is called “Hello World.”

In the first priority of our extension, we answer the call. In the second, we play a sound file named hello-world, and in the third we hang up the call. The code we are interested in for this example looks like this:

exten =&gt; 200,1,Answer\(\)

 same =&gt; n,Playback\(hello-world\)

 same =&gt; n,Hangup\(\)

If you followed along in [Chapter 5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch05.html%22%20/l%20%22asterisk-DeviceConfig), you’ll already have a channel or two configured, as well as the sample dialplan that contains this code. If not, what you need is an extensions.conf file in your /etc/asterisk directory that contains the following code:

\[general\]

\[globals\]

\[sets\]

exten =&gt; 100,1,Dial\(PJSIP/0000f30A0A01\) ; Replace 0000f30A0A01 with your device name

exten =&gt; 101,1,Dial\(PJSIP/SOFTPHONE\_A\)

exten =&gt; 102,1,Dial\(PJSIP/0000f30B0B02\)

exten =&gt; 103,1,Dial\(PJSIP/SOFTPHONE\_B\)

exten =&gt; 200,1,Answer\(\)

 same =&gt; n,Playback\(hello-world\)

 same =&gt; n,Hangup\(\)

**Tip**

If you don’t have any channels configured, now is the time to do so. There is real satisfaction that comes from passing your first call into an Asterisk dialplan on a system that you’ve built from scratch. People get this funny grin on their faces as they realize that they have just created a telephone system. This pleasure can be yours as well, so please, don’t go any further until you have made this little bit of dialplan work. If you have any problems, get back to [Chapter 5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch05.html%22%20/l%20%22asterisk-DeviceConfig) and work through the examples there.

If you don’t have this dialplan code built yet, you’ll need to add it and reload the dialplan with this CLI command:

$ sudo asterisk -rvvvvv \# \('r' attaches to a daemonized Asterisk; 'v's are for verbosity\)

\*CLI&gt; dialplan reload

or you can issue the command directly from the shell with:

$ sudo asterisk -rx "dialplan reload" \# \('rx' execute an Asterisk command and return\)

Calling extension 200 from either of your configured phones[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408218280) should reward you with the friendly voice of Allison Smith saying “Hello, World.”

If it doesn’t work, check the Asterisk console for error messages, and make sure your channels are assigned to the sets context.

**Warning**

We do not recommend that you move forward in this book until you have verified the following:

1. Calls between extension 100 and 101 are working.
2. Calling extension 200 plays “Hello World.”

Even though this example is very short and simple, it emphasizes the core dialplan concepts of contexts, extensions, priorities, and applications. You now have the fundamental knowledge on which all dialplans are built.

As you build out a dialplan, it will be helpful to have the Asterisk CLI open in a new window. You will be reloading the dialplan often, and while testing your call flow, you will want to see what is happening, as it happens. The Asterisk CLI is useful for both of those things.

$ sudo asterisk -rvvvvv

\*CLI&gt; dialplan reload \# this Asterisk CLI command reloads the dialplan

Best practice, then, would be to edit in one window, and to reload and debug in another.

## Building an Interactive Dialplan

The dialplan we just built was static; it will always perform the same actions on every call. Many dialplans will also need logic to perform different actions based on input from the user, so let’s take a look at that now.

### The Goto\(\), Background\(\), and WaitExten\(\) Applications

As its name implies, the Goto\(\) application is used to send a call to another part of the dialplan. Goto\(\) requires us to pass the destination context, extension, and priority as arguments, like this:

 same =&gt; n,Goto\(context,extension,priority\)

We’re going to create a new context called TestMenu, and create an extension in our sets context that will pass calls to that context using Goto\(\):

exten =&gt; 200,1,Answer\(\)

 same =&gt; n,Playback\(hello-world\)

 same =&gt; n,Hangup\(\)

exten =&gt; 201,1,Goto\(TestMenu,start,1\) ; add this to the end of the

 ; \[sets\] context

\[TestMenu\]

exten =&gt; start,1,Answer\(\)

Now, whenever a device enters the \[sets\] context and dials 201, the call will be passed to the start extension in the TestMenu context \(which currently won’t do anything interesting because we still have more code to write\).

**Note**

We used the extension start in this example, but we could have used anything we wanted as an extension name, either numeric or alpha. We prefer to use alpha characters for extensions that are not directly dialable, as this makes the dialplan easier to read. Point being, we could have named our target extension 123 or xyz321, or 99luftballons, or whatever we wanted instead of start. The word start doesn’t mean anything special to the dialplan; it’s simply the name of an extension.

One of the more useful applications in an interactive Asterisk dialplan is the Background\(\)[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408187512) application. Like Playback\(\), it plays a recorded sound file. Unlike Playback\(\), however, when the caller presses a key \(or series of keys\) on their telephone keypad, it interrupts the playback and passes the call to the extension that corresponds with the pressed digit\(s\). If a caller presses 5, for example, Asterisk will stop playing the sound prompt and send control of the call to the first priority of extension 5 \(assuming there is an extension 5 to send the call to\).

The most common use of the Background\(\) application is to create basic voice menus \(often called auto attendants, IVRs,[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408181448) or phone trees\). Many companies use voice menus to direct callers to the proper extensions, thus relieving their receptionists from having to answer every single call.

Background\(\) has the same syntax as Playback\(\):

\[TestMenu\]

exten =&gt; start,1,Answer\(\)

 same =&gt; n,Background\(enter-ext-of-person\)

If you want Asterisk to wait for input from the caller after the sound prompt has finished playing, you can use WaitExten\(\). The WaitExten\(\) application waits for the caller to enter DTMF digits and is used directly following the Background\(\) application, like this:

\[TestMenu\]

exten =&gt; start,1,Answer\(\)

 same =&gt; n,Background\(enter-ext-of-person\)

 same =&gt; n,WaitExten\(\)

If you’d like the WaitExten\(\) application to wait a specific number of seconds for a response \(instead of using the default timeout\),[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408171000) simply pass the number of seconds as the first argument to WaitExten\(\), like this:

 same =&gt; n,WaitExten\(5\) ; We always pass a time argument to WaitExten\(\)

Both Background\(\) and WaitExten\(\) allow the caller to enter DTMF digits. Asterisk then attempts to find an extension in the current context that matches the digits that the caller entered. If Asterisk finds a match, it will send the call to that extension. Let’s demonstrate by adding a few lines to our dialplan example:

\[TestMenu\]

exten =&gt; start,1,Answer\(\)

 same =&gt; n,Background\(enter-ext-of-person\)

 same =&gt; n,WaitExten\(5\)

exten =&gt; 1,1,Playback\(digits/1\)

exten =&gt; 2,1,Playback\(digits/2\)

After making these changes, save and reload your dialplan:

\*CLI&gt; dialplan reload

If you call into extension 201, you should hear a sound prompt that says, “Enter the extension of the person you are trying to reach.” The system will then wait 5 seconds for you to enter a digit. If the digit you press is either 1 or 2, Asterisk will match the relevant extension, and read that digit back to you. Since we didn’t provide any further instructions, your call will then end. You’ll also find that if you enter a different digit \(such as 3\), the dialplan will be unable to proceed.

Let’s embellish things a little. We’re going to use the Goto\(\) application to have the dialplan repeat the greeting after playing back the number:

\[TestMenu\]

exten =&gt; start,1,Answer\(\)

 same =&gt; n,Background\(enter-ext-of-person\)

 same =&gt; n,WaitExten\(5\)

exten =&gt; 1,1,Playback\(digits/1\)

 same =&gt; n,Goto\(TestMenu,start,1\)

exten =&gt; 2,1,Playback\(digits/2\)

 same =&gt; n,Goto\(TestMenu,start,1\)

These new lines will send control of the call back to the start extension after playing back the selected number.

**Tip**

If you look up the details of the Goto\(\) application, you’ll find that you can actually pass either one, two, or three arguments to the application. If you pass a single argument, Asterisk will assume it’s the destination priority in the current extension. If you pass two arguments, Asterisk will treat them as the extension and the priority to go to in the current context.

In this example, we’ve passed all three arguments for the sake of clarity, but passing just the extension and priority would have had the same effect, since the destination context is the same as the source context.

### Handling Invalid Entries and Timeouts

We need an extension for invalid entries. In Asterisk, when a context receives a request for an extension that is not valid within that context \(e.g., pressing 9 in the preceding example\), the call is sent to the i extension. We also need an extension to handle situations when the caller doesn’t give input in time \(the default timeout is 10 seconds\). Calls will be sent to the t extension if the caller takes too long to press a digit after WaitExten\(\) has been called. Here is what our dialplan will look like after we’ve added these two extensions:

\[TestMenu\]

exten =&gt; start,1,Answer\(\)

 same =&gt; n,Background\(enter-ext-of-person\)

 same =&gt; n,WaitExten\(5\)

exten =&gt; 1,1,Playback\(digits/1\)

 same =&gt; n,Goto\(TestMenu,start,1\)

exten =&gt; 2,1,Playback\(digits/2\)

 same =&gt; n,Goto\(TestMenu,start,1\)

exten =&gt; i,1,Playback\(pbx-invalid\)

 same =&gt; n,Goto\(TestMenu,start,1\)

exten =&gt; t,1,Playback\(please-try-again\)

 same =&gt; n,Goto\(TestMenu,start,1\)

Using the i[11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408142552) and t extensions makes our menu a little more robust and user-friendly. That being said, it is still quite limited, because outside callers still have no way of connecting to a live person. To do that, we’ll need to learn about the Dial\(\) application.

### Using the Dial\(\) Application

One of Asterisk’s most valuable features is its ability to connect different callers to each other. While Asterisk currently is used mostly for SIP connections, it supports a wide variety of channel types \(from Analog to SS7, and various old VoIP protocols such as MGCP and SCCP\). Asterisk takes much of the hard work out of connecting and translating between disparate networks. All you have to do is learn how to use the Dial\(\) application.

The syntax of the Dial\(\) application is more complex than that of the other applications we’ve used so far, but it’s also where much of the magic of Asterisk happens. Dial\(\) takes up to four arguments, which we’ll look at next.

The syntax of Dial\(\) looks like this:

Dial\(Technology/Resource\[&Technology2/Resource2\[&...\]\]\[,timeout\[,options\[,URL\]\]\]\)

Put simply, you tell Dial\(\) what channel[12](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408129896) you want to send the call out to, and set a few options to tweak the behavior. The use of Dial\(\) can get complex, but at its most basic, it’s that simple.

#### Argument 1: destination

The first argument is the destination you’re attempting to call, which \(in its simplest form\) is made up of a technology \(or transport\) across which to make the call, a forward slash, and the address of the remote endpoint or resource.

**Note**

These days, you’re most likely to be using PJSIP as your channel type, but in the not-too-distant past, common technology types also included DAHDI \(for analog and T1/E1/J1 channels\), the old SIP channel \(prior to PJSIP\), and IAX2.[13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408125064) If you’re looking at an older dialplan, you may see some of these other protocols represented. Going forward, only PJSIP and DAHDI are recommended and maintained.

Let’s assume that we want to call one of our PJSIP channels named SOFTPHONE\_B. The technology is PJSIP, and the resource \(or channel\) identifier is SOFTPHONE\_B. Similarly, a call to a DAHDI device \(defined in chan\_dahdi.conf\) might have a destination of DAHDI/14169671111. If we wanted Asterisk to ring the PJSIP/SOFTPHONE\_B channel when extension 103 is reached in the dialplan, we’d add the following extension:

exten =&gt; 101,1,Dial\(PJSIP/SOFTPHONE\_A\)

exten =&gt; 103,1,Dial\(PJSIP/SOFTPHONE\_B\)

exten =&gt; 200,1,Answer\(\)

We can also dial multiple channels at the same time, by concatenating the destinations with an ampersand \(&\), like this:

exten =&gt; 101,1,Dial\(PJSIP/SOFTPHONE\_A\)

exten =&gt; 103,1,Dial\(PJSIP/SOFTPHONE\_B\)

exten =&gt; 110,1,Dial\(PJSIP/0000f30A0A01&PJSIP/SOFTPHONE\_A&PJSIP/SOFTPHONE\_B\)

exten =&gt; 200,1,Answer\(\)

The Dial\(\) application will ring all of the specified destinations simultaneously, and bridge the inbound call with whichever destination channel answers first \(the other channels will immediately stop ringing\). If the Dial\(\) application can’t contact any of the destinations, Asterisk will set a variable called DIALSTATUS with the reason that it couldn’t dial the destinations, and continue with the next priority in the extension.[14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-3)

The Dial\(\) application also allows you to connect to a remote VoIP endpoint not previously defined in one of the channel configuration files. The full syntax is:

Dial\(technology/user\[:password\]@remote\_host\[:port\]\[/remote\_extension\]\)

The full syntax for the Dial\(\) application is slightly different for DAHDI channels:

Dial\(DAHDI/\[gGrR\]channel\_or\_group\[/remote\_extension\]\)

For example, here is how you would dial 1-800-555-1212 on DAHDI channel number 4:[15](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408102536)

exten =&gt; 501,1,Dial\(DAHDI/4/18005551212\)

#### Argument 2: timeout

The second argument to the Dial\(\) application is a timeout, specified in seconds. If a timeout is given, Dial\(\) will attempt to call the specified destination\(s\) for that number of seconds before giving up and moving on to the next priority in the extension. If no timeout is specified, Dial\(\) will continue to dial the called channel\(s\) until someone answers or the caller hangs up. Let’s add a timeout of 10 seconds to our extension:

exten =&gt; 101,1,Dial\(PJSIP/SOFTPHONE\_A\)

exten =&gt; 102,1,Dial\(PJSIP/0000f30B0B02,10\)

exten =&gt; 103,1,Dial\(PJSIP/SOFTPHONE\_B\)

If the call is answered before the timeout, the channels are bridged and the dialplan is done. If the destination simply does not answer, is busy, or is otherwise unavailable, Asterisk will set a variable called DIALSTATUS and then continue on with the next priority in the extension.

Let’s put what we’ve learned so far into another example:

exten =&gt; 102,1,Dial\(PJSIP/0000f30B0B02,10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

As you can see, this example will play the vm-nobodyavail.gsm sound file if the call goes unanswered \(and then hang up\). Note that this doesn’t actually provide voicemail; we’re just playing a prompt, which could have been any valid prompt. We’ll cover sending calls to voicemail later.

#### Argument 3: option

The third argument to Dial\(\) is an option string. It may contain one or more characters that modify the behavior of the Dial\(\) application. While the list of possible options is too long to cover here, one of the most popular is the m option. If you place the letter m as the third argument, the calling party will hear hold music instead of ringing while the destination channel is being called \(assuming, of course, that music on hold has been configured correctly\). To add the m option to our last example, we simply change the first line:

exten =&gt; 102,1,Dial\(PJSIP/0000f30B0B02,10,m\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

#### Argument 4: URI

The fourth and final argument to the Dial\(\) application is a URI. If the destination channel supports receiving a URI at the time of the call, the specified URI will be sent \(for example, if you have an IP telephone that supports receiving a URI, it will appear on the phone’s display; likewise, if you’re using a softphone, the URI might pop up on your computer screen\). This argument is very rarely used.

#### Updating the dialplan

Let’s modify extensions 1 and 2 in our menu to use the Dial\(\) application, and add extensions 3 and 4 just for good measure:

\[TestMenu\]

exten =&gt; start,1,Answer\(\)

 same =&gt; n,Background\(enter-ext-of-person\)

 same =&gt; n,WaitExten\(5\)

exten =&gt; 1,1,Dial\(PJSIP/0000f30A0A01,10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; 2,1,Dial\(PJSIP/0000f30B0B02,10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; 3,1,Dial\(PJSIP/SOFTPHONE\_A,10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; 4,1,Dial\(PJSIP/SOFTPHONE\_B,10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; i,1,Playback\(pbx-invalid\)

 same =&gt; n,Goto\(TestMenu,start,1\)

exten =&gt; t,1,Playback\(vm-goodbye\)

 same =&gt; n,Hangup\(\)

#### Blank arguments

Note that the second, third, and fourth arguments may be left blank; only the first argument is required. For example, if you want to specify an option but not a timeout, simply leave the timeout argument blank, like this:

exten =&gt; 4,1,Dial\(SIP/SOFTPHONE\_B,,m\)

### Using Variables

If you have programming experience, you already understand what a variable is. If not, we’ll briefly explain what variables are and how they are used. Any dialplan work beyond the very simple examples just given will greatly benefit from the use of variables. They are one of the useful features of a customizable dialplan that you will not find in a typical proprietary PBX.

A variable is a named container that can hold a value. Think of it like a post office box. The advantage of a variable is that its contents may change, but its name does not, which means you can write code that references the variable name and not worry about what the value will be. It is almost impossible to do any sort of useful programming without variables.

There are two ways to reference a variable. To reference the variable’s name, simply type the name of the variable. If, on the other hand, you want to reference the value of the variable, you must type a dollar sign, an opening curly brace, the name of the variable, and a closing curly brace. So, to use the post office box analogy, you refer to the box itself by simply using its name, and you refer to the contents with the use of the ${} wrapper. A variable named MyVar is referred to as MyVar, and its contents are accessed with ${MyVar}. Here’s how we might use a variable inside the Dial\(\) application:[16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408063256)

exten =&gt; 203,1,Noop\(say some digits\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Set\(SomeDigits=123\)

 same =&gt; n,SayDigits\(${SomeDigits}\)

 same =&gt; n,Wait\(.25\)

 same =&gt; n,Set\(SomeDigits=543\)

 same =&gt; n,SayDigits\(${SomeDigits}\)

In our dialplan, whenever we refer to ${SomeDigits}, Asterisk will automatically replace it with whatever value has been assigned to the variable named SomeDigits.

**Tip**

Note that variable names are case-sensitive. A variable named SOMEDIGITS is different from a variable named SomeDigits. You should also be aware that any variables set by Asterisk will be uppercase. Some variables, such as CHANNEL and EXTEN, are reserved by Asterisk. You should not attempt to set these variables. It is popular to write global variables in uppercase and channel variables in Pascal/Camel case, but it is not strictly required.

There are three types of variables we can use in our dialplan: global variables, channel variables, and environment variables. Let’s take a moment to look at each type.

#### Global variables

As their name implies, global variables are visible to all channels at all times. Global variables are useful in that they can be used anywhere within a dialplan to increase readability and manageability. Suppose for a moment that you had a large dialplan and several hundred references to the PJSIP/0000f30A0A01 channel. Now imagine you replaced the phone with a different unit \(perhaps a different MAC address\), and had to go through your dialplan and change all of those references to PJSIP/0000f30A0A01. Not pretty.

On the other hand, if you had defined a global variable that contained the value PJSIP/0000f30A0A01 at the beginning of your dialplan and then referenced that instead, you would have to change only one line of code to affect all places in the dialplan where that channel was used.

Global variables should be declared in the \[globals\] context at the beginning of the extensions.conf file. As an example, we will create a few global variables that store the channel identifiers of our devices. These variables are set at the time Asterisk parses the dialplan:

\[globals\]

UserA\_DeskPhone=PJSIP/0000f30A0A01

UserA\_SoftPhone=PJSIP/SOFTPHONE\_A

UserB\_DeskPhone=PJSIP/0000f30B0B02

UserB\_SoftPhone=PJSIP/SOFTPHONE\_B

We’ll come back to these later.

#### Channel variables

A channel variable is a variable that is associated only with a particular call. Unlike global variables, channel variables are defined only for the duration of the current call and are available only to the channels participating in that call.

There are many predefined channel variables available for use within the dialplan, which are explained in the [Asterisk wiki](https://wiki.asterisk.org/wiki/display/AST/Channel+Variables). You define a channel variable with extension 203 and the Set\(\) application:

exten =&gt; 203,1,Noop\(say some digits\)

 same =&gt; n,Set\(SomeDigits=123\)

 same =&gt; n,SayDigits\(${SomeDigits}\)

 same =&gt; n,Wait\(.25\)

 same =&gt; n,Set\(SomeDigits=543\)

 same =&gt; n,SayDigits\(${SomeDigits}\)

You’re going to be seeing a lot more channel variables. Read on.

#### Environment variables

Environment variables are a way of accessing Unix environment variables from within Asterisk. These are referenced using the ENV\(\) dialplan function.[17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408032008) The syntax looks like ${ENV\(var\)}, where var is the Unix environment variable you wish to reference. Environment variables aren’t commonly used in Asterisk dialplans, but they are available should you need them.

#### Adding variables to our dialplan

Now that we’ve learned about variables, let’s put them to work in our dialplan. We’re going to add three global variables that will associate a variable name to a channel name:

\[general\]

\[globals\]

UserA\_DeskPhone=PJSIP/0000f30A0A01

UserA\_SoftPhone=PJSIP/SOFTPHONE\_A

UserB\_DeskPhone=PJSIP/0000f30B0B02

UserB\_SoftPhone=PJSIP/SOFTPHONE\_B

\[sets\]

exten =&gt; 100,1,Dial\(${UserA\_DeskPhone}\)

exten =&gt; 101,1,Dial\(${UserA\_SoftPhone}\)

exten =&gt; 102,1,Dial\(${UserB\_DeskPhone},10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; 103,1,Dial\(${UserB\_SoftPhone}\)

exten =&gt; 110,1,Dial\(${UserA\_DeskPhone}&${UserA\_SoftPhone}&${UserB\_SoftPhone}\)

exten =&gt; 200,1,Answer\(\)

Let’s update the test menu as well:

\[TestMenu\]

exten =&gt; start,1,Answer\(\)

 same =&gt; n,Background\(enter-ext-of-person\)

 same =&gt; n,WaitExten\(5\)

exten =&gt; 1,1,Dial\(${UserA\_DeskPhone},10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; 2,1,Dial\(${UserA\_SoftPhone},10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; 3,1,Dial\(${UserB\_DeskPhone},10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; 4,1,Dial\(${UserB\_SoftPhone},10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; i,1,Playback\(pbx-invalid\)

It rarely makes sense to hardcode data in a dialplan. It’s almost always better to use a variable.

Make sure you test this out to ensure you don’t have any typos, and also to see what it looks like when executed on the Asterisk CLI:

\# asterisk -rvvvvvv

\*CLI&gt; dialplan reload

 -- Executing \[201@sets:1\] Goto\("PJSIP/0000f30A0A01", "TestMenu,start,1"\)

 -- Goto \(TestMenu,start,1\)

 -- Exec \[start@TestMenu:1\] Answer\("PJSIP/0000f30A0A01", ""\)

 -- Exec \[start@TestMenu:2\] BackGround\("PJSIP/0000f30A0A01", "enter-ext-of-person"\)

 -- &lt;PJSIP/0000f30A0A01&gt; Playing 'enter-ext-of-person.slin' \(language 'en'\)

 -- Exec \[1@TestMenu:1\] Dial\("PJSIP/0000f30A0A01", "PJSIP/0000f30A0A01,10"\)

 -- Called PJSIP/0000f30A0A01

 -- PJSIP/0000f30A0A01-00000011 is ringing

 == Spawn extension \(TestMenu, 1, 1\) exited non-zero on 'PJSIP/0000f30A0A01'

#### Variable concatenation

To concatenate variables, simply place them together, like this:

exten =&gt; 204,1,Answer\(\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Set\(ONETWO=12\)

 same =&gt; n,Set\(THREEFOUR=34\)

 same =&gt; n,SayDigits\(${ONETWO}${THREEFOUR}\) ; easy peasy

 same =&gt; n,Wait\(0.2\)

 same =&gt; n,Set\(NOTFIVE=${THREEFOUR}${ONETWO}\) ; peasy easy

 same =&gt; n,SayNumber\(${NOTFIVE}\) ; see what we did here?

 same =&gt; n,Wait\(0.2\)

 same =&gt; n,SayDigits\(2${ONETWO}3\) ; you can concatenate literals and variables

#### Inheriting channel variables

Channel variables are always associated with the original channel that set them, and are no longer available once the channel is transferred.

In order to allow channel variables to follow the channel as it is transferred around the system, you must employ channel variable inheritance. There are two modifiers that can allow the channel variable to follow the channel: single underscore and double underscore.

The single underscore \(\_\) causes the channel variable to be inherited by the channel for a single transfer, after which it is no longer available for additional transfers. If you use a double underscore \(\_\_\), the channel variable will be inherited throughout the life of that channel.

Setting channel variables for inheritance simply requires you to prefix the channel name with a single or double underscore. The channel variables are then referenced exactly the same as they would be normally.

Here’s an example of setting a channel variable for single transfer inheritance:

exten =&gt; example,1,Set\(\_MyVariable=thisValue\)

Here’s an example of setting a channel variable for infinite transfer inheritance:

exten =&gt; example,1,Set\(\_\_MyVariable=thisValue\)

When you wish to read the value of the channel variable, you do not use the underscore\(s\):

exten =&gt; example,1,Verbose\(1,Value of MyVariable is: ${MyVariable}\)

### Pattern Matching

If we want to be able to allow people to dial through Asterisk and have Asterisk connect them to outside resources, we need a way to match on any possible phone number that the caller might dial. For situations like this, Asterisk offers pattern matching. Pattern matching allows you to create one extension in your dialplan that matches many different numbers. This is enormously useful.

#### Pattern-matching syntax

When we are using pattern matching, certain letters and symbols represent what we are trying to match. Patterns always start with an underscore \(\_\). This tells Asterisk that we’re matching on a pattern, and not on an explicit extension name.

**Warning**

If you forget the underscore at the beginning of your pattern, Asterisk will think it’s just a named extension and won’t do any pattern matching. This is one of the most common mistakes people make when starting to learn Asterisk.

After the underscore, you can use one or more of the following characters:

X

Matches any single digit from 0 to 9.

Z

Matches any single digit from 1 to 9.

N

Matches any single digit from 2 to 9.

**Note**

Another common mistake is to try to use the letters X, Z, and N literally in a pattern match; to do that, wrap them in square brackets \(case-insensitive\), such as \_ale\[X\]\[Z\]a\[N\]der.

\[15-7\]

Matches a single character from the range of digits specified. In this case, the pattern matches a single 1, as well as any number in the range 5, 6, 7.

. \(period\)

Wildcard match; matches one or more characters, no matter what they are.

**Warning**

If you’re not careful, wildcard matches can make your dialplans do things you’re not expecting \(like matching built-in extensions such as i or h\). You should use the wildcard match in a pattern only after you’ve matched as many other digits as possible. For example, the following pattern match should probably never be used:

\_.

In fact, Asterisk will warn you if you try to use it. Instead, if you really need a catchall pattern match, use this one to match all strings that start with a digit followed by one or more characters \(see ! if you want to be able to match on zero or more characters\):

\_X.

Or this one, to match any alphanumeric string:

\_\[0-9a-zA-Z\].

! \(bang\)

Wildcard match; matches zero or more characters, no matter what they are.

To use pattern matching in your dialplan, simply put the pattern in the place of the extension name \(or number\):

exten =&gt; \_4XX,1,Noop\(User Dialed ${EXTEN}\)

 same =&gt; n,Answer\(\)

 same =&gt; n,SayDigits\(${EXTEN}\)

 same =&gt; n,Hangup\(\)

In this example, the pattern matches any three-digit extension from 400 through 499.[18](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178407965128)

One other important thing to know about pattern matching is that if Asterisk finds more than one pattern that matches the dialed extension, it will use the most specific one \(going from left to right\). Say you had defined the following two patterns, and a caller dialed 555-1212:

exten =&gt; \_555XXXX,1,Answer\(\)

 same =&gt; n,SayDigits\(${EXTEN}\)

exten =&gt; \_55512XX,1,Answer\(\)

 same =&gt; n,Playback\(tt-monkeys\)

In this case the second extension would be selected, because it is more specific. Load this in and make calls to 5550000, 5550123, 5551212, 5551200, 5551300, 5551299, and so forth to get a feel for how this works. Play around with different pattern matches. For example, what would pattern \_555NNNN match? What would pattern \_\[0-9\]. match?

#### North American Numbering Plan—pattern-matching examples

This pattern matches any seven-digit number, as long as the first digit is 2 or higher:

\_NXXXXXX

The preceding pattern would be compatible with any North American Numbering Plan local seven-digit number.

In areas with 10-digit dialing, that pattern would look like this:

\_NXXNXXXXXX

Note that neither of these two patterns would handle long-distance calls. We’ll cover those shortly.

**The NANP and Toll Fraud**

The North American Numbering Plan \(NANP\) is a shared telephone numbering scheme used by 19 countries in North America and the Caribbean. All of these countries share country code 1.

In the United States and Canada, there is sufficient competition that you can place a long-distance call to most numbers in country code 1 and expect to pay a reasonable toll. However, many people don’t realize that 17 other countries, many of which have very different telecom regulations, [share the NANP](http://www.nanpa.com/). Some of these places are quite expensive to call.

One popular scam using the NANP tries to trick naïve North Americans into calling expensive per-minute toll numbers in a Caribbean country; the callers believe that since they dialed 1-NPA-NXX-XXXX to reach the number, they’ll be paying their standard national long-distance rate for the call. Since the country in question may have regulations that allow for this form of extortion, the caller is ultimately held responsible for the call charges.

It may be prudent to block calls to area codes to NANP countries outside the US and Canada until you’ve had a chance to review your toll rates to those countries. Wikipedia has [a good reference](http://bit.ly/2Ztku7l) for the basics of what you need to know about NANP, including what NPAs \(area codes\) belong to what country.

Let’s try another:

\_1NXXNXXXXXX

This one will match the number 1, followed by an area code between 200 and 999, then any seven-digit number that does not start with 0 or 1. In the NANP calling area, you would use this pattern to match any long-distance number.[19](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-8)

And finally this one:

\_011.

Note the period on the end. This pattern matches any number that starts with 011 and has at least one more digit. In the NANP, this indicates an international phone number. \(We’ll be using these patterns in the next section to add outbound dialing capabilities to our dialplan.\)

#### Common global pattern matches

Outside of North America, there is wide variance in how numbering is handled; however, some patterns are common. Here are a few simple examples:

; UK, Germany, Italy, China, etc.

exten =&gt; \_00X.,1,noop\(\) ; international dialing code

exten =&gt; \_0X.,1,noop\(\) ; national dialing prefix

exten =&gt; 112,1,Noop\(--==\[ Emergency call \]==--\)

; Australia

exten =&gt; \_0011X.,1,noop\(\) ; international dialing code

exten =&gt; \_0X.,1,noop\(\) ; national dialing prefix

; Dutch Caribbean \(Saba\)

exten =&gt; \_00X.,1,noop\(\) ; international

exten =&gt; \_416XXXX,1,noop\(\) ; local \(on-island\)

exten =&gt; \_0\[37\]XXXXXX,1,noop\(\) ; call to country code 599 off-island \(not Curacao\)

exten =&gt; \_09XXXXXXX,1,Noop\(\) ; call to country code 599 off-island \(Curacao\)

You will need to understand the dialing plan of your region in order to produce a useful pattern match.

#### Using the ${EXTEN} channel variable

So what happens if you want to use pattern matching but need to know which digits were actually dialed? Enter the ${EXTEN} channel variable. Whenever you dial an extension, Asterisk sets the ${EXTEN} channel variable to the digits that were received. We used the application SayDigits\(\) to demonstrate this.

exten =&gt; \_4XX,1,Noop\(User Dialed ${EXTEN}\)

 same =&gt; n,Answer\(\)

 same =&gt; n,SayDigits\(${EXTEN}\)

 same =&gt; n,Hangup\(\)

exten =&gt; \_555XXXX,1,Answer\(\)

 same =&gt; n,SayDigits\(${EXTEN}\)

In these examples, the SayDigits\(\) application read back to you the extension you dialed.

Often, it’s useful to manipulate the ${EXTEN} by stripping a certain number of digits off the front of the extension. This is accomplished by using the syntax ${EXTEN:x}, where x is where you want the returned string to start, from left to right. For example, if the value of ${EXTEN} is 95551212, ${EXTEN:1} equals 5551212. Let’s try another example:

exten =&gt; \_XXX,1,Answer\(\)

 same =&gt; n,SayDigits\(${EXTEN:1}\)

In this example, the SayDigits\(\) application would start at the second digit, and thus read back only the last two digits of the dialed extension.

**More Advanced Digit Manipulation**

The ${EXTEN} variable properly has the syntax ${EXTEN:x:y}, where x is the starting position and y is the number of digits to return. Given the following dial string:

94169671111

we can extract the following digit strings using the ${EXTEN:x:y} construct:

* ${EXTEN:1:3} would contain 416
* ${EXTEN:4:7} would contain 9671111
* ${EXTEN:-4:4} would start four digits from the end and return four digits, giving us 1111
* ${EXTEN:2:-4} would start two digits in and exclude the last four digits, giving us 16967
* ${EXTEN:-6:-4} would start six digits from the end and exclude the last four digits, giving us 67
* ${EXTEN:1} would give us everything after the first digit, or 4169671111 \(if the number of digits to return is left blank, it will return the entire remaining string\)

This is a very powerful construct, but most of these variations are not very common in normal use. For the most part, you will be using ${EXTEN} \(or perhaps ${EXTEN:1} if you need to strip off an external access code, such as a prepended 9\).

### Includes

Asterisk has an important feature that allows extensions from one context to be available from within another context. This is accomplished through use of the include directive, which allows us to control access to different sections of the dialplan.

The include statement takes the following form, where context is the name of the remote context we want to include in the current context:

include =&gt; context

Including one context within another context allows extensions within the included context to be dialable.

When we include other contexts within our current context, we have to be mindful of the order in which we are including them. Asterisk will first try to match the dialed extension in the current context. If unsuccessful, it will then try the first included context \(including any contexts included in that context\), and then continue to the other included contexts in the order in which they were included.

We will discuss the include directive more in [Chapter 7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22asterisk-OutsideConn).

## Conclusion

And there you have it—a basic but functional dialplan. There is still much we have not covered, but you’ve got all of the fundamentals. In the following chapters, we’ll continue to build on this foundation.

If parts of this dialplan don’t make sense, you may want to go back and reread a section or two before continuing on to the next chapter. It’s imperative that you understand these principles and how to apply them, as the next chapters build on this information.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-1-marker) Please note that the space is conspicuously absent from the list of allowed characters. Don’t use spaces in your context names—you won’t like the result!

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408408152-marker) The default context used to be a popular way to whip up simple configurations, but this proved to be somewhat problematic for security. Best practice these days is to avoid all use of it.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408333272-marker) Asterisk permits simple arithmetic within the priority, such as n+200, and the priority s \(for same\), but their usage is somewhat deprecated due to the existence of priority labels. Please note that extension s and priority s are two distinct concepts.

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408295288-marker) OK, so feeding the cat isn’t a common use for a telephone system, but through Asterisk, such things are not impossible. Doc Brown would’ve loved this thing.

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408274728-marker) There is another application called Background\(\) that is very similar to Playback\(\), except that it does allow input from the caller. You can read more about this application in Chapters [14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22asterisk-AA) and [16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22asterisk-IVR).

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-2-marker) Asterisk selects the best file based on translation cost—that is, it selects the file that is the least CPU-intensive to convert to its native audio format. When you start Asterisk, it calculates the translation costs between the different audio formats \(they often vary from system to system\). You can see these translation costs by typing core show translation at the Asterisk CLI. The numbers shown represent how many microseconds it takes Asterisk to transcode one second of audio.

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408218280-marker) If you haven’t configured two phones yet, please consider heading back to [Chapter 5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch05.html%22%20/l%20%22asterisk-DeviceConfig) and getting a couple of phones set up so you can play with them. You can get away with only one phone for testing, but really two is ideal. There are lots of free softphones available, and some of them are rather good.

[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408187512-marker) It should be noted that some people expect that Background\(\), due to its name, will continue onward through the next steps in the dialplan while the sound is being played. In reality, its name refers to the fact that it is playing a sound in the background, while waiting for DTMF in the foreground.

[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408181448-marker) More information about auto attendants and IVR can be found in [Chapter 14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22asterisk-AA).

[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408171000-marker) See the dialplan function TIMEOUT\(\) for information on how to change the default timeouts. See [Chapter 10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-DP-Deeper) for information on what dialplan functions are.

[11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408142552-marker) The i extension is for catching invalid entries supplied to a dialplan application such as Background\(\). It is not used for matching on invalidly dialed extensions or nonmatching pattern matches.

[12](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408129896-marker) Or channels, if you want to ring more than one at a time.

[13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408125064-marker) IAX2 \(pronounced “EEKS”\), is the Inter Asterisk Exchange protocol \(v2\). In the early days of Asterisk it was popular for trunking, as it greatly reduced signaling overhead on busy circuits. Bandwidth has become far less expensive, and SIP protocol has become nearly ubiquitous. The IAX2 protocol is no longer actively maintained, but it still retains some popularity for its ability to traverse firewalls, and a few carriers might still support it. However, its use is deprecated, and in fact discouraged.

[14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-3-marker) We’ll cover variables in the section [“Using Variables”](6.%20Dialplan%20Basics%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22asterisk-DP-Basics-SECT-3.5). In future chapters we’ll discuss how to have your dialplan make decisions based on the value of DIALSTATUS.

[15](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408102536-marker) Bear in mind that this assumes that this channel connects to something that knows how to reach external numbers.

[16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408063256-marker) Specifically, what we are setting here is a channel variable.

[17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408032008-marker) We’ll get into dialplan functions later. Don’t worry too much about environment variables right now. They are not important to understanding the dialplan.

[18](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178407965128-marker) We’ve used the EXTEN channel variable without introducing it. Read on, as it will be covered later in this chapter.

[19](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-8-marker) If you grew up in North America, you may believe that the 1 you dial before a long-distance call is “the long-distance code.” This is not completely correct. The number 1 is also the international country code for NANP. Keep this in mind if you send your phone number to someone in another country. The recipient may not know your country code, and thus be unable to call you with just your area code and phone number. Your full phone number with country code is +1 NPA NXX XXXX \(where NPA is your area code\)—for example, +1 416 555 1212. This is also known as E.164 format \([Wikipedia](http://bit.ly/2ImniNO) can tell you all about E.164\).

