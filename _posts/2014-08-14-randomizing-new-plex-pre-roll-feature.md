---
layout: post
title: Randomizing new Plex pre-roll feature
date: 2014-08-01 11:32:33
summary: You wouldn't download a car.
category: blog
categories: htpc software
---

![Simpsons THX Preroll](/assets/vlcsnap-2014-08-14-00h28m55s45_large.png)

Last week the wonderful folks at Plex announced support for extras and trailers to play before you watch a movie.  Which was already pretty great, but then a few days ago they [added support][1] for a custom "pre-roll" to play after the trailers and before your movie.  This immediately made me think of all the awesome stuff that used to play in movie theatres and on home video - THX, creepy lobby snack commercials, etc.

Right now the feature requires that you set one video from your library as the pre-roll clip.  I wanted to randomize this so the clip would change occationally.  Luckily the setting is stored in the registry (by the way this is for folks using Windows as their Plex server) under this key:

```
HKEY_CURRENT_USER\Software\Plex, Inc.\Plex Media Server\CinemaTrailersPrerollID
```

So for instance, the full registry key for one of my clips looks like this:

```
[HKEY_CURRENT_USER\Software\Plex, Inc.\Plex Media Server]
"CinemaTrailersPrerollID"="http://plex.tv/web/app#!/server/2b6b5...
...099e7fcbd0/details/%2Flibrary%2Fmetadata%2F4655"
```

There's probably a million ways to do this but I created a .reg file for each pre-roll in my collection and then used this VBScript to randomly insert the registry value for one of them.

{% highlight vbnet %}
Dim WshShell, intOption

Set WshShell = Wscript.CreateObject("Wscript.Shell")
Randomize()
intOption = Int(5*Rnd())

Select Case intOption
  Case 0
    WshShell.Run "regedit /S C:\etc\pre-roll-1.reg",0,True
  Case 1
    WshShell.Run "regedit /S C:\etc\pre-roll-2.reg",0,True
  Case 2
    WshShell.Run "regedit /S C:\etc\pre-roll-3.reg",0,True
  Case 3
    WshShell.Run "regedit /S C:\etc\pre-roll-4.reg",0,True
  Case 4
    WshShell.Run "regedit /S C:\etc\pre-roll-5.reg",0,True
End Select

Set WshShell = Nothing
{% endhighlight %}

Finally, I set that to run on a schedule every hour.  

I imagine we'll have the option to do something like this from Plex itself before too long.  On the blog linked above there were already a few requests for it.  Regardless, these new features are slick and I hope convince more folks to sign up for Plex pass. 


[1]:https://blog.plex.tv/2014/08/11/new-trailers-features/