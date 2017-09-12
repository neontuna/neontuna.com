---
layout: post
title: Pinballer
summary: One API request is better than 50.
date: 2017-09-11 16:30
category: blog
tags: javascript vuejs developer
---

Pinball, like most competitive games, has a heaping of stats.  Arcades/barcades host tournaments and league play.  Our local barcade [Abari][1] wanted to post standings for their tournaments online.  They use [Matchplay.events][2] which exposes a pretty robust [API][3].  At first we just needed something basic that that hit the "Standings" endpoint of Matchplay and enumerated the players name and rank.

I've been doing a lot of work with [VueJS][4] recently. The framework makes creating a nice table from an API response a breeze.  

We'll setup a new Vue instance and in the `created` lifecycle hook generate a XHR.  Then parse the response and set an array defined in the `data` property of the Vue instance.  We could use something like [Axios][5] instead of XHR but it seems silly to add another library to the mix for a single request.

{% highlight javascript %}
var vm = new Vue({
  el: '#el',
  created () {
    var xhr = new XMLHttpRequest()
    
    xhr.onreadystatechange = function(vm) {
      if (this.readyState === XMLHttpRequest.DONE) {
        if (this.status === 200) {
          vm.players = JSON.parse(this.responseText);
        } 
      }
    }.bind(xhr, this)
    
    xhr.open("GET", 'https://matchplay.events/api-beta/tournaments/0nbw/standings');
    xhr.send();
  },
  data () {
    return {
      players: []
    }
  }
})
{% endhighlight %}

The HTML is even more basic.  Just have Vue grab onto a table using the `id` attribute and then iterate over the players array using the built-in [v-for directive][6].

{% highlight html %}
{% raw %}
<table id="el" class="table">
  <thead>
    <tr>
      <th>Rank</th>
      <th>Points</th>
      <th>Name</th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="player in players">
      <td>{{ player.position }}</td>
      <td>{{ player.points }}</td>
      <td>{{ player.name }}</td>
    </tr>
  </tbody>
</table>
{% endraw %}
{% endhighlight %}

You can see the results in this [jsfiddle][7].

So all that's pretty simple.  Why not make it needlessly complicated so we have to write exponentially more code?! 😛

[IFPA][8] or The International Flipper Pinball Association maintains stats on Pinball players through its World Pinball Player Rankings (WPPR).  Most of the players in Abari's tournaments will have a WPPR score so wouldn't it be cool to show that along with their standing in the current tournament?  And IFPA even has its own [API][9].

It turns out that Matchplay keeps track of the IFPA ID of players, as we can see in this result from its tournament endpoint

{% highlight javascript %}
{
    "player_id": 10263,
    "user_id": 187,
    "claimed_by": 1168,
    "ifpa_id": 30959,
    "name": "Justin Richardson",
    "status": "active",
    "tournament": {
        "tournament_id": 10866,
        "status": "active",
        "seed": null
    }
}
{% endhighlight %}

And we can use that ID from Matchplay to pull the player's stats from the IFPA API.

{% highlight javascript %}
{
    "player": {
        "player_id": "30959",
        "first_name": "Justin",
        "last_name": "Richardson ",
        "city": "Charlotte",
        "state": "NC",
        "country_code": "US",
        "country_name": "United States",
        "initials": "JSR",
        "age": "",
        "excluded_flag": "N",
        "ifpa_registered": "Y"
    },
    "player_stats": {
        "current_wppr_rank": "2076",
        "last_month_rank": "2211",
        "last_year_rank": "3650",
        "highest_rank": "2140",
        "highest_rank_date": "2017-09-01",
        "current_wppr_value": "33.07",
        "wppr_points_all_time": "38.41",
        "best_finish": "1",
        "best_finish_count": "1",
        "average_finish": "12",
        "average_finish_last_year": "17",
        "total_events_all_time": "35",
        "total_active_events": "35",
        "total_events_away": "0",
        "ratings_rank": "405",
        "ratings_value": "1585.70",
        "efficiency_rank": "1257",
        "efficiency_value": "13.550"
    },
    "championshipSeries": [
        {
            "view_id": "100",
            "group_code": "NC",
            "group_name": "North Carolina",
            "rank": "14",
            "country_name": "US"
        }
    ]
}
{% endhighlight %}

But here's the rub - the IFPA API doesn't have a way to look up a whole bunch of players at once.  There's a [search page][10] on the site that allows for multiple lookups but it doesn't appear to use the same API they expose to the public.

So that means we'd need to hit the IFPA API for each player in the tournament.  For a visitor on Abari's website that means looking at either a loading screen or a partial table that gradually populates with each player's stats.  Not to mention that this happens every time someone visits the site, which would not be very nice to IFPA's API.

We can't get around doing an individual lookup on IFPA for each player but what if we could cache the results and only update the cache periodically?  And how do we keep the user experience nice on the Abari site - ideally as few requests as possible so the site loads quickly?

