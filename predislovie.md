# Предисловие

This is a book for anyone who uses Asterisk.

Asterisk is an open source, converged telephony platform, which is designed primarily to run on Linux. Asterisk combines more than 100 years of telephony knowledge into a robust suite of tightly integrated telecommunications applications. The power of Asterisk lies in its customizable nature, complemented by unmatched standards compliance. No other private branch exchange \(PBX\) can be deployed in so many creative ways.

Applications such as voicemail, hosted conferencing, call queuing and agents, music on hold, and call parking are all standard features built right into the software. Moreover, Asterisk can integrate with other business technologies in ways that closed, proprietary PBXs can scarcely dream of.

Asterisk can appear quite daunting and complex to a new user, which is why documentation is so important to its growth. Documentation lowers the barrier to entry and helps people contemplate the possibilities.

Produced with the generous support of O’Reilly Media, [_Asterisk:_ _The Definitive Guide_](http://shop.oreilly.com/product/0636920025894.do) is the fifth edition of what was formerly called [_Asterisk: The Future of Telephony_](http://shop.oreilly.com/product/9780596510480.do).

This book was written for, and by, members of the Asterisk community.

## Audience

This book is intended to be gentle toward those new to Asterisk, but we assume that you’re familiar with basic Linux administration, networking, and other IT disciplines. If not, we encourage you to explore the vast and wonderful library of books that O’Reilly publishes on these subjects. We also assume you’re fairly new to telecommunications \(both traditional switched telephony and the new world of Voice over IP\).

However, this book will also be useful for the more experienced Asterisk administrator. We ourselves use the book as a reference for features that we haven’t used for a while.

## Software

This book is focused on documenting Asterisk version 16; however, many of the conventions and much of the information in this book is version-agnostic. Linux is the operating system we have run and tested Asterisk on, and we have documented installation instructions for CentOS \(Red Hat Enterprise Linux, or RHEL\).

## Conventions Used in This Book

The following typographical conventions are used in this book:_Italic_

Indicates new terms, URLs, email addresses, filenames, file extensions, pathnames, directories, and package names, as well as Unix utilities, commands, modules, parameters, and arguments.`Constant width`

Used to display code samples, file contents, command-line interactions, database commands, library names, and options.**`Constant width bold`**

Indicates commands or other text that should be typed literally by the user. Also used for emphasis in code._`Constant width italic`_

Shows text that should be replaced with user-supplied values._`[ Keywords and other stuff ]`_

Indicates optional keywords and arguments._`{ choice-1 | choice-2 }`_

Signifies either _`choice-1`_ or _`choice-2`_.

**Tip**

This element signifies a tip or suggestion.

**Note**

This element signifies a general note.

**Caution**

This element indicates a warning or caution.

## O’Reilly Online Learning

**Note**

For almost 40 years, [O’Reilly Media](http://oreilly.com/) has provided technology and business training, knowledge, and insight to help companies succeed.

 Our unique network of experts and innovators share their knowledge and expertise through books, articles, conferences, and our online learning platform. O’Reilly’s online learning platform gives you on-demand access to live training courses, in-depth learning paths, interactive coding environments, and a vast collection of text and video from O’Reilly and 200+ other publishers. For more information, please visit [_http://oreilly.com_](http://www.oreilly.com/).

## How to Contact Us

Please address comments and questions concerning this book to the publisher:

* O’Reilly Media, Inc.
* 1005 Gravenstein Highway North
* Sebastopol, CA 95472
* 800-998-9938 \(in the United States or Canada\)
* 707-829-0515 \(international or local\)
* 707-829-0104 \(fax\)

We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at [_https://oreil.ly/asterisk\_tdg\_5E_](https://oreil.ly/asterisk_tdg_5E).

To comment or ask technical questions about this book, send email to [_bookquestions@oreilly.com_](mailto:bookquestions@oreilly.com).

For more information about our books, courses, conferences, and news, see our website at [_http://www.oreilly.com_](http://www.oreilly.com/).

Find us on Facebook: [_http://facebook.com/oreilly_](http://facebook.com/oreilly)

Follow us on Twitter: [_http://twitter.com/oreillymedia_](http://twitter.com/oreillymedia)

Watch us on YouTube: [_http://www.youtube.com/oreillymedia_](http://www.youtube.com/oreillymedia)

## Acknowledgments from Jim Van Meggelen

To David Duffett, thanks for the chapter on internationalization, which properly looks at this technology from a more global perspective.

Thanks to Leif Madsen, Jared Smith, and Russell Bryant, for your contributions to the previous editions of this book. It was fun flying solo, but I can’t deny I missed you guys!

Specific thanks to Matt Fredrickson and Matt Jordan of Digium, who generously shared their time and knowledge with me, and without whom I would have been lost. Thanks guys!

Thanks to my editor, Jeff Bleiel, for keeping me on track and helping me make important decisions about the content and pacing of the book.

Also thanks to the rest of the unsung heroes in O’Reilly’s production department. These are the folks that take a book and make it an _O’Reilly book_.

Thanks especially to Joyce Wilmot and Dan Jenkins, my technical review team, for taking the time to work through the book and provide essential feedback.

Thomas Cameron of RedHat generously shared his knowledge of Selinux with me, and helped to demystify a Linux component that is too often left disabled.

Everyone in the Asterisk community also needs to thank the late Jim Dixon for creating the first open source telephony hardware interfaces, starting the revolution, and giving his creations to the community at large.

Finally, and most importantly, thanks go to Mark Spencer, the original author of Asterisk and founder of Digium, for Asterisk, for [Pidgin](http://www.pidgin.im/), and for contributing his creations to the open source community. Asterisk is your legacy!

