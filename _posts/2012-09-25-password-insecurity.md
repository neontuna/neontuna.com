---
layout: post
title: Password Insecurity
summary: How to deal with passwords without going insane
category: blog
tag: blog
---

>This is a copy of a post on my [company's blog][1].  So its more of a simplified approach to discussing Passwords.  At least as simple as I can get *pushes up glasses*

![Ferris Bueller](/assets/vlcsnap-2012-09-24-19h59m53s114_large.jpg)

Towards the beginning of the movie _Ferris Bueller's Day Off_, Matthew Broderick tells us "I asked for a car, I got a computer."... as he's changing his days absent in the school's server.  We're not shown how Ferris gets into the school network but he may have just guessed someone's password.

One question I come across pretty frequently when helping someone with their system is "Why?".  Why do people create viruses?  Why do they care about my computer?  Why do they want access to my account?  The bad guys of the computer world used to be something like Ferris, mostly harmless and just playing pranks.  However "hacking", viruses and the like are now big business and its not just a bunch of kids anymore.  Simply put - There's a lot of money in taking control of a system or even something as simple as an email account.

Fortunately one of the best things anyone can do to protect themselves online doesn't take a degree in Computer Science -  Strong, unique passwords are worth more than the most expensive Firewall.  A lot of folks groan at the suggestion of this, particularly unique passwords.  And, trust me, I know - In IT I've dealt with having to enter hundreds of unique passwords any given week.  But give me a chance to explain why this is so important.

If you're at all active on the internet you've probably created accounts on many, many websites.  Some you might have even forgotten about.  From Facebook to Youtube to eBay you can't do a whole lot online without signing up and logging in.  But what happens to your account after you sign-up, where is your information kept?  What if you used the same password for your bank that you used for Yahoo?  This is where the problem of non-unique passwords comes into play.  You could have a password that was 30 letters long and full of numbers and @ signs and all kinds of things only you would know about - but that password might be kept somewhere completely insecure.  Maybe even by a company you would assume would take security very seriously.  [Yahoo][2], [LinkedIn][3], [Gawker Media][4], [Sony][5] - just to name a few, have all had their systems breached and users' accounts stolen.  Any accounts with the same password could be considered compromised: your bank, credit card, etc.   In fact you can even check if your information was included in a password "leak" - [PwnedList.com][6] will run your email through a list of nearly one billion and report if it was found in any of the password leaks they've collected.

On the flip side of this issue, remembering a unique password for 100s of different websites is _impossible_.  Particularly if you're forced to create what is considered a "secure" password.  For instance maybe your bank considers S0m3Th1ng L1k3 T4is to be the paramount of security, no matter how much hair pulling goes on every time you're forced to type that in.  So what exactly can you do?

Well one thing is to keep a list.  You could write down every password you've ever needed, or maybe even keep them in a super secret word document hidden on your computer.   And if you're laughing right now - trust me, I've seen the "hidden word document" technique in action.  There are some _slight_ downfalls to this of course.  However,  there are several different services and applications specifically meant to keep a password database.  I have used one in particular, [Lastpass][7], for several years and have never looked back.  Every time you create a new account for a website Lastpass will remember the credentials and re-enter them for you when you return.  It's web browser add-in can generate passwords, keep secure notes and even keep a profile to automatically fill in your address, phone number etc when they're required online.  The system is setup so that you have one password that you need to remember, your Lastpass password.   Once signed in your password "vault" is decrypted\* and Lastpass handles the rest.  And one of the best parts of Lastpass is that its free.  You can pay a small subscription fee, 12 dollars a year, to have access to the vault from your smartphone via an App but to use it on your computer's web browser is free.

A word on the security of Lastpass itself.  Your password vault is kept on Lastpass' servers, which is good because you don't need to worry as much about keeping a backup of the vault (although it is still possible to do this if you'd like).  This probably immediately causes some alarm though, after all we just finished talking about big time websites being hacked.  However even if Lastpass itself is compromised you should still be safe.  In order to access your personal vault your Lastpass password is required and your vault is encrypted from your computer.  So if your personal database was stolen it would be useless without your password to decrypt the contents.

One final note before closing - on "strong" passwords.  If you've tried to create an account when a complex password was required you know this can be frustrating.  And, unfortunately, as [this][8] Xkcd comic illustrates - sort of pointless.  Basically length is the only thing that truly matters.

Thanks for taking the time to delve into password security with us today. While not the most interesting subject I think you'll agree that playing hooky on your own security could end up being a real pain down the line, Bueller :)

\* Footnotes
I mention encryption here in passing, if you're curious this [Youtube][9] video explains the concept pretty well using paint and clocks!

   [1]: http://f1-networks.com/blog/
   [2]: http://www.engadget.com/2012/07/12/yahoo-security-breach/
   [3]: http://www.nytimes.com/2012/06/11/technology/linkedin-breach-exposes-light-security-even-at-data-companies.html?pagewanted=all&_moc.semityn.www
   [4]: http://news.cnet.com/8301-27080_3-20025558-245.html
   [5]: http://www.reuters.com/article/2011/04/26/us-sony-stoldendata-idUSTRE73P6WB20110426
   [6]: http://pwnedlist.com/
   [7]: https://lastpass.com
   [8]: http://xkcd.com/936/
   [9]: https://www.youtube.com/watch?v=3QnD2c4Xovk