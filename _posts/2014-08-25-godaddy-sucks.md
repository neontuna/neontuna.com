---
layout: post
title: Godaddy sucks
date: 2014-08-25 11:42:44
summary: Surprise!
category: blog
tags: rant
---

Last week the HTTP monitoring sensors for one of our clients started going haywire.  Their site was taking way too long to load or not loading at all.  Going to the site in a web browser presented alternating database errors and partial loads of the content.  

Most of our clients are hosted through Godaddy and were originally setup years ago.  If we were making that decision again it'd probably be with a different company.  But the cost of moving clients off Godaddy pretty much outweighs our personal feelings about them.

I called Godaddy support about the site.  For hosting support you end up speaking with a first level phone rep and if you're escalated it just means that same phone rep talks to someone at a higher level for you.  So after deflecting her initial assertion that our Wordpress was configured incorrectly she escalated the call.  Second level apparently told her that we should enable SSH access to our account which would migrate everything to a different datacenter.  I protested this, we didn't need SSH access and this seemed like a really hack method of resolving issues.  She continued to insist this was the only option and so I went ahead with it.

A few days later and the site was loading fine.  The rep had said that some updates would need to be made to the Wordpress configuration to point to the new database location but we never received the email that was supposed to detail these changes.  So I logged into our client's hosting panel and was presented with a page that stated we didn't have a hosting plan.

I called Godaddy again.  The phone rep reported that the site never migrated completely.  The content is on the new server while the database is still on the old system.  Godaddy's solution to this?  Remove the (Wordpress) database, which will bring the site down, but will allow the migration to complete.  And then restore the database on the new system.  Total downtime?  1-3 **Days**.  We still have access to the data and old database, and so my suggestion was that a new hosting account be created and we simply upload everything and restore a database backup.  Godaddy says this will work but we'll need to purchase the hosting plan.  And we'll need to do all the migration legwork ourselves.

So basically Godaddy support tells me to do something which breaks our hosting plan and then tells me to take the site down to fix it, or migrate to a new hosting plan on our dime to fix things without any downtime.