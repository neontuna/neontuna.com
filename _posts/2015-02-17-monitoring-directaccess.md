---
layout: post
title: Monitoring DirectAccess
date: 2015-02-17 11:51:42
summary: With PowerShell and PRTG
category: blog
tag: blog
---

We recently deployed [Windows DirectAccess][3] for one of our clients and wanted to monitor the deployment.  In particular I was interested in the number of active DirectAccess sessions and if any errors were detected.  The Remote Access MMC snap-in (in Server 2012 R2) has a great  status page that runs down all of DirectAccess' components and reports any errors that are detected.

![DirectAccess built-in monitoring](/assets/Screen%20Shot%202015-02-17%20at%203.17.44%20PM_large.png)

You can basically assume that if any of these are red you're going to have an issue.  With PRTG the number of components in an error state could be a channel in a sensor which we could monitor and report on (for instance if the number of errors is greater than 0).   Similarly another channel could be the number of sessions.

DirectAccess comes loaded with a few PowerShell cmdlets for monitoring and reporting on its status.  In particular **Get-RemoteAccessHealth** and **Get-RemoteAccessConnectionStatisticsSummary** will give you a lot of information.  In order to make that data useful for PRTG we want to strip that output down from something like this (the raw output of Get-RemoteAccessHealth)

{% highlight text %}
Component            RemoteAccessServer   HealthState     TimeStamp            Id              OperationStatus
---------            ------------------   -----------     ---------            --              ---------------
Server               localhost            OK              1/27/2015 11:05:4...
6to4                 localhost            OK              1/27/2015 11:01:0...
Vpn Addressing       localhost            Disabled        1/27/2015 11:01:0...
Network Security     localhost            OK              1/27/2015 11:01:0...
Dns                  localhost            OK              1/27/2015 11:05:4...
IP-Https             localhost            OK              1/27/2015 11:01:0...
Nat64                localhost            OK              1/27/2015 11:01:0...
Dns64                localhost            OK              1/27/2015 11:01:0...
IPsec                localhost            OK              1/27/2015 11:01:0...
Kerberos             localhost            OK              1/27/2015 11:01:0...
Domain Controller    localhost            OK              1/27/2015 11:01:0...
Management Servers   localhost            Disabled        1/27/2015 11:01:0...
Network Location ... localhost            OK              1/27/2015 11:05:4...
Otp                  localhost            Disabled        1/27/2015 11:01:0...
High Availability    localhost            Disabled        1/27/2015 11:01:0...
Isatap               localhost            Disabled        1/27/2015 11:01:0...
Vpn Connectivity     localhost            Disabled        1/27/2015 11:01:0...
Teredo               localhost            Disabled        1/27/2015 11:01:0...
Network Adapters     localhost            OK              1/27/2015 11:01:0...
Services             localhost            OK              1/27/2015 11:05:4...
{% endhighlight %}

To literally just the number of components with a HealthState of "error".  This gives us our first sensor channel, "error count".   Our second channel is similar, I just want know how many active DirectAccess sessions we have which involves stripping down the output of Get-RemoteAccessConnectionStatisticsSummary

{% highlight text %}
TotalConnections           : 8
TotalDAConnections         : 8
TotalVpnConnections        : 0
TotalUniqueUsers           :
MaxConcurrentConnections   : 19
TotalCumulativeConnections : 2550
TotalBytesIn               : 4024483872
TotalBytesOut              : 8136960808
TotalBytesInOut            : 12161444680
{% endhighlight %}

To just the number of TotalDAConnections.

The full PowerShell script is below

{% highlight powershell %}
Write-Host "<?xml version=`"1.0`" encoding=`"Windows-1252`" ?>"
Write-Host "<prtg>"

Try
 {
    $daErrorCount = (Get-RemoteAccessHealth -ComputerName SERVER | Where-Object {$_.HealthState -match 'Error'} | measure).count
    $daUserCount = Get-RemoteAccessConnectionStatisticsSummary -ComputerName SERVER | Select -ExpandProperty TotalDAConnections
 
    Write-Host "<result>"
    Write-Host "<channel>DirectAccess Errors</channel>"
    Write-Host "<value>$daErrorCount</value>"
    Write-Host "</result>"

    Write-Host "<result>"
    Write-Host "<channel>DirectAccess Sessions</channel>"
    Write-Host "<value>$daUserCount</value>"
    Write-Host "</result>"
 }

Catch [system.exception]
 {
    Write-Host  "<error>1</error>"
    Write-Host  "<text>$_</text>"
 }

Write-Host "</prtg>"
{% endhighlight %}

This results in a PRTG sensor with the following channel setup

![Silvrback blog image](/assets/Screen%20Shot%202015-02-17%20at%2010.50.01%20PM_large.png)

The key lines here that give PRTG its value for the channels are

{% highlight text %}
(Get-RemoteAccessHealth -ComputerName SERVER | Where-Object {$_.HealthState -match 'Error'} | measure).count
{% endhighlight %}

and

{% highlight text %}
Get-RemoteAccessConnectionStatisticsSummary -ComputerName SERVER | Select -ExpandProperty TotalDAConnections
{% endhighlight %}


The first is getting our error count.  As we move down the command we're filtering the output of Get-RemoteAccessHealth for the HealthState column and then looking for any matches on the value "Error".  Both measure and count are needed to just return in integer, measure by itself returns this.

{% highlight text %}
Count    : 0
Average  :
Sum      :
Maximum  :
Minimum  :
Property :
{% endhighlight %}


Get-RemoteAccessHealth is returning a table and PowerShell makes it pretty easy to search through it like you might search for something in Excel.  Get-RemoteAccessConnectionStatisticsSummary is a little trickier.   Its returning a list and if you try to use the same method of stripping its output as Get-RemoteAccessHealth that won't work.  However Select -ExpandProperty will.  This took me a while to figure out until I piped the output of each command through Get-Member which reveals the Object-Types being returned.  [This article][1] was very helpful during that process.

The rest of the script is pretty standard fare for PRTG's custom XML sensors (a lot of great examples can be found [here][2]).  The entire script is returning back to PRTG something that looks like this

{% highlight xml %}
<?xml version=`"1.0`" encoding=`"Windows-1252`" ?>
  <prtg>
    <result>
      <channel>DirectAccess Errors</channel>
      <value>0</value>
    </result>

    <result>
      <channel>DirectAccess Sessions</channel>
      <value>10</value>
    </result>
</prtg>
{% endhighlight %}

Which is just the XML formatting that PRTG needs to construct a sensor.  The try statement is used to catch any errors since PowerShell can be annoying to get setup for remote monitoring and I wanted to make sure that any issues with the sensor itself were handled by PRTG instead of the sensor silently failing (for instance returning 0 for a channel, which wouldn't necessarily set off any alarms but might happen if one of our cmdlets failed)

[1]:http://windowsitpro.com/powershell/powershell-basics-formatting
[2]:https://code.google.com/p/prtg-addons/source/browse/#svn%2Ftrunk%2FCustom_EXE_Sensors%2FWindows_Powershell
[3]:https://technet.microsoft.com/en-us/library/dn636118.aspx