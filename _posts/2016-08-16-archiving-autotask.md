---
layout: post
title: Archiving Autotask
date: 2016-08-16 16:45
summary: Taking 7 years of tickets from MS SQL to Postgres
category: blog
tags: rails postgresql developer
---

After much wailing and gnashing of teeth our team finally said goodbye to [Autotask][1] last week.  Turns out it might not be a good idea to take a customer that's been month to month for years and suddenly demand they sign a three year contract.  

When you leave Autotask they give you a nice MS SQL .bak file of all of your data.   After getting a trial of SQL 2016 installed I discovered that that Autotask's data structure looks like something out of Dante's Inferno.  1,215 tables.  558 views. 😯 

I set about extracting the data that's relevant to us - tickets, time entries, and internal notes.  After digging around for a little while I honed in on the tables we'd need to see the ticket data as well as the technician that worked on it, client, and end user.  

### Exporting, first attempt

My goal is to get the data out of MS SQL and into Postgresql which will eventually get plumbed by a Rails application.  The application is just for browsing and searching old tickets, the collective memory of the company so its important to hang onto.  I think its the IT guy in me but my first thought was to try for a CSV of all the data.  I found some great blogs about the [BCP utility][2] which is literally for bulk import and export of data.

This command exports a single table of tasks (tickets) to CSV with tabs separating the fields and ```\n``` for a new line.  

```
bcp "dbo.tblTask" out “c:\export\tblTask.csv" -S localhost \
-d databasename -C RAW -T -c -t “\t” -r "\n"
```

* First option is the table name
* **out** destination file name
* **-S**: server name
* **-d**: the database name
* **-C**: the code page to use, don't really know about this.  MSDN states that RAW means "No conversion from one code page to another occurs. This is the fastest option because no conversion occurs."
* **-T**: use Windows authentication with the currently logged in user
* **-c**: Stops BCP from prompting for a character type for each field, it uses the character data format for each field.  This is specifically to use when exporting data bound for something other than another MS SQL instance.
* **-t** and **-r** the field and record terminators, not actually needed when using ```-c``` as these are the default

Almost immediately I ran into two issues.  One, BCP
does not export field names.  So there's no headers in the CSV.  This was easily resolved by simply running a select query on the tables in question and literally copying the first couple of output rows using the option to include headers in Management Studio (Control + Shift + C).

The bigger problem was new lines in the data.  A lot our tickets and internal notes contain new line characters.  This is obviously no problem when the data is in a proper DBMS but a CSV file doesn't handle newlines very well.  BCP can technically terminate a record using anything you like,  ▓ for instance, or maybe ╣. However, Excel and [Ron's Editor][3] (seriously awesome if you deal with a lot of CSVs) don't like this, they expect new lines to terminate records.  Officially CSV is supposed to support new lines in the data but I've never been able to figure out.

At this point I was starting to have flashbacks to another large data migration involving CSVs and hunting for line breaks and other naughty characters. So I decided to rethink my method.  If the goal is to go from one SQL DB to another, why even bother with the intermediary step?  In particular risking data loss with CSV mangling something.  Surely others have done this, maybe there's even some sort of neato open source tool for the job...

### Exporting, second time's the charm

Poking around a little more I found a neato [Python tool][4] for this exact purpose - migrating from MS SQL to Postgresql.  After installing a few prerequisites the tool works in 5 steps.  First you take a SQL dump of the database or individual tables you're interested in.  You then run the tool itself to generate some SQL files for Postgres.  Then psql the generated "before" script. Next, run a migration using the prerequisite Kettle software.  Finally psql the generated "after" script.

This, somewhat incredibly, just worked for the most part.  It barked about some missing associations. I went back into Management Studio and ran a diagram of associations for the six tables we needed - that brought up about 50 tables, most of which contained hilariously irrelevant data.  I was getting a little sad at the idea of migrating 4 dozen extra tables when I decided to check the new Postgres database - everything we needed was there!

### Rails

As it turns out, massaging Rails to use an existing database is actually quite simple.  I had to do a pretty minimum amount of work on the Postgres database itself.  I spent some time cleaning up field names to match the "model_id" naming that Rails expects for associations.   

You'll need to make sure you've got the 'pg' gem set, and then just plug in the database in ```config\database.yml```

{% highlight YAML %}
default: &default
  adapter: postgresql
  encoding: unicode
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: autotask_backup
{% endhighlight %}

This of course will also be my production database but for now we're just in development mode.

You then will need to generate the schema file.  I'm not sure if this is actually necessary but its a good test of whether or not Rails is connecting to your database properly.

```rake db:schema:dump```

I noticed at this point that Rails wasn't picking up on the  primary key (id).  After correcting this in [Postico][5] and re-dumping the schema...

{% highlight ruby %}
ActiveRecord::Schema.define(version: 0) do

  # These are extensions that must be enabled in order to support this database
  enable_extension "plpgsql"

  create_table "contacts", force: :cascade do |t|
    t.integer  "customer_id",                              null: false
    t.string   "phone",                        limit: 25,  null: false
    t.string   "phoneext",                     limit: 10,  null: false
    t.string   "fax",                          limit: 25,  null: false
...
{% endhighlight %}

Booya!

Now you need the actual models for Rails.  The regular command line to generate the model is going to create a migration file which you do not want.  So ```rails g model Contact --skip migration``` (or you could just delete the migration file, but that seems sketchy to me)

Alright.  So let's drop down to the console and see if all this actually worked.

```
[1] pry(main)> Contact.all.sample
  Contact Load (12.4ms)  SELECT "contacts".* FROM "contacts"
+----------+-----------+----------+-------------+
| id       | firstname | lastname | customer_id |
+----------+-----------+----------+-------------+
| 29683607 | Fake      | Customer | 29683605    |
+----------+-----------+----------+-------------+
1 row in set
```

Booya!²

From here on its just standard Rails development.  I
considered blogging about the rest of the application but this is literally for indexing and reading old data.  So I can't imagine it will be that interesting. 

If you've been traveling down a similar path, I hope this post has helped. And if you noticed anything incredibly stupid or wrong here - <a href="{{ site.baseurl }}/contact/">let me know!</a> 

[1]: http://www.autotask.net
[2]: https://msdn.microsoft.com/en-us/library/ms162802.aspx
[3]: http://www.ronsplace.eu/Products/RonsEditor
[4]: https://github.com/dalibo/sqlserver2pgsql
[5]: https://eggerapps.at/postico/
