---
description: Погружение в диалплан
---

# Глава 10

For a list of all the ways technology has failed to improve the quality of life, please press three.

Alice Kahn

Alrighty. You’ve got the basics of dialplans down, but you know there’s more to come. If you don’t have [Chapter 6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics) sorted out yet, please go back and give it another read. We’re about to get into more advanced topics.

## Expressions and Variable Manipulation

As we begin our dive into the deeper aspects of dialplans, it is time to introduce you to a few tools that will greatly add to the power you can exercise in your dialplan. These constructs add incredible intelligence to your dialplan by enabling it to make decisions based on different criteria you define. Put on your thinking cap, and let’s get started.

**Note**

Throughout this chapter we use best practices that have been developed over the years in dialplan creation. The primary one you’ll notice is that all the first priorities start with the NoOp\(\) application \(which simply means No Operation; nothing functional will happen\). The other one is that all following lines will start with same =&gt; n, which is a shortcut that says, “Use the same extension as was just previously defined.” Additionally, the indentation is two spaces.

### Basic Expressions

Expressions are combinations of variables, operators, and values that you string together to produce a result. An expression can test values, alter strings, or perform mathematical calculations. Let’s say we have a variable called COUNT. In plain English, two expressions using that variable might be \[COUNT plus 1\], or \[COUNT divided by 2\]. Each of these expressions has a particular result or value, depending on the value of the given variable.

In Asterisk, expressions always begin with a dollar sign and an opening square bracket and end with a closing square bracket, as shown here:

$\[expression\]

Thus, we would write our two examples like this:

$\[${COUNT} + 1\]

$\[${COUNT} / 2\]

When Asterisk encounters an expression in a dialplan, it replaces the entire expression with the resulting value. It is important to note that this takes place after variable substitution. To demonstrate, let’s look at the following code:[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-CHP-6-FN-1)

