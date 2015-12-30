---
layout: post
title: A Year of Programming
date: 2015-12-30 08:38
summary: a long way from precious gems
category: blog
categories: viking developer
callout: This is the third in a series of posts detailing my experience in the [Viking Code School](/blog/2015/06/02/puts(hello-world!)/)
---

Today marks roughly 5 months since I last posted about my course work in Viking.  In that time I've moved through data structures, Sinatra, APIs and web scraping, SQL, and finally Rails.  As I look to  the final section of the course, Javascript, I have a functional clone of Facebook.  Users can sign up, post, comment, like, friend, search for other users and upload photos.  The latter part of the development was all test driven with feature, model, and controller specs.

{% highlight sql %}
/* Who took the cheapest flight? */
SELECT MIN(Flights.price) AS cheapest_ticket
FROM Users JOIN Itineraries ON Users.id = Itineraries.user_id
           JOIN Tickets ON Itineraries.id = Tickets.itinerary_id
           JOIN Flights ON Tickets.flight_id = Flights.id
WHERE Users.id IN 
(SELECT Users.id
   FROM Users JOIN Itineraries ON Users.id = Itineraries.user_id
              JOIN Tickets ON Itineraries.id = Tickets.itinerary_id
              JOIN Flights ON Tickets.flight_id = Flights.id
   GROUP BY Users.id
   HAVING COUNT(*) = 1)
{% endhighlight %}

Reflecting back on the year I think the biggest difference between then and now is confidence.  Understanding that an know how an application is put together in something like Rails is more important than remembering every single detail.  I've coded in airports while waiting on flights, I've worked in the passenger seat of my girlfriend's car while we drove to visit friends in the pouring rain.  Which isn't to say I'm some sort of superhero, just that I really love doing this.

{% highlight ruby %}
# Putting together an efficient query can be messy business
def self.search_by_name(search_string)
  unless search_string.blank?
    search_terms = search_string.split
    search_conditions = [(['first_name ILIKE ?'] * 
                        search_terms.length).join(' OR ') + " OR " +
                        (['last_name ILIKE ?'] * 
                        search_terms.length).join(' OR ')] + 
                        search_terms.map { |term| "%#{term}%" } +
                        search_terms.map { |term| "%#{term}%" }

    results = Profile.where(search_conditions)
    find(results.pluck(:user_id)) if results.any?
  end
end
{% endhighlight %}

The only real issue I've faced is finding the time needed to study (and being in front of the computer on top of my regular IT hours).  The Viking course load is no joke, and even though I've dedicated around 70 hours a month on average its not nearly the ideal pace.  The main problem is retention, for instance the work I originally did with web scraping is starting to get fuzzy and will require some refreshing when I put that knowledge to use in future projects.

Ahead of me still is Javascript, and the end of the course.  Happy New Year!