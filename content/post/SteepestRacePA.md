+++
date = "2015-06-04T15:13:32-04:00"
draft = false
title = "Pennsylvania's Steepest Race"

menu = "main.topics"

topics = ["Running"]
tags = ["data", "trail races"]

+++

### The Spark of the Project
A few days ago, I went on a run with the North Park Trail Runners, 
and a discussion came up about different races people signed up for. 
It inevitably led to a conversation about how tough each race is. 
Don't get me wrong, if you are doing a race, it will be challenging 
no matter what - if it's flat, you're probably pushing your body to 
go real fast; if it's a little hilly, maybe you're sprinting up all 
the hills; or if it's very steep, you'll be walking up the climbs and 
running down the hills or trotting (and even the downhill can be 
challenging based on the terrain).

So while there is no one true metric to answer the question 
of "what is the most challenging race?", 
I have attempted to answer a piece of that puzzle.

### The Races With Hills

> *It's a hill - get over it!*


In my time here in Pennsylvania, I've heard the names of different 
races thrown around - Rothrock Challenge, Megatransect, Lost Turkey, etc. 
I tried going through some of those races and quantifying the vertical gain per mile.

The tricky thing here is that not every race advertises its 
estimated elevation gain, and even if it does, you can't always 
trust it. You also can't trust the advertised miles - actually, nor the ruuners... well, 
especially the runners that like to challenge themselves and 
add extra off-course miles (I do this quite often... some people call it getting lost).
To assist with accuracy, I gathered together GPS data for the different races.

I put together a small list of "challening" races
and pulled out their elevation gain in feet and divided
it by the total miles of the race, based on data recorded from racers.

So far the races are as follows (in alphabetical order):

 - Call of the Wilds Marathon
 - Eastern States 100
 - Hyner View Challenge
 - Laurel Highlands Ultra
 - Megatransect
 - Rock'N the Knob
 - Rothrock Challenge
 - Frozen Snot
 - Laurel Highlands (50K)
 - Worlds End Ultra
 - Worlds End Ultra (50K)
 - Dirty Kiln Half Marathon
 - Pittsburgh Marathon
 - Rachel Carson Trail Challenge
 - Glacier Ridge Trail Ultra
 - Glacier Ridge Trail Ultra (50k)
 - Hell Hath No Hurry
 - Hell Hath No Hurry (50k)

Hopefully I can update/add to this list as I do more research and get more suggestions.

### Results

> Categorizing the races by their elevation gain/mile only reveals one degree of difficulty.

![Graph of vertical gain (ft) per mile](/images/SteepPA/SteepestPARace.jpg)

By purely looking at numbers, the Frozen Snot is by far the steepest race with 389 ft/mile.
What makes that race even more of a challenge is the fact that it's held at the end of
January. Aside from that, the Wilds Marathon has the second most vertical gain per mile at 258 ft/mile, 
with the Hyner View Challenge (50K) coming in closely behind it at 256 ft/mile.


Below you can see for yourself, all the numbers that went into the graph above.

![Race data](/images/SteepPA/RaceData.jpg)

I decided to add in the Pittsburgh Marathon for reference on how a "hilly" road race compares.

### Update with Interactive Graph

I put this data into a CSV file and used a webservice called [Plotly](https://plot.ly) to
style the data a little differently than excel was letting me and add comprehension to the races.

See below:
<div>
    <a href="https://plot.ly/~elShambler/244/" target="_blank" title="PA Races - Races by pace and Elevation gain/mile" style="display: block; text-align: center;"><img src="https://plot.ly/~elShambler/244.png" alt="PA Races - Races by pace and Elevation gain/mile" style="max-width: 100%;width: 1598px;"  width="1598" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="elShambler:244"  src="https://plot.ly/embed.js" async></script>
</div>

I have added some additional data that I used to make this plot - pace. I grabbed an average of
the top 10 finishers' paces and used that as an additional data point to graph against. The
reasoning behind this and the above graph is to add additional character to the different
races.

My hope in including this data is to give an overall enlightenment to how a race
will feel when it comes time for raceday. The higher the bubble is, the slower
pace you can expect, and the darker the bubble, the more climbing there is.

Something kind of interesting that I have been pondering after making this graph is
if there is some sort of plateau of base pace for varying lengths of races. I know
that as the body's metabolism switches systems over long periods of high expenditure,
the pace will drop, but wonder how it plays into elevation gain too?


### Data
Since the crux of what I'm saying relies on data that I put together, 
I think it is worthwhile noting where it came from. I tried to gather 
together blog race reports and the numbers that people reported there, 
but I was also able to track down some data from publicly available sources (i.e. Strava).

To be thorough with my results, I tried using a decent sample size to 
get an average worth reporting since I know the way GPS works varies 
between watches and the different algorithms companies utilize. 
Decent sample size meant 4 or more people (not that many, I know, 
but finding the data was hard).

##### Distance

The distance category for the race should be obvious, 
that's where you know the distance you're signing up for... 
roughly.

For the most part, the reported distance is what was ran. 
The data used here was the GPS recorded distance. I compiled
different racer's data, and averaged them together. I tried to
ensure the GPS route followed the plotted course for the race,
although this was more of a rough estimate. I also tried to make
sure the distance made sense and that the runners finished the
course.

All races cannot guarantee you'll run the exact amount that was
initially advertised though, and it may depend on the
course you take (i.e. getting lost). The one interesting result 
was the Laurel Highlands coming in about 3 miles short. I haven't
run the race, so I'm not sure where the discrepancy would come in.
Potentially, the uphill + downhill gains could make up the 3 miles.

##### Elevation
Elevation was the main data that I used for my thoughts here. 
Elevation is a tricky thing. I won't go into the details, but 
[Jeff Friedl](http://regex.info/blog/2015-05-09/2568) blogged about 
how Strava may inflate/overestimate their elevation numbers. It's very difficult to 
tell, and quite honestly, I used Strava for 95% of my data, 
so at least they're all consistently inflated (hopefully).

I, again, used the same method of just pulling different 
racer's GPS Elevation gain data and then averaged at the end.

### Final Thoughts
Does this mean that the Wilds Marathon or Frozen Snot is the toughest race in PA? 
Heck no! The distance of the Eastern States add a whole new element 
of pain to the table. But then you have Rothrock and Megatransect 
that both add a layer of complexity with the amount of rocks and 
technical portions that also make it challenging.

The Frozen Snot definitely is an interesting beast because of its
steep ascents over short distance - it approaches the climbing
difficulty of a VK (vertical kilometer race). Runnability is also
huge into the challenges of a race - this has to do with the
technicality of the course. As I said above, the body's method
of producing energy shifts at a certain point (I don't remember
what it is, but I want to say after maybe 8 hours of heavy
calorie expenditure?) and after that, maintaining a constant
energy output becomes even more challenging.

All this to say that no single race is quantitatively better than
another race. The beauty of trail running is each race offers its own set of 
challenges. I think the lesson to learn here is you may need to try 
each one to determine which is the most challenging for yourself!
