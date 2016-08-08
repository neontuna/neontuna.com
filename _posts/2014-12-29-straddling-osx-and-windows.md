---
layout: post
title: Straddling OSX and Windows
date: 2014-12-29 11:45:57
summary: Another Windows guy jumps ship
category: blog
tags: hardware software
---

>According to stock photo sites in order to use an Apple product you must obsessively arrange your desk into perfect geometric patterns, like a serial killer.  I have not done this yet.

![my messy desk](/assets/IMG_1434_large.JPG)

For a while now I've been interested in web development.  I'll run through a tutorial here and there - [railstutorial.org][5] and [learn python the hard way][6] have been my favorites (yes I realize these are two different languages, I have commitment issues).  If you're primarily using Windows modern web development can be frustrating to learn.  Every tutorial starts out instructing you to use an OSX package manager and generally include some huge caveat for getting your development environment configured in Windows.  I had no desire to learn Microsoft's languages, such as ASP.NET, so I decided I should look into a Macbook.  Also that my Dell notebook has always been kind of a piece of junk and that I don't particularly like Windows 8/8.1 helped push me towards "switching".

I'm now typing this from my Macbook Pro and Thunderbolt display.  I haven't fully made the switch for my day job as a remote IT worker.  I'm still trying to suss out how I want my browsers setup, which softphone to use, and some other odds and ends.  Funny thing is, there isn't much I need Windows for in order to support our clients that mostly run Windows.  I will probably always keep around a Windows system ([or 12][server]) because other IT people don't respect you if you only have one computer.

This is sort of a compilation of things I've noticed and some tips for anyone coming from the Windows world.    I've arranged things with buying tips first and then good, bad, and miscellaneous observations.

#Buying Tips 

Besides the notebook I researched and bought a few things from Amazon. The [mStand laptop stand][1] which is way overpriced for a piece of bent aluminum but I guess it holds the notebook at eye level adequately.  I also picked up an [ethernet adapter/usb hub][2] from Anker which instantly became useful because I didn't have the display (with its NIC) initially.  I really like Anker's products, they're generally pretty inexpensive but well built.  I also grabbed an [Incase Neoprene Sleeve][3] that is apparently made for the 13" MacBook Pro but has a little too much extra space in it. I feel like it ought to fit the notebook snugly and not allow it to move at all inside the sleeve.  Finally, I purchased the [Thunderbolt Display][4] from Amazon as well.  I was going to purchase a Magsafe>Magsafe 2 power adapter as the Thunderbolt display's charging cable isn't compatible with the latest Macbooks but then I found out that these are now being included with the displays.