exten =&gt; 321,1,NoOp\(\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Set\(COUNT=3\)

 same =&gt; n,Set\(NEWCOUNT=$\[${COUNT} + 1\]\)

 same =&gt; n,SayNumber\(${NEWCOUNT}\)

In the second priority, we assign the value of 3 to the variable named COUNT.

In the third priority, only one application—Set\(\)—is involved, but three things actually happen:

1. Asterisk substitutes ${COUNT} with the number 3 in the expression. The expression effectively becomes this:

 same =&gt; n,Set\(NEWCOUNT=$\[3 + 1\]\)

1. Asterisk evaluates the expression, adding 1 to 3, and replaces it with its computed value of 4:

 same =&gt; n,Set\(NEWCOUNT=4\)

1. The Set\(\) application assigns the value 4 to the NEWCOUNT variable.

The third priority simply invokes the SayNumber\(\) application, which speaks the current value of the variable ${NEWCOUNT} \(set to the value 4 in priority two\).

Try it out in your own dialplan.

### Operators

When you create an Asterisk dialplan, you’re really writing code in a specialized scripting language. This means that the Asterisk dialplan—like any programming language—recognizes symbols called operators that allow you to manipulate variables. Let’s look at the types of operators that are available in Asterisk:

Boolean operators

These operators evaluate the “truth” of a statement. In computing terms, that essentially refers to whether the statement is something or nothing \(nonzero or zero, true or false, on or off, and so on\). The Boolean operators are:

expr1 \| expr2

This operator \(called the “or” operator, or “pipe”\) returns the evaluation of expr1 if it is true \(neither an empty string nor zero\). Otherwise, it returns the evaluation of expr2.

expr1 & expr2

This operator \(called “and”\) returns the evaluation of expr1 if both expressions are true \(i.e., neither expression evaluates to an empty string or zero\). Otherwise, it returns zero.

expr1 {=, &gt;, &gt;=, &lt;, &lt;=, !=} expr2

These operators return the results of an integer comparison if both arguments are integers; otherwise, they return the results of a string comparison. The result of each comparison is 1 if the specified relation is true, or 0 if the relation is false. \(If you are doing string comparisons, they will be done in a manner that’s consistent with the current local settings of your operating system.\)

Mathematical operators

Want to perform a calculation? You’ll want one of these:

expr1 {+, -} expr2

These operators return the results of the addition or subtraction of integer-valued arguments.

expr1 {\*, /, %} expr2

These return, respectively, the results of the multiplication, integer division, or remainder of integer-valued arguments.

Regular expression operator

You can also use the regular expression operator in Asterisk:

**Note**

Some additional information about the peculiarities of the regular expression operator in Asterisk can be found at [Walter Doekes’s website](http://wjd.nu/notes/2011%22%20/l%20%22asterisk-dialplan-peculiarities-regex).

expr1 : expr2

This operator matches expr1 against expr2, where expr2 must be a regular expression.[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-CHP-6-FN-2) The regular expression is anchored to the beginning of the string with an implicit ^.[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-CHP-6-FN-3)

If the pattern contains no subexpression, the number of matched characters is returned. This will be 0 if the match failed. If the pattern contains a subexpression -- \\(...\\) -- the string corresponding to \1 is returned. If the match fails, the empty string is returned.

expr1 =~ expr2

This operator works the same as the : operator, except that it is not anchored to the beginning.

## Dialplan Functions

Dialplan functions allow you to add more power to your expressions; you can think of them as intelligent variables. Dialplan functions allow you to calculate string lengths, dates and times, MD5 checksums, and so on, all from within a dialplan expression.

**Note**

You’ll see usage of Playback\(silence/1\) throughout the examples in this chapter. We are doing this as it will answer the line if it hasn’t already been answered for us, and plays back some silence on the line. This allows other applications such as SayNumber\(\) to play back audio without gaps.

### Syntax

Dialplan functions have the following basic syntax:

FUNCTION\_NAME\(argument\)

You reference a function’s name the same way as a variable’s name, but you reference a function’s value with the addition of a dollar sign, an opening curly brace, and a closing curly brace:

${FUNCTION\_NAME\(argument\)}

Functions can also encapsulate other functions, like so:

${FUNCTION\_NAME\(${FUNCTION\_NAME\(argument\)}\)}

 ^ ^ ^ ^ ^^^^

 1 2 3 4 4321

As you’ve probably already figured out, you must be very careful about making sure you have matching parentheses and braces. In the preceding example, we have labeled the opening parentheses and curly braces with numbers and their corresponding closing counterparts with the same numbers.

### Examples of Dialplan Functions

Functions are often used in conjunction with the Set\(\) application to either get or set the value of a variable. As a simple example, let’s look at the LEN\(\) function. This function calculates the string length of its argument:

exten =&gt; 205,1,Answer\(\)

 same =&gt; n,SayDigits\(123\)

 same =&gt; n,SayNumber\(123\)

 same =&gt; n,SayNumber\(${LEN\(123\)}\)

Let’s look at another simple example. If we wanted to set one of the various channel timeouts, we could use the TIMEOUT\(\) function. The TIMEOUT\(\) function accepts one of three arguments: absolute, digit, and response. To set the digit timeout with the TIMEOUT\(\) function, we could use the Set\(\) application, like so:

exten =&gt; 206,1,Answer\(\)

 same =&gt; n,Set\(TIMEOUT\(response\)=1\)

 same =&gt; n,Background\(enter-ext-of-person\)

 same =&gt; n,WaitExten\(\) ; TIMEOUT\(\) has set this to 1

 same =&gt; n,Playback\(like\_to\_tell\_valid\_ext\)

 same =&gt; n,Set\(TIMEOUT\(response\)=5\)

 same =&gt; n,Background\(enter-ext-of-person\)

 same =&gt; n,WaitExten\(\) ; Should be 5 seconds now

 same =&gt; n,Playback\(like\_to\_tell\_valid\_ext\)

 same =&gt; n,Hangup\(\)

Notice the lack of ${ } surrounding the assignment using the function. Just as if we were assigning a value to a variable, we assign a value to a function without the use of the ${ } encapsulation; however, if we want to use the value returned by the function, then we need the encapsulation.

exten =&gt; 207,1,Answer\(\)

 same =&gt; n,Set\(TIMEOUT\(response\)=1\)

 same =&gt; n,SayNumber\(${TIMEOUT\(response\)}\)

 same =&gt; n,Set\(TIMEOUT\(response\)=5\)

 same =&gt; n,SayNumber\(${TIMEOUT\(response\)}\)

 same =&gt; n,Hangup\(\)

You can get a list of all active functions with the following CLI command:

\*CLI&gt; core show functions

Or, to see a specific function such as CALLERID\(\), the command is:

\*CLI&gt; core show function CALLERID

Near the end of this chapter, we explore a handful of functions you will want to experiment with. Later in the book we’ll show you how to create database-based functions using func\_odbc.

## Conditional Branching

The advanced logic provided through expressions and functions will allow your dialplan to make more powerful decisions, which will often result in conditional branching.

### The GotoIf\(\) Application

The key to conditional branching is the GotoIf\(\) application. GotoIf\(\) evaluates an expression and sends the caller to a specific destination based on whether the expression evaluates to true or false.

GotoIf\(\) uses a special syntax, often called the conditional syntax:

GotoIf\(expression?destination1:destination2\)

If the expression evaluates to true, the caller is sent to destination1. If the expression evaluates to false, the caller is sent to the second destination. So, what is true and what is false? An empty string and the number 0 evaluate as false. Anything else evaluates as true.

The destinations can each be one of the following:

* A priority label within the same extension, such as weasels
* An extension and a priority label within the same context, such as 123,weasels
* A context, extension, and priority label, such as incoming,123,weasels

Let’s use GotoIf\(\) in an example. Here’s a little coin toss application. Call it several times to properly test.

exten =&gt; 209,1,Noop\(Test use of conditional branching to labels\)

 same =&gt; n,GotoIf\($\[ ${RAND\(0,1\)} = 1 \]?weasels:iguanas\)

; same =&gt; n,GotoIf\(${RAND\(0,1\)}?weasels:iguanas\) ; works too, but won't in every situation

 same =&gt; n\(weasels\),Playback\(weasels-eaten-phonesys\) ; NOTE THIS IS SAME EXTENSION

 same =&gt; n,Hangup\(\)

 same =&gt; n\(iguanas\),Playback\(office-iguanas\) ; STILL THE SAME EXTENSION

 same =&gt; n,Hangup\(\)

**Note**

You will notice that we have used the Hangup\(\) application following each use of the Playback\(\) application. This is done so that when we jump to the weasels label, the call stops before execution gets to the office-iguanas sound file. It is becoming increasingly common to see extensions broken up into multiple components \(protected from each other by the Hangup\(\) command\), each one a distinct sequence of steps executed following a GotoIf\(\).

#### Providing Only a False Conditional Path

Either of the destinations may be omitted \(but not both\). If an expression evaluates to a blank destination, Asterisk simply goes on to the next priority in the current extension.

We could have crafted the preceding example like this:

exten =&gt; 209,1,Noop\(Test use of conditional branching\)

 same =&gt; n,GotoIf\($\[ ${RAND\(0,1\)} = 1 \]?:iguanas\)

 same =&gt; n,Playback\(weasels-eaten-phonesys\) ; No weasels label anymore

 same =&gt; n,Hangup\(\)

 same =&gt; n\(iguanas\),Playback\(office-iguanas\) ; NOTE THIS IS THE SAME EXTEN

 same =&gt; n,Hangup\(\)

There’s nothing between the ? and the : so if the statement evaluates to true, execution will continue at the next step. Since that’s what we want, a label isn’t needed.

We don’t really recommend doing this, because it’s hard to read. Nevertheless, you will see dialplans like this, so it’s good to be aware that this syntax is technically correct.

Rather than using labels, we could also send the call to different extensions. Since they’re not dialable, we can use alphabet characters rather than digits for the extension “numbers.” In this example, the conditional branch sends the call to completely different extensions within the same context. The result is otherwise the same.

exten =&gt; 210,1,Noop\(Test use of conditional branching to extensions\)

 same =&gt; n,GotoIf\($\[ ${RAND\(0,1\)} = 1 \]?weasels,1:iguanas,1\)

exten =&gt; weasels,1,Playback\(weasels-eaten-phonesys\) ; DIFFERENT EXTENSION

 same =&gt; n,Hangup\(\)

exten =&gt; iguanas,1,Playback\(office-iguanas\) ; ALSO A DIFFERENT EXTEN

 same =&gt; n,Hangup\(\)

Let’s look at another example of conditional branching. This time, we’ll use both Goto\(\) and GotoIf\(\) to count down from 5 and then hang up:

exten =&gt; 211,1,NoOp\(\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Set\(COUNT=5\)

 same =&gt; n\(start\),GotoIf\($\[ ${COUNT} &gt; 0 \]?:goodbye\)

 same =&gt; n,SayNumber\(${COUNT}\)

 same =&gt; n,Set\(COUNT=$\[ ${COUNT} - 1 \]\)

 same =&gt; n,Goto\(start\)

 same =&gt; n\(goodbye\),Playback\(vm-goodbye\)

 same =&gt; n,Hangup\(\)

Let’s analyze this example. In the second priority, we set the variable COUNT to 5. Next, we check to see if COUNT is greater than 0. If it is, we move on to the next priority. \(Don’t forget that if we omit a destination in the GotoIf\(\) application, control goes to the next priority.\) From there, we speak the number, subtract 1 from COUNT, and go back to priority label start. Again, if COUNT is less than or equal to 0, control goes to priority label goodbye; otherwise, we run through the loop one more time.

#### Quoting and Prefixing Variables in Conditional Branches

Now is a good time to take a moment to look at some nitpicky stuff with conditional branches. In Asterisk, it is invalid to have a null value on either side of the comparison operator. Let’s look at examples that would produce an error:

$\[ = 0 \]

$\[ foo = \]

$\[ &gt; 0 \]

$\[ 1 + \]

Any of our examples would produce a warning like this:

WARNING\[28400\]\[C-000000eb\]: ast\_expr2.fl:470 ast\_yyerror: ast\_yyerror\(\):

syntax error: syntax error, unexpected '=', expecting $end; Input:

 = 0

 ^

It’s fairly unlikely \(unless you have a typo\) that you’d purposefully implement something like our examples. However, when you perform math or a comparison with an unset channel variable, this is effectively what you’re doing.

The examples we’ve used to show you how conditional branching works are not invalid. We’ve first initialized the variable and can clearly see that the channel variable we’re using in our comparison has been set, so we’re safe. But what if you’re not always so sure?

In Asterisk, strings do not need to be double- or single-quoted like in many programming languages. In fact, if you use a double or single quote, it is a literal construct in the string. If we look at the following snippets of an extension...

 same =&gt; n,Set\(TEST\_1=foo\)

 same =&gt; n,Set\(TEST\_2='foo'\)

 same =&gt; n,NoOp\(Are TEST\_1 and TEST\_2 equiv? $\[${TEST\_1} = ${TEST\_2}\]\)

...we need to note that the value returned by our comparison in the NoOp\(\) will not be a value of 1 \(values match; or true\) the return value will be 0 \(values do not match; or false\).

We can use this to our advantage when performing comparisons by wrapping our channel variables in single or double quotes. By doing this we make sure even when the channel variable might not be set, our comparison will be valid syntax.

In the following example, we would get an error:

exten =&gt; 212,1,NoOp\(\)

 same =&gt; n,GotoIf\($\[ ${TEST} != valid \]?error\_handling\)

 same =&gt; n,Hangup\(\) ; We're getting an error and ending up here

 same =&gt; n\(error\_handling\),Playback\(goodbye\)

 same =&gt; n,Hangup\(\)

However, we can circumvent this by wrapping what we’re comparing in extra characters \(in this case quotes\). The same example, but made valid:

exten =&gt; 213,1,NoOp\(\)

 same =&gt; n,GotoIf\($\[ "${TEST}" != "valid" \]?error\_handling\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(error\_handling\),Playback\(goodbye\)

 same =&gt; n,Hangup\(\)

Even if ${TEST} hasn’t been set \(in other words it does not exist and therefore has no value\), we’re still doing a comparison of something:

$\["" != "valid"\]

If you get into the habit of recognizing these situations and using the wrapping and prefixing techniques we’ve outlined, you’ll write much safer dialplans.

Note again that the quote character doesn’t have any special meaning here. We used it because it’s a logical character for this purpose. The following works too:

 same =&gt; n,GotoIf\($\[\_${TEST}\_ != \_valid\_\]?error\_handling\)

;OR

 same =&gt; n,GotoIf\($\[AAAAA${TEST}AAAAA != AAAAAvalidAAAAA\]?error\_handling\)

Not all characters will work, as some may have other meanings to Asterisk and cause problems. Stay with the quote character and you should be fine.

The classic example of conditional branching is affectionately known as the “psycho-ex” logic. If the caller ID number of the incoming call matches the phone number of somebody you never want to talk to again, Asterisk gives a different message than it ordinarily would to any other caller. While somewhat simple and primitive, it’s a good example for learning about conditional branching within the Asterisk dialplan.

This example uses the CALLERID\(\) function, which allows us to retrieve the caller ID information on the inbound call. Let’s assume for the sake of this example that the victim’s phone number is 888-555-1212:[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406809256)

exten =&gt; 214,1,NoOp\(CALLERID\(num\): ${CALLERID\(num\)} CALLERID\(name\): ${CALLERID\(name\)}\)

 same =&gt; n,GotoIf\($\[ ${CALLERID\(num\)} = 8885551212 \]?reject:allow\)

 same =&gt; n\(allow\),Dial\(${UserA\_DeskPhone}\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(reject\),Playback\(abandon-all-hope\)

 same =&gt; n,Hangup\(\)

In priority 1, we call the GotoIf\(\) application. It tells Asterisk to go to priority label reject if the caller ID number matches 8885551212, and otherwise to go to priority label allow \(we could have simply omitted the label name, causing the GotoIf\(\) to fall through\).[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406801064) If the caller ID number matches, control of the call goes to priority label reject, which plays back a subtle hint to the undesired caller. Otherwise, the call attempts to dial the recipient on the channel referenced by the UserA\_DeskPhone global variable.

### Time-Based Conditional Branching with GotoIfTime\(\)

Another way to use conditional branching in your dialplan is with the GotoIfTime\(\) application. Whereas GotoIf\(\) evaluates an expression to decide what to do, GotoIfTime\(\) looks at the current system time and uses that to decide whether or not to follow a different branch in the dialplan.

The most obvious use of this application is to give your callers a different greeting before and after normal business hours.

The syntax for the GotoIfTime\(\) application looks like this:

GotoIfTime\(times,days\_of\_week,days\_of\_month,months?label\)

In short, GotoIfTime\(\) sends the call to the specified label if the current date and time match the criteria specified by times, days\_of\_week, days\_of\_month, and months. Let’s look at each argument in more detail:

times

This is a list of one or more time ranges, in a 24-hour format. As an example, 9:00 A.M. through 5:00 P.M. would be specified as 09:00-17:00. The day starts at 0:00 and ends at 23:59.

**Note**

It is worth noting that times will properly wrap around. So, if you wish to specify the times your office is closed, you might write 18:00-9:00 in the times parameter, and it will perform as expected. Note that this technique works as well for the other components of GotoIfTime\(\). For example, you can write sat-sun to specify the weekend days.

days\_of\_week

This is a list of one or more days of the week. The days should be specified as mon, tue, wed, thu, fri, sat, and/or sun. Monday through Friday would be expressed as mon-fri. Tuesday and Thursday would be expressed as tue&thu.

**Note**

Note that you can specify a combination of ranges and single days, as in: sun-mon&wed&fri-sat, or, more simply: wed&fri-mon.

days\_of\_month

This is a list of the numerical days of the month. Days are specified by the numbers 1 through 31. The 7th through the 12th would be expressed as 7-12, and the 15th and 30th of the month would be written as 15&30. This can be useful for holidays, which often fall on the same day of the month, but not typically on the same day of the week.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406763720)

months

This is a list of one or more months of the year. The months should be written as jan-apr for a range, and separated with ampersands when wanting to include nonsequential months, such as jan&mar&jun. You can also combine them like so: jan-apr&jun&oct-dec.

If you wish to match on all possible values for any of these arguments, simply put an \* in for that argument.

The label argument can be any of the following:

* A priority label within the same extension, such as time\_has\_passed
* An extension and a priority within the same context, such as 123,time\_has\_passed
* A context, extension, and priority, such as incoming,123,time\_has\_passed

Now that we’ve covered the syntax, let’s look at a couple of examples. The following example would match from 9:00 A.M. to 5:59 P.M., on Monday through Friday, on any day of the month, in any month of the year:

exten =&gt; s,1,NoOp\(\)

 same =&gt; n,GotoIfTime\(09:00-17:59,mon-fri,\*,\*?open,s,1\)

If the caller calls during these hours, the call will be sent to the first priority of the start extension in the context named open. If the call is made outside of the specified times, it will simply carry on with the next priority of the current extension. We’re going to add a new context named \[closed\] right after the pattern match example 55512XX, and modify the \[TestMenu\] context we built in [Chapter 6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics) to handle our new time condition.

exten =&gt; \_55512XX,1,Answer\(\)

 same =&gt; n,Playback\(tt-monkeys\)

 same =&gt; n,Hangup\(\)

exten =&gt; \*98,1,NoOp\(Access voicemail retrieval.\)

 same =&gt; n,VoiceMailMain\(\)

\[closed\]

exten =&gt; start,1,Noop\(after hours handler\)

 same =&gt; n,Playback\(go-away2\)

 same =&gt; n,Hangup\(\)

\[TestMenu\]

exten =&gt; start,1,Noop\(main autoattendant\)

 same =&gt; n,GotoIfTime\(16:59-08:00,mon-fri,\*,\*?closed,start,1\)

 same =&gt; n,GotoIfTime\(11:59-09:00,sat,\*,\*?closed,start,1\)

 same =&gt; n,GotoIfTime\(00:00-23:59,sun,\*,\*?closed,start,1\)

 same =&gt; n,Background\(enter-ext-of-person\)

 same =&gt; n,WaitExten\(5\)

exten =&gt; 1,1,Dial\(${UserA\_DeskPhone},10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

## GoSub

The GoSub\(\) dialplan application allows you to send a call off to a separate section of the dialplan, make something useful happen, and then return the call to the point in the dialplan where it came from. You can pass arguments to GoSub\(\), and also receive a return code back from it. This cranks up the functionality of your dialplan quite a bit.

**Note**

Subroutines are a critical skill in any programming language, and no less so in an Asterisk dialplan. For those new to programming, a subroutine allows you to create a block of generic code that can be reused by different parts of the dialplan to avoid repetition. Think of it like a template in a word processing document, or a blank form, and you’ve got the general idea. Once you see them in operation, it should become clear how useful they can be.

### Defining Subroutines

There are no special naming requirements when using GoSub\(\) in the dialplan. In fact, you can use GoSub\(\) within the same context and extension if you want to. In most cases, however, your subroutines should be written in separate contexts: one context for each subroutine. When creating the context, we like to prepend the name with sub so we know the context is called from the GoSub\(\) application.

Let’s explore an obvious example of where a subroutine would be useful.

As you might have noticed when we were building our sample dialplan for the users we have created, the dialplan logic for each user can require several lines of code.

\[sets\]

exten =&gt; 100,1,Dial\(${UserA\_DeskPhone},12\)

 same =&gt; n,Voicemail\(100@default\)

 same =&gt; n,GotoIf\($\["${DIALSTATUS}" = "BUSY"\]?busy:unavail\)

 same =&gt; n\(unavail\),VoiceMail\(100@default,u\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(busy\),VoiceMail\(100@default,b\)

 same =&gt; n,Hangup\(\)

exten =&gt; 101,1,Dial\(${UserA\_SoftPhone}\)

 same =&gt; n,GotoIf\($\["${DIALSTATUS}" = "BUSY"\]?busy:unavail\)

 same =&gt; n\(unavail\),VoiceMail\(101@default,u\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(busy\),VoiceMail\(101@default,b\)

 same =&gt; n,Hangup\(\)

exten =&gt; 102,1,Dial\(${UserB\_DeskPhone},10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; 103,1,Dial\(${UserB\_SoftPhone}\)

 same =&gt; n,Hangup\(\)

We’ve only provided two users with actual, working voicemail, and we’ve only defined four phones as extensions, and yet we’ve already got a mess of repetitive code, which is only going to get more and more difficult to maintain and expand. This will quickly become unmanageable if we don’t find a better way.

Let’s write a subroutine to handle dialing our users. Add the following to the very bottom of your dialplan:

; SUBROUTINES

\[subDialUser\]

exten =&gt; \_\[0-9\].,1,Noop\(Dial extension ${EXTEN},channel: ${ARG1}, mailbox: ${ARG2}\)

 same =&gt; n,Noop\(mboxcontext: ${ARG3}, timeout ${ARG4}\)

 same =&gt; n,Dial\(${ARG1},${ARG4}\)

 same =&gt; n,GotoIf\($\["${DIALSTATUS}" = "BUSY"\]?busy:unavail\)

 same =&gt; n\(unavail\),VoiceMail\(${ARG2}@${ARG3},u\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(busy\),VoiceMail\(${ARG2}@${ARG3},b\)

 same =&gt; n,Hangup\(\)

Now, modify the top of your dialplan as follows:

\[OLD\_sets\] ; what was \[sets\] is now \[OLD\_sets\] \(call it whatever, so long as name changes\)

exten =&gt; 100,1,Dial\(${UserA\_DeskPhone},12\)

 same =&gt; n,Voicemail\(100@default\)

 same =&gt; n,GotoIf\($\["${DIALSTATUS}" = "BUSY"\]?busy:unavail\)

;\(etc\)

We’ve renamed our \[sets\] context, which of course breaks our dialplan since our phones enter the dialplan there. So, we’re going to reinsert it a little farther down:

exten =&gt; 103,1,Dial\(${UserB\_SoftPhone}\)

 same =&gt; n,Hangup\(\)

\[sets\]

exten =&gt; 110,1,Dial\(${UserA\_DeskPhone}&${UserA\_SoftPhone}&${UserB\_SoftPhone}\)

 same =&gt; n,Hangup\(\)

;\(etc\)

OK, so now we’ve got our \[sets\] context working again, and also this \[OLD\_sets\] context that’s got our old, orphaned code. How do we dial our telephones? How does this subroutine we just wrote help us?

exten =&gt; 103,1,Dial\(${UserB\_SoftPhone}\)

 same =&gt; n,Hangup\(\)

\[sets\]

;subDialUser args:

; - ARG1 Channel\(s\) to dial

; - ARG2 Mailbox

; - ARG3 Mailbox Context

; - ARG4 Timeout

exten =&gt; 100,1,Gosub\(subDialUser,${EXTEN},1\(${UserA\_DeskPhone},${EXTEN},default,12\)\)

exten =&gt; 101,1,Gosub\(subDialUser,${EXTEN},1\(${UserA\_SoftPhone},${EXTEN},default,3\)\)

exten =&gt; 102,1,Gosub\(subDialUser,${EXTEN},1\(${UserB\_DeskPhone},${EXTEN},default,6\)\)

exten =&gt; 103,1,Gosub\(subDialUser,${EXTEN},1\(${UserB\_SoftPhone},${EXTEN},default,24\)\)

exten =&gt; 110,1,Dial\(${UserA\_DeskPhone}&${UserA\_SoftPhone}&${UserB\_SoftPhone}\)

 same =&gt; n,Hangup\(\)

Plug that in, reload your dialplan, and make some test calls. Play with the parameters and see what changes. Add some mailboxes to your database and see what that does. If you’re inspired, write a new subDialUserNEW subroutine and see what you can come up with. At this point you can also delete all the code in the \[OLD\_sets\] context, since it’s now abandoned, but you can also leave it there as it does no harm.

Now, you can add hundreds of extensions, and each one will only use one line of the dialplan.

Whenever you find yourself writing duplicate dialplan code somewhere, stop. It’s very likely that it’s time to write a subroutine.

### Returning from a Subroutine

The GoSub\(\) dialplan application does not return automatically once it is done executing. If you’re done with the call, you can of course use Hangup\(\); however, if you don’t want to disconnect, but rather need to return the call from where it came, you can use the Return\(\) application.

Since you can nest subroutines within subroutines and also execute them one after another, as you get into more complex subroutines you will find this an essential capability.

## Local Channels

Local channels are a method of executing other areas of the dialplan from the Dial\(\) application \(as opposed to sending the call out a channel\). Think of them as subroutines you can call from within Dial\(\).

They may seem like a bit of a strange concept when you first start using them, but believe us when we tell you they can be the answer to a problem you can’t figure out any other way. You will almost certainly want to make use of them when you start writing advanced dialplans. The best way to illustrate the use of local channels is through an example. Let’s suppose we have a situation where we need to ring multiple people, but we need to provide delays of different lengths before dialing each of the members. The use of local channels is the solution to the problem.

With the Dial\(\) application, you can certainly ring multiple endpoints \(see extension 110 in your dialplan for an example of this\), but all three channels will ring at the same time, and for the same length of time.

exten =&gt; 110,1,Dial\(${UserA\_DeskPhone}&${UserA\_SoftPhone}&${UserB\_SoftPhone}\)

 same =&gt; n,Hangup\(\)

However, let’s say we want to introduce some delays prior to ringing a user, and also stop ringing locations at different times. Using local channels gives us independent control over each of the channels we want to dial, so we can introduce delays and control the period of time for which each channel rings independently.

Let’s say we have a small company, where the receptionist is primarily responsible for the incoming calls, but there are also two team members who are tasked with backing up reception, and finally the owner wants to help out if need be too.

These are the requirements:

* The reception phone should ring right away, and keep ringing and not stop until answered.
* The team member phones shouldn’t ring for the first 9 seconds, at which point they can ring until answered.
* The owner’s phone should only ring if the call has gone on for 12 seconds with no answer. Also, we’re pretending it’s a cell phone, and thus should stop ringing 18 seconds later so that the call is not answered by the cell phone voicemail.

We’ll use our existing configured channels to play the various roles. If you have any way to do so, please try to have them all registered somewhere so they can all ring when called. It’ll give you a much better idea of what’s going on when testing.[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406696296)

This is a great time for a subroutine:

\[subDialDelay\]

exten =&gt; \_\[a-zA-Z0-9\].,1,Noop\(channel ${ARG1}, pre-delay ${ARG2}, timeout ${ARG3}\)

; same =&gt; n,Progress\(\) ; Optional; Signals back that the call is proceeding

 same =&gt; n,Wait\(${ARG2}\) ; how long to wait before dialing

 same =&gt; n,Dial\(${ARG1},${ARG3}\) ; timeout can be blank \(infinite\)

 same =&gt; n,Hangup\(\)

**Note**

You already have a subroutine at the bottom of the file. Add this one down there too so all your subroutines end up grouped together.

Now we want a context in which we’ll build out the extensions to be used by the local channel:

;LOCAL CHANNELS

\[localDialDelay\]

exten =&gt; receptionist,1,Gosub\(subDialDelay,${EXTEN},1\(${UserA\_DeskPhone},0,600\)\)

exten =&gt; team\_one,1,Gosub\(subDialDelay,${EXTEN},1\(${UserA\_SoftPhone},9,600\)\)

exten =&gt; team\_two,1,Gosub\(subDialDelay,${EXTEN},1\(${UserB\_DeskPhone},9,600\)\)

exten =&gt; owner,1,Gosub\(subDialDelay,${EXTEN},1\(${UserB\_SoftPhone},12,18\)\)

**Note**

Even though the destination for a local channel is really just dialplan—the same as you might jump to with a Goto\(\)—these constructs tend to be very special-purpose, and fit into the dialplan better in their own area, down with the subroutines. That’s why we named the context with the prefix local. It’s not required, but makes things easier to make sense of.

Now we stitch it all together in our \[sets\] context.

First, let’s provide a way to dial each local channel individually, so we can sanity check each one to be sure it’s doing what it should.

exten =&gt; 103,1,Gosub\(subDialUser,${EXTEN},1\(${UserB\_SoftPhone},${EXTEN},default,24\)\)

; These are for testing individually before we put them together

exten =&gt; 104,1,Dial\(Local/receptionist@localDialDelay\)

exten =&gt; 105,1,Dial\(Local/team\_one@localDialDelay\)

exten =&gt; 106,1,Dial\(Local/team\_two@localDialDelay\)

exten =&gt; 107,1,Dial\(Local/owner@localDialDelay\)

Finally, let’s deliver the finished product.

exten =&gt; 107,1,Dial\(Local/owner@localDialDelay\)

;We're going to assign some variables in order to

;keep the dial string easier to read

exten =&gt; 108,1,Noop\(DialDelay\)

 same =&gt; n,Set\(Recpn=Local/receptionist@localDialDelay\)

 same =&gt; n,Set\(Team1=Local/team\_one@localDialDelay\)

 same =&gt; n,Set\(Team2=Local/team\_two@localDialDelay\)

 same =&gt; n,Set\(Boss=Local/owner@localDialDelay\)

 same =&gt; n,Dial\(${Recpn}&${Team1}&${Team2}&${Boss},600\)

You really need to register a few phones and try this out, to see it all come together.

The solution we have created here is perfect for learning about local channels, but it has a few problems that need to be understood if you ever want to put it into production:

* Even though we have set a dial timeout, you will find that SIP endpoints have minds of their own. It’s not uncommon for a SIP endpoint to have its own ideas about timeout. So, you might set it to ring for 600 seconds, and wonder why it drops the call after a minute or so. You could spend hours troubleshooting your dialplan, only to discover the problem was a setting at the other end. Test each piece of the solution before you glue them all together.
* Cell phones have their own voicemail, and if that answers the call, Asterisk will connect the call to that “answered” channel. One way around this is to hang up before that happens, and then call immediately back. It’s ugly though, and not recommended.
* Cell phones will often go immediately to a voicemail if they’re out of range or turned off. That counts as an answer as far as Asterisk is concerned. This solution does not handle that.
* Call setup to a cell phone \(i.e., the time between when you dial and when it starts ringing\) typically takes a dozen seconds or so.
* Remember that a softphone on a cell phone is not at all the same as a phone call to that cell phone. One is a SIP connection, the other is a PSTN call. \(You can actually ring both at the same time if you want, but that’s not necessarily a good idea.\)
* Some types of smartphones will give priority to incoming GSM calls. If you are on a call on the softphone, and somebody calls your cell number, the softphone may get put on hold. Different phones handle this differently.
* We haven’t really handled overflow here. What happens if nobody answers? It doesn’t matter in the lab, but you can be sure it’ll matter in a production environment.
* Dial\(\) expects ringing back from the destination. If all of your local channels have a Wait\(\) delay, the caller will hear silence until something indicates ringing. You can fix this by having Dial\(\) fake the ringing with the 'r' option, or by adding a dummy local channel that just returns ringing.

**Note**

If you check the sample dialplan, we’ve added a solution to the silence problem on delayed local channels

That’s it. Local channels: build them piece-by-piece and you’ll be delivering a powerful dialplan in no time.

They’re incredibly useful when building complex queueing applications as well.

## Using the Asterisk Database

Asterisk provides a simple mechanism for storing data called the Asterisk database \(AstDB\). This is not an external relational database, but simply an SQLite-based backend for storing simple key/value pairs.

The Asterisk database stores its data in groupings called families, with values identified by keys. Within a family, a key may be used only once. For example, if we had a family called test, we could store only one value with a key called count. Each stored value must be associated with a family.

### Storing Data in the AstDB

To store a new value in the Asterisk database, we use the Set\(\) application with the DB\(\) function. For example, to assign the count key in the test family with the value of 1, we would write the following:

exten =&gt; 216,1,NoOp\(\)

 same =&gt; n,Set\(DB\(testkey/count\)=1\)

Make a test call to 216 to set the value. Note that if a key named count already exists in the test family, its value will be overwritten with the new value \(in this case, the value is hardcoded so obviously will get overwritten with the same value, but later we’ll see how we can change the value, and have that stored\).

You can also store values from the Asterisk command line, by running the command database put family key value. For our example, you would type database put test count 1.

So, while we’re at it, let’s also plug a value into the database from the console:

\*CLI&gt; database put somekey somevalue 42

Let’s query the database from the console to see what values are in there:

\*CLI&gt; database show

If all is well, you should see output similar to the following:

/pbx/UUID : d562019a-d2c4-4b88-bcd9-602b3b46fe07

/somekey/count : 1

/somekey/somevalue : 42

/testkey/count : 1

4 results found.

localhost\*CLI&gt;

### Retrieving Data from the AstDB

To retrieve a value from the Asterisk database and assign it to a variable, we will again use the Set\(\) application and the DB\(\) function. Let’s retrieve the value of somevalue \(from the somekey family\), assign it to a variable called THE\_ANSWER, and then speak the value to the caller:

exten =&gt; 217,1,NoOp\(\)

 same =&gt; n,Set\(THE\_ANSWER=${DB\(somekey/somevalue\)}\)

 same =&gt; n,Answer\(\)

 same =&gt; n,SayNumber\(${THE\_ANSWER}\)

You may also check the value of a given key from the Asterisk command line by running the command database get family key. To view the entire contents of the AstDB, use the database show command.

### Deleting Data from the AstDB

There are two ways to delete data from the Asterisk database. To delete a key, you can use the DB\_DELETE\(\) application. It takes the path to the key as its arguments, like this:

; deletes the key and returns its value in one step

exten =&gt; 218,1,Verbose\(0, We just blew away ${DB\_DELETE\(somekey/somevalue\)}\)

You can also delete an entire key family by using the DBdeltree\(\) application. The DBdeltree\(\) application takes a single argument: the name of the key family to delete. To delete the entire test family, do the following:

exten =&gt; 219,1,DBdeltree\(somekey\)

To delete keys and key families from the AstDB via the command-line interface, use the database del key and database deltree family commands, respectively.

If you call extension 217 now, you will see that there is nothing said, because nothing is returned by the database. You can also run database show from the CLI, and note that that family and key have been removed.

### Using the AstDB in the Dialplan

There are an infinite number of ways to use the Asterisk database in a dialplan. To introduce the AstDB, we’ll look at two simple examples. The first is a simple counting example to show that the Asterisk database is persistent \(it even survives system reboots\). In the second example, we’ll use the BLACKLIST\(\) function to evaluate whether or not a number is on the blacklist and should be blocked.

To begin the counting example, let’s first retrieve a number \(the value of the count key\) from the database and assign it to a variable named COUNT. If the key doesn’t exist, DB\(\) will return NULL \(no value\). Therefore, we can use the ISNULL\(\) function to verify whether or not a value was returned. If not, we will initialize the AstDB with the Set\(\) application, where we will set the value in the database to 1. This will only happen if the database entry does not exist:

exten =&gt; 220,1,NoOp\(\)

 same =&gt; n,Set\(COUNT=${DB\(test/count\)}\) ; retrieve current value in database

 same =&gt; n,GotoIf\($\[${ISNULL\(${COUNT}\)}\]?firstcount:saycount\) ; is there a value?

 same =&gt; n\(firstcount\),Set\(DB\(test/count\)=1\) ; set the value to 1

 same =&gt; n,Goto\(saycount\)

 same =&gt; n\(saycount\),NoOp\(\)

 same =&gt; n,Answer

 same =&gt; n,SayNumber\(${COUNT}\)

 same =&gt; n,Goto\(increment\) ; not reqd but a good habit

 same =&gt; n\(increment\),Set\(COUNT=$\[${COUNT} + 1\]\) ; increment by one

 same =&gt; n,Set\(DB\(test/count\)=${COUNT}\) ; and assign new value to database

 same =&gt; n,Goto\(saycount\) ; loop back and say it again

Test this out. Listen to it count for a while, and then hang up. When you dial this extension again, it will continue counting from where it left off. The value stored in the database will be persistent, even across a restart of Asterisk.

In the early days of Asterisk, the built-in database was essential. Today, however, it’s not as commonly used. It’s probably good for setting a few semaphores here and there, but for the most part, if you want to store data, use one of the relational database backends \(we discuss relational database integration in later chapters\).

## Handy Asterisk Features

Now that we’ve gone over some more of the basics, let’s look at a few popular functions that have been incorporated into Asterisk.

### Conferencing with ConfBridge\(\)

The ConfBridge\(\) application allows multiple callers to converse together, as if they were all in the same physical location. Some of the main features include:

* The ability to create password-protected conferences
* Conference administration \(mute conference, lock conference, or kick off participants\)
* The option of muting all but one participant \(useful for company announcements, broadcasts, etc.\)
* Static or dynamic conference creation
* High-definition audio that can be mixed at sample rates ranging from 8 kHz to 96 kHz
* Video capabilities, including the addition of dynamically switching video feeds based on loudest talker
* Dynamically controlled menu system for both conference administrators and users
* Additional options available in the confbridge.conf configuration file

In this chapter we are focused on the dialplan, so we’re only going to demonstrate a basic audio conference bridge:

$ sudo -u asterisk vim /etc/asterisk/confbridge.conf

\[general\]

\[default\_user\]

type=user

\[default\_bridge\]

type=bridge

After building the confbridge.conf file, we need to load the app\_confbridge.so module. This can be done at the Asterisk console:

\*CLI&gt; module load app\_confbridge.so

With the module loaded, we can build a simple dialplan to access our conference bridge:

exten =&gt; 221,1,NoOp\(\)

 same =&gt; n,ConfBridge\(${EXTEN}\)

This is just the tip of the iceberg for conferencing. We’ve got the base configuration done, but there is much more functionality to be configured. We’ll cover it in a little more detail in [Chapter 11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22asterisk-SysAdmin).

## Handy Dialplan Functions

We discussed functions earlier in this chapter, but there’s more to say. There are currently around 150 dialplan functions provided by the Asterisk dialplan. Here is a small, curated list of a few worth experimenting with.

### CALLERID\(\)

CALLERID\(\) supports many different datatypes, but you’ll find that you’ll typically use one of name or num.

exten =&gt; 222,1,Noop\(CALLERID function\)

 same =&gt; n,Noop\(CALLERID currently ${CALLERID\(all\)}\)

 same =&gt; n,Set\(CALLERID\(num\)=4169671111\)

 same =&gt; n,Noop\(CALLERID now ${CALLERID\(all\)}\)

 same =&gt; n,Set\(CALLERID\(name\)="Somename"\)

 same =&gt; n,Noop\(CALLERID now ${CALLERID\(all\)}\)

 same =&gt; n,Hangup\(\)

Don’t worry about the rest of them. If you need ’em, you’ll know what they are or why you want to use them.

### CHANNEL\(\)

CHANNEL\(\) allows you to interact with an absolute boatload of data relating to the channel. Some items allow you to modify them, while others will only be useful for reference \(for example, peerip will allow you to read, but not change, the IP address of the peer\). There are also channel variables that only work with certain channel types \(for example, pjsip items can of course only be used on PJSIP channels\).

exten =&gt; 223,1,Noop\(CHANNEL function\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Noop\(CHANNEL\(name\) is ${CHANNEL\(name\)}\)

 same =&gt; n,Noop\(CHANNEL\(musicclass\) is ${CHANNEL\(musicclass\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,all\_jitter\) is ${CHANNEL\(rtcp,all\_jitter\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,all\_loss\) is ${CHANNEL\(rtcp,all\_loss\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,all\_rtt\) is ${CHANNEL\(rtcp,all\_rtt\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,txcount\) is ${CHANNEL\(rtcp,txcount\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,rxcount\) is ${CHANNEL\(rtcp,rxcount\)}\)

 same =&gt; n,Noop\(CHANNEL\(pjsip,local\_uri\) is ${CHANNEL\(pjsip,local\_uri\)}\)

 same =&gt; n,Noop\(CHANNEL\(pjsip,remote\_uri\) is ${CHANNEL\(pjsip,remote\_uri\)}\)

 same =&gt; n,Noop\(CHANNEL\(pjsip,request\_uri\) is ${CHANNEL\(pjsip,request\_uri\)}\)

 same =&gt; n,Noop\(CHANNEL\(pjsip,local\_tag\) is ${CHANNEL\(pjsip,local\_tag\)}\)

### CURL\(\)

CURL\(\) is a simple yet powerful function that provides a one-liner method for resolving URLs, which in many cases is all you need for a basic interaction with an external web service.

exten =&gt; 224,1,Noop\(CURL function\)

 same =&gt; n,Set\(ExternalIP=${CURL\(http://whatismyip.akamai.com\)}\)

 same =&gt; n,Noop\(The external IP address is ${ExternalIP}\)

If you need a more complex interaction with an external service, it could be that you are going to want an AGI program of some sort. Still, you can embed a ton of data in a URL, and for simplicity, CURL\(\) is hard to beat.

### CUT\(\)

If you need to slice-and-dice your variables, you’ll find the CUT\(\) function essential. The form is simple:

CUT\(varname,char-delim,range-spec\)

It can be visually tricky, as the delimiter character can be difficult to see nested in between two commas \(for example, if the delimiter was a dot/decimal/period\). Let’s expand on the previous example to see what it’s good for \(and give you a visual example of how the delimiter can get lost in the syntax\).

exten =&gt; 225,1,Noop\(CUT function\)

 same =&gt; n,Set\(ExternalIP=${CURL\(http://whatismyip.akamai.com\)}\)

 same =&gt; n,Noop\(The external IP address is ${ExternalIP}\)

 same =&gt; n,Answer\(\)

 same =&gt; n,SayDigits\(=${CUT\(ExternalIP,.,1\)}\)

 same =&gt; n,Playback\(letters/dot\)

 same =&gt; n,SayDigits\(=${CUT\(ExternalIP,.,2\)}\)

 same =&gt; n,Playback\(letters/dot\)

 same =&gt; n,SayDigits\(=${CUT\(ExternalIP,.,3\)}\)

 same =&gt; n,Playback\(letters/dot\)

 same =&gt; n,SayDigits\(=${CUT\(ExternalIP,.,4\)}\)

**Note**

Note that you call the CUT\(\) function with the braces ${CUT\(\)}, but the variable being referenced inside CUT\(\) is defined without the braces. This is because we are naming the variable, not asking for its contents \(CUT\(\) will deal with the contents, so we just need to name the variable it will be slicing and dicing, and it will dive into what is stored there\).

### IF\(\) and STRFTIME\(\)

The combination of IF\(\) and STRFTIME\(\) is a powerful construct, and you will find it an essential part of your dialplan logic:

exten =&gt; 226,1,Noop\(IF\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Playback\(${IF\($\[$\[${STRFTIME\(,,%S\)} % 2\] = 1\]?hear-odd-noise:good-evening\)}\)

Wait...what?[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406549320)

Let’s break this down \(we’re going to indent the code in an odd manner in order to show the progression of the nested functions and operators\):

exten =&gt; 227,1,Noop\(IF\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Wait\(.5\)

 same =&gt; n,Wait\(.5\)

 same =&gt; n,Noop\(${STRFTIME\(,,%S\)}\) ; current time - just seconds

 same =&gt; n,Noop\($\[ ${STRFTIME\(,,%S\)} % 2 \]\) ; divide by 2 - return remainder

 same =&gt; n,Noop\(${IF\($\[ $\[ ${STRFTIME\(,,%S\)} % 2 \] = 1 \]?odd:even\)}\)

same =&gt; n,Playback\(${IF\($\[ $\[ ${STRFTIME\(,,%S\)} % 2 \] = 1 \]?hear-odd-noise:good-evening\)}\)

The IF\(\) function allows us to pass logic to the Playback\(\) application. We’re effectively saying, “If it’s true that the time, in seconds, is odd, play the hear-odd-noise prompt, otherwise, play the good-evening prompt.”

If we line up the code in a more typical fashion, it looks like this \(note that some of the optional spaces have also been removed\):

exten =&gt; 228,1,Noop\(IF\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Wait\(.5\)

 same =&gt; n,Noop\(${STRFTIME\(,,%S\)}\)

 same =&gt; n,Noop\($\[${STRFTIME\(,,%S\)} % 2\]\)

 same =&gt; n,Noop\(${IF\($\[$\[${STRFTIME\(,,%S\)} % 2 \] = 1\]?odd:even\)}\)

 same =&gt; n,Playback\(${IF\($\[$\[${STRFTIME\(,,%S\)} % 2 \] = 1\]?hear-odd-noise:good-evening\)}\)

The final line is very difficult to read unless you know how we got there, but it demonstrates the power of nesting.

At first these constructs may seem difficult to write, so break them down and perform them line by line, and eventually they’ll get easier \(and your dialplan will subsequently become more powerful\). Play with them.

### LEN\(\)

Being able to return the length of something with the LEN\(\) function can be very handy.

exten =&gt; 229,1,Noop\(LEN\)

 same =&gt; n,Set\(LengthyString=${RAND\(1,2000\)}\)

 same =&gt; n,Noop\(${LEN\(${LengthyString}\)}\)

 same =&gt; n,Noop\(${IF\( $\[ ${LEN\(${LengthyString}\)} &lt;= 3 \]?tooshort:youcanride\)}\)

### REGEX\(\)

Yes, you can use regular expressions within Asterisk. This is a somewhat advanced topic, not because REGEX\(\) is a complicated function in itself, but because regular expressions are a study in themselves.

Check out [http://www.regular-expressions.info/](http://www.regular-expressions.info/) for more info, or grab a copy of O’Reilly’s Mastering Regular Expressions by Jeffrey E. F. Friedl.

Get used to using other functions in Asterisk, get some experience with regular expressions, and then give REGEX\(\) a try.

### STRFTIME\(\)

We just saw the STRFTIME\(\) function in our IF\(\) example. It allows you to return a time in various formats. In general, you want the input to be empty \(which defaults to the current time\). You can also give this function a specific Unix epoch string and it’ll work from that.

exten =&gt; 230,1,Noop\(STRFTIME\)

 same =&gt; n,Noop\(${STRFTIME\(,,%S\)}\) ; we've seen this before

 same =&gt; n,Noop\(${STRFTIME\(,,%B\)}\) ; month

 same =&gt; n,Noop\(${STRFTIME\(,,%H\)}\) ; hour in 24hr format

 same =&gt; n,Noop\(${STRFTIME\(,,%m\)}\) ; month as a decimal

 same =&gt; n,Noop\(${STRFTIME\(,,%M\)}\) ; minute

 same =&gt; n,Noop\(${STRFTIME\(,,%Y\)}\) ; year - 4 digits

 same =&gt; n,Noop\(${STRFTIME\(,,%Y-%m-%d %H:%m:%S\)}\) ; string some together

## Conclusion

In this chapter, we’ve covered a few more of the many applications in the Asterisk dialplan, and hopefully we’ve given you some more tools that you can use to further experiment with creating your own dialplans. As with other chapters, we invite you to go back and reread any sections that require clarification.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-CHP-6-FN-1-marker) Remember that when you reference a variable you can call it by its name, but when you refer to a variable’s value, you have to use the dollar sign and brackets around its name.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-CHP-6-FN-2-marker) For more on regular expressions, grab a copy of the ultimate reference, Jeffrey E. F. Friedl’s [Mastering Regular Expressions](http://shop.oreilly.com/product/9780596528126.do) \(O’Reilly, 2006\), or visit [http://www.regular-expressions.info](http://www.regular-expressions.info/).

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-CHP-6-FN-3-marker) If you don’t know what a ^ has to do with regular expressions, you simply must read [Mastering Regular Expressions](http://shop.oreilly.com/product/9780596528126.do). It will change your life!

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406809256-marker) If you want to test this \(which you do\), you can pick one of your working lab devices, and in the asterisk database, under the ps\_endpoints table, set the callerid field to '8885551212'. Then you can make a call from it to 214 to see the block in action.

UPDATE asterisk.ps\_endpoints SET callerid='8885551212' WHERE id='&lt;endpoint you chose as the victim&gt;'

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406801064-marker) But we do it this way because it’s easier to read.

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406763720-marker) We have no idea how to implement Easter, but are open to suggestions.

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406696296-marker) Obsolete Android phones and tablets can be great for this.

[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406549320-marker) There is a C language function named STRFTIME\(\) that returns the current time as a formatted string. This works similarly to that. In fact, the format portion of the function takes the exact same syntax as the C function.