The solution I ended up with is [Pinballer][11], which somehow isn't already the name of a novelty store in the mall.  It's a Rails API that:

  1. Requests standings of a Matchplay Event based on its ID
  2. Plucks all the IFPA player IDs returned from the Matchplay response
  3. Requests the IFPA stats for each player ID and stores them
  4. Returns the Matchplay standings with IFPA stats for each player merged

The first time it sees a Matchplay ID, it essentially just returns the standings without any IFPA stats (unless it happens to already have some of the players' stats in the database).  However in the background it begins caching the stats for each player.  The same request run again a few minutes later ought to return all the players' stats along with their standings.  There's also a scheduled task that updates the stats for all players in the database once an hour.  I rate limit the amount of requests to the IFPA API so we're not spamming them into oblivion but hopefully a few dozen requests once an hour won't kick up too much dust.

The app takes advantage of Postgresql's JSONB data type and just chucks the entire result for each player into the DB, no need to remodel IFPA's data locally.

{% highlight ruby %}
create_table "players", force: :cascade do |t|
  t.jsonb "ifpa_stats"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
  t.string "ifpa_id"
  t.string "matchplay_player_id"
end
{% endhighlight %} 

I also found the [Suckerpunch][12] gem while making the API.  I've previously used delayed_job to run methods asynchronously in Rails.  Suckerpunch also works with the new ActiveJob class but it doesn't require a separate process.  So for instance a Rails application on Heroku won't require a second dyno to spin up every time there's jobs to run.  Less 💰 = 👍

After setting up Pinballer, our final code for Abari looks like [this jsfiddle][13].  We also pull the NC state ranks out of the IFPA stats and a time stamp for the last IFPA update.

{% highlight javascript %}
var vm = new Vue({
  el: '#el',
  created () {
    this.getData()
  },
  data () {
    return {
      players: [],
      loading: true
    }
  },
  methods: {
    getData: function() {
      var xhr = new XMLHttpRequest()

      xhr.onreadystatechange = function(vm) {
        if (this.readyState === XMLHttpRequest.DONE) {
          if (this.status === 200) {
            vm.players = JSON.parse(this.responseText);
          } 
        vm.loading = false
        }
      }.bind(xhr, this)

      xhr.open("GET", 'https://pinballer.neonwine.com/matchplay_events/n32j.json');
      xhr.send();    
    },
    getNcRank: function(series) {
      var nc = series.find(function(state){ 
        return state.group_code === "NC" }
      )
      return nc ? nc.rank : ''
    },
    ifpaLastUpdated: function() {
      var ifpa_player = this.players.find(function(standing){
        return standing.ifpa_updated_at
      })
      return ifpa_player ? 
        new Date(ifpa_player.ifpa_updated_at).toLocaleString() : 'No IFPA Players?'
    }
  }
})
{% endhighlight %}
{% highlight html %}
{% raw %}
<div id="el">

<span v-if="loading">
  Getting tournament standings...
</span>

<span v-else-if="!players.length" v-cloak>
  ¯\_(ツ)_/¯ Something went wrong!
</span>

<div v-else v-cloak>
<table class="table">
  <thead>
    <tr>
      <th rowspan="2">Rank</th>
      <th rowspan="2">Points</th>
      <th rowspan="2">Name</th>
      <th colspan="3">IFPA Stats (updated: {{ ifpaLastUpdated() }})</th>
    </tr>
    <tr>
      <th>WPPR Rank</th>
      <th>WPPR Points</th>
      <th>NC State Rank</th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="player in players">
      <td>{{ player.position }}</td>
      <td>{{ player.points }}</td>
      <td>{{ player.name }}</td>
      <template v-if="player.ifpa_stats">
        <td>{{ player.ifpa_stats.player_stats.current_wppr_rank }}</td>
        <td>{{ player.ifpa_stats.player_stats.current_wppr_value }}</td>
        <td>{{ getNcRank(player.ifpa_stats.championshipSeries) }}</td>
      </template>
      <template v-else>
        <td></td>
        <td></td>
        <td></td>
      </template>
    </tr>
  </tbody>
</table>  
</div>

</div>
{% endraw %}
{% endhighlight %}

This was a pretty fun project and I'm excited to help Abari with more in the future.  I just hope I can keep coming up with excellent names for my side projects.

[1]: http://abarigamebar.com
[2]: https://matchplay.events
[3]: https://matchplay.events/api-docs
[4]: https://vuejs.org/
[5]: https://github.com/mzabriskie/axios
[6]: https://vuejs.org/v2/guide/list.html
[7]: https://jsfiddle.net/nonadmin/8fk8c5w9/7/
[8]: https://www.ifpapinball.com/
[9]: https://www.ifpapinball.com/api/documentation/
[10]: https://www.ifpapinball.com/players/find.php
[11]: https://github.com/neontuna/pinballer
[12]: https://github.com/brandonhilkert/sucker_punch
[13]: https://jsfiddle.net/nonadmin/8fk8c5w9/