For the notebook itself I found a really handy [price guide][7] from Apple Insider and ended up picking up the system from Adorama with 3 years of AppleCare bundled.   They threw in an Apple TV with the promo code from Apple Insider but checking the guide now that has changed to a mighty mouse.  I also checked the [Buyer's Guide][8] from Mac Rumors, just incase a new model was about to drop.

Initially I was not going to buy the Thunderbolt display but one issue with living in Apple's world is you have to play by their rules.  There is no official dock for the Macbook, the Thunderbolt Display is the closest thing to it.  My other options were Thunderbolt hubs which are several hundred dollars and may or may not have worked with my existing monitor which uses DVI-D.  So I reluctantly decided to get the Thunderbolt display but I feel like Apple could make a lot of people happy if they'd just release a dock.   I use a simple KVM to switch keyboard and mouse between the notebook and my PC tower, which is exclusively connected to my old display.  I thought I would need an Apple keyboard but really its no big deal.  The keys are in the same position except with the Windows key acting as the Command key.

#The Good

I occasionally listen to the [Mac Power Users podcast][mpu], which was kind of funny to do before I actually owned an Apple computer.  But from it I gleaned a lot of information on OSX applications I'd want to pick up.  So far [TextExpander ][te]has been my favorite.  If you're not familiar with text expansion, you basically type a shortcut and the application automatically fills in something for you.  For instance if I type ``;gm`` I get my email address.    It seems like I type my email address about 30 thousand times a day so I love having that.  I've also started using the advanced features (known as macros) to [quickly fill out][tefif] some reminder emails I send frequently that are always the same text.  I was already a [1Password][1pw] user and its definitely been my favorite password manager.  It is expensive to get the bundle for Windows, OSX, and iOS but really worth it considering how much information I store in the app and how much [more secure][pi] it makes my online life.  1password might be the prettiest piece of software I use, which _does_ matter (damn it).  I still have some other applications I'd like to check out, namely Hazel and Omnifocus.

The Macbook's trackpad and keyboard are the best I've used on a notebook, period.  Trackpads on PC notebooks have long been a point of contention.  Whether they're too small, have weird gestures, or just plan don't work there's always some issue.  The latest EliteBooks from HP have come the closest to getting it right that I've seen.  Battery life on the notebook has been great and I love that OSX will tell you which apps are eating a lot of energy.  Coming from a system that mysteriously always ran awful on battery this has been most excellent.

The [continuity features][con] of Yosemite were a big reason I was interested in switching.  I already own an iPhone and iPad.  Being able to iMessage, and even send regular text, from the computer is very convenient.    This is where I feel like Apple is really edging out Microsoft, it would give people a reason to actually consider Windows Phone to have this sort of functionality.

#The Bad

The Macbook has only locked up twice but both were kind of worrying.  The first  was actually right out of the box during setup.  It hung forever when I declined to setup iCloud Drive.  When I forced the system to reboot it went  through but had apparently already created my user account.  I elected to just wipe the system out and start over using the download from internet option to install Yosemite.  This worked without issue.  The system also froze completely when I first connected it to the Thunderbolt Display.

I almost immediately discovered an unfortunate bug with Exchange Online and Yosemite.  The Contacts App will not sync contacts from an Office 365 Exchange Online account (or at least from my account).  An issue [documented by others][9] but remains unfixed even in the latest Yosemite beta (currently 10.10.2).  I opened a case with Microsoft support, who promptly laughed and then closed the case.  I haven't tried opening one with Apple just yet.  I worked around the problem by manually copying my contacts into iCloud.  There's a sort of hack to use Microsoft's OWA app in iOS to sync your contacts into iCloud but this only works on the default/base Contacts folder and I maintain more than one.

#The Miscellaneous

One of the things that I notice the most is that I've gotten a feel for performance on a PC.  You get a sixth sense for if something isn't right.  Like if Outlook takes longer than 30 seconds to open its probably not going to open at all.  I feel a little lost sometimes on the Mac if something unexpected happens and I find myself starring at the beachball.  I also really miss indicator lights for disk activity, etc, and find myself tapping the caps lock key to make sure my Macbook is alive.

There are way more boot options for the Mac than Windows and I sort of wish they were all consolidated into one menu.  Although not having the POST and then Windows load that I'm used to is what makes for the variety of options/confusion.

I immediately loaded Bootcamp on the system but I haven't really had much reason to use it.  [VMWare Fusion][vmw] can use your Bootcamp partition as a virtual machine to run Windows apps within OSX, this is good because you don't need to devote any more disk space to another virtual hard drive.  Initially I thought I would probably use Fusion to run Outlook 2013 instead of the latest Outlook for Mac but then I discovered that Fusion takes a good chunk of battery and really the latest Outlook for Mac is good enough. I'd still like it to have more feature parity with the Windows version and to stop using Exchange Web Services (Windows connects to Exchange using MAPI).  At work I've been going back and forth with Microsoft for months to try to figure out why the latest Outlook for Mac will not send attachments larger than 10 or so MB when using an Exchange 2010 account. 

So far Apple has lived up to its promise to "just work" in every way except for the Contacts bug.  I hope to update this post and add others with more good things as time goes on.

[1]:http://amzn.to/1D3fmGd
[2]:http://amzn.to/1wYwpJ5
[3]:http://amzn.to/1HXbO9r
[4]:http://www.amazon.com/gp/product/B004YLCKYA/ref=oh_aui_detailpage_o02_s01?ie=UTF8&psc=1
[5]:http://railstutorial.org
[6]:http://learnpythonthehardway.org
[7]:http://appleinsider.com/mac_price_guide
[8]:http://buyersguide.macrumors.com/
[9]:http://www.andrewconnell.com/blog/resolving-contacts-sync-on-os-x-error-soapwebserviceserrordomain
[mpu]:http://www.macpowerusers.com/
[te]:http://smilesoftware.com/TextExpander/index.html
[1pw]:https://agilebits.com/onepassword
[pi]:http://www.nonadmin.com/password-insecurity
[con]:https://www.apple.com/osx/continuity/
[vmw]:http://www.vmware.com/products/fusion
[server]:http://www.nonadmin.com/home-server
[tefif]:http://vimeo.com/44456425