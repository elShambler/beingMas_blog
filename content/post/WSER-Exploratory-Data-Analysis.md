+++
date = "2016-12-27T22:26:35-05:00"
draft = true
title = "WSER Exploratory Data Analysis"

+++

## Basic Overview

My project is to explore the data provided from the Western States Endurance Challenge 100 mile run. The reason for choosing this race is because of its status in the ultra running world as the golden standard to compare all other races. It holds the aura of the premiere race to be in, and consistently draws the largest pool of athletes to its waiting list.

The numbers are capped for the race, with a handful of athletes getting based on achievements, others get lucky by getting their name pulled from a lottery. Either way, if you toe the line for the start of the Western States, you will be racing a historic race.

After each race, the race organization not only provides the official finish times, but also the results for time into each aid station. Quick clarifier, an aid station is a position along the race that offers aid (food, drink, area to rest, etc.). WSER (Western States Endurance Run) is a point-to-point course, and therefore has unique aid stations since there is no looping. Based on the distance an aid station is from the start, I can use the times to give an overview of how each racer does during the race. I will use this data to see not only how the race has evolved over time, but also to be able to predict how runners will finish, especially as they progress along the course.

### Table of Contents

1.  [Origin of the Data](#data-origin)
2.  [Process of Data Wrangling](#wrangling)
    -  [Massaging the Data](#data-massage)
    -  [Code for Wrangling](#wrangling-code)
    -  [Configuring Time](#time-configure)
    -  [Additional Time Fields](#time-fields)
3.  [Adding Features](#feature-adding)
    -  [Troubleshooting](#troubleshooting)
4.  [Wide and Long](#wide-format)
    -  [Long into Wide](#long-to-wide)
5.  [Exploring Finish Times](#finish-times)
    -  [Overall Finish Times](#overall-times)
    -  [Ranking Finishing Deciles](#ranking-deciles)
    -  [Aid Station Specific](#aid-station-specific)
    -  [Correlation Plot](#correlation)
6.  [Final Thoughts](#final-thoughts)

<DIV id = "data-origin">

Origin of the Data

</DIV>

![Western States Splits Data](/images/WSER-EDA/wserSplitsPage.png)

The data that is compiled here had to come in from the different files that are lisited above. When pulling in the data, I had to make a few edits, and therefore I did omit some data from the splits on years where I could not find the distance to a certain aid station. Otherwise, I did try and be as true to the original data as possible by including all information from the aid station for both time in and out of the aid stations.

I came up with a naming convention for each station that is broken down listed below. I'll list a few of the different ones as it will be helpful for not only you (reader) but for me as well. I'll include documentation for all the different aid stations for the final project.

<table>
<thead>
<tr class="header">
<th>AS Name</th>
<th>Dataset Name</th>
<th>Distance (miles)</th>
<th>In or Out</th>
<th>Type of Measurement</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Red Star Ridge</td>
<td><code>t16_in</code></td>
<td>16</td>
<td>In</td>
<td>Time</td>
</tr>
<tr class="even">
<td>Red Star Ridge</td>
<td><code>p16_in</code></td>
<td>16</td>
<td>In</td>
<td>Place</td>
</tr>
<tr class="odd">
<td>Duncan Canyon</td>
<td><code>t23.8_in</code></td>
<td>23.8</td>
<td>In</td>
<td>Time</td>
</tr>
<tr class="even">
<td>Duncan Canyone</td>
<td><code>p23.8_in</code></td>
<td>23.8</td>
<td>In</td>
<td>Place</td>
</tr>
<tr class="odd">
<td>Miller's Defeat</td>
<td><code>t34.4_in</code></td>
<td>34.4</td>
<td>In</td>
<td>Time</td>
</tr>
<tr class="even">
<td>Miller's Defeat</td>
<td><code>p34.4_in</code></td>
<td>34.4</td>
<td>In</td>
<td>Place</td>
</tr>
<tr class="odd">
<td>Miller's Defeat</td>
<td><code>t34.4_out</code></td>
<td>34.4</td>
<td>Out</td>
<td>Time</td>
</tr>
<tr class="even">
<td>Miller's Defeat</td>
<td><code>p34.4_out</code></td>
<td>34.4</td>
<td>Out</td>
<td>Place</td>
</tr>
</tbody>
</table>

The list goes on, and I got the full list of [aid stations here](http://www.wser.org/course/aid-stations/).

The dataset name listed above was of my own creation, and I used that in order to easily compare year to year and ensure that whatever data was captured from the race would line up with what I had compiled previously. This was a painstakingly long task, but it seems to have paid off (or at least, I hope).

The rationale for the naming convention was to create an easy way to parse out the different identifiers of the aidstation names and be able to create different columns based on that when converting the data to long format.

<DIV id="wrangling">

##   Process of Data Wrangling   ##

</DIV>

Below are the steps and different methods I used to get the data into the format that I could use.

<DIV id="data-massage">

### Massaging the Data ###

</DIV>

![Sample of some raw data](/images/WSER-EDA/wserRaw_sample.png)

After compiling the different years, I was able to use MS excel and paste together all the runners, and their information for each aid station and finish times for all the years. From there, I then edited a few columns that seemed to have a different formatting than the others, and was able to create a CSV file that I could then pull in with R.

The data still needs quite a bit of wrangling - to make it easy, you can see a short list of tasks that need to be accomplished.

-   Convert wide data into long format
-   Parse out the aid station information into both a time and place column, along with its value
-   Convert all years' time value for aid stations into time elapsed (except for 2010, 2012-2016)
-   Add column for DNF (*Did Not Finish*) - If racer does not have a final time
-   Remove NA rows for aid stations and retain only `time_in` data
-   Rank overall finishing place, place into aid stations, and pace into aid stations
-   Convert back to wide

From here, the data will be in a format much more conducive to analyzing and doing what we want. Additional information that would be nice, but seemingly hard to come by, is the elevation change between each aid station. The reason for this challenge is that the course would change from year to year and therefore it is not always the same. The website has some maps available though, so I may be able to plot up the course and extract that data -- may be an idea for a future project with this.

<DIV id="wrangling-code">

### Code for Wrangling

</DIV>

One of the major steps for wrangling will be parsing out teh

Steps:

1.  Gather all the aid station columns using `tidyr::gather` to make a really long and skinny dataset
2.  We'll use some *REGEX* magic to split the aid station name into 3 columns using `tidyr::extract` (that expression took me a long time to come up with!)
    -   `t16_in` --&gt; \[`t`, `16`, `in`\]
3.  Create separate columns based on whether the first letter was a `t` or `p` with `tidyr::spread`
4.  Reformat a few columns
5.  Remove empty aid station times since it doesn't tell us anything

<pre><code class="r">

    ## Specify columns for aidstations
    keycol <- 'AS_time_name'
    valuecol <- 'AS_time'

    ## Use the underscore (_) to easily select headers
    gathercols <- colnames(wser)[grep('_', colnames(wser))]

    ### A regex expression is used to group the titles in the way we want

    wserLong <- wser %>%
      gather_(keycol, valuecol, gathercols) %>% 
      extract_(keycol, c("AS_indicator", "AS_distance", "AS_InOut"),
               "(^.)([0-9.]*)\\_([:alpha:]*)") %>% 
      spread(AS_indicator, AS_time)

    # Rename Column Names for p --> AS_Place, t --> AS_Time
    colnames(wserLong)[which(names(wserLong) == "p")] <- "AS_Place"
    colnames(wserLong)[which(names(wserLong) == "t")] <- "AS_Time"

    # Reformat a few of the columns, remove empty aid station values
    wserLong$AS_distance <- as.double(wserLong$AS_distance)
    wserLong$AS_Place <- as.integer(wserLong$AS_Place)
    wserLong$Year <- as.factor(wserLong$Year)
    wserLong$Place <- as.integer(wserLong$Place)
    wserLong$First.Name <- str_trim(wserLong$First.Name)
    wserLong <- wserLong %>% filter(!is.na(AS_Time)) %>% arrange(Year, Place, Last.Name)

</code></pre>

Now the top of our data looks like this:

<pre><code class = "r">

    ## Random Sample of 12 rows of data
    wserLong[sample(nrow(wserLong), 12),]

    ##        Year Place   Last.Name First.Name Bib Sex Age          City State
    ## 64838  2000    85    Ianacone       Eric  10   M  53          <NA>  <NA>
    ## 49154  1996   241     Frankum      Betty 176   F  57          <NA>  <NA>
    ## 88172  2005   164      Nelson     Gordon 307   M  46          <NA>  <NA>
    ## 76744  2002   192   Humphreys       Mark 218   M  42          <NA>  <NA>
    ## 90337  2005   292    Longwith        Jim 263   M  59          <NA>  <NA>
    ## 59180  1998   270       Brown       Will 107   M  51          <NA>  <NA>
    ## 146723 2015   152 Westergaard      Danny  91   M  56 Rolling Hills    CA
    ## 46060  1996    56    Ianacone       Eric 224   M  49          <NA>  <NA>
    ## 137819 2014    90      Pittam      Byron  47   M  31      Berkeley    CA
    ## 127    1986    10   Stevenson       Dave  13   M  33          <NA>  <NA>
    ## 22779  1991    80    Crawford      Terry  87   F  42          <NA>  <NA>
    ## 8671   1988    18     Daniels      Roger 130   M  52          <NA>  <NA>
    ##        timeFinish timeElapsed AS_distance AS_InOut AS_Place AS_Time
    ## 64838     7:22:01    26:22:01        16.0       in      175    7:25
    ## 49154        <NA>        <NA>        29.7      out      337   13:09
    ## 88172        7:42    26:42:24        70.7       in      238   23:24
    ## 76744     9:47:30    28:47:30        43.3       in      119   13:51
    ## 90337       10:24    29:24:45        43.3       in      297   15:19
    ## 59180        <NA>        <NA>        47.8       in      335   19:07
    ## 146723    8:36:32    27:36:32        16.0       in      120 3:27:00
    ## 46060     4:26:20    23:26:20        62.0       in       74   18:38
    ## 137819    4:03:02    23:03:02        23.8       in       77 4:33:00
    ## 127       0:29:21    19:29:21        85.2       in       10   20:58
    ## 22779     4:34:32    23:34:32        29.7      out      105   11:22
    ## 8671      0:25:42    19:25:42        16.0       in        4    7:30

</code></pre>

<DIV id="time-configuring">

### Configuring Time

</DIV>

We'll make a few different transformations below to convert some of the time fields that are formatted as `chr` into `POSIXct`. From there, the time can be converted into the number of minutes and formatted as `num`, so much easier to deal with.

<pre><code class="r">

    # Convert Aid Station Splits from Time to a Number
    wserLong$AS_Time_NUM <- (as.numeric(as.POSIXct(paste("2000-01-01", 
                                                         wserLong$AS_Time), 
                                                   format = "%Y-%m-%d %H:%M")) -
      as.numeric(as.POSIXct("2000-01-01 0:0:0")))/60

    ## Change pre-2010 split times to "Time Elapsed"
    wserLong$YearInt <- as.numeric(levels(wserLong$Year))[wserLong$Year]

    wserLong <- wserLong %>% mutate(AS_Time_NUM = 
                                      ifelse(AS_distance >= 55 & 
                                               AS_Time_NUM <= 690 & 
                                               YearInt < 2010,
                                             AS_Time_NUM + 1440,
                                             AS_Time_NUM))

    # Times over 24 hours (recorded as time elapsed) return NA - fix
    wserLong <- wserLong %>% 
      mutate(AS_Time_NUM = ifelse(!is.na(AS_Time_NUM),
                                  AS_Time_NUM,
                                  sapply(strsplit(AS_Time, ":"),
                                         function(x) {
                                           x <- as.numeric(x)
                                           x[1]*60 + x[2] + x[3]/60
                                          })))

    # Convert Time Elapsed field into number
    wserLong <- wserLong %>% 
      mutate(timeElapsed_MIN =
               ifelse(is.na(timeElapsed),
                      NA,
                      sapply(strsplit(timeElapsed, ":"),
                             function(x) {
                               x <- as.numeric(x)
                               x[1]*60 + x[2] + x[3]/60
                             })))
</code></pre>

<DIV id="time-fields">

### Additional Time Fields

</DIV>

One of the challenges with this data is the method the race organization recorded the time into the aid stations, and how it changed over the years. Below is a good representation of how the time into an aid station changed from "Time of Day" to "Time Elapsed".

<pre><code class="r">

    ## Setup Custom DF
    wsertime <- wserLong %>% 
      filter(AS_distance == 16 | AS_distance == 15.2) %>% 
      select(Year, Place, timeFinish, timeElapsed, AS_distance:timeElapsed_MIN)

    ggplot(wsertime, aes(x = Year, y = AS_Time_NUM)) +
      geom_point(position = position_jitter(width = 1), alpha = 0.2, col = '#b0bcbf') +
      geom_point(stat = 'summary', 
                 fun.y = mean, 
                 size =2) +
      labs(x = "Year of Race",
           y = "Time into Aid Station [minutes]",
           title = "Aid Station Timing (Time of Day + Time Elapsed)\n2nd AS") +
      scale_y_continuous(breaks = seq(0, 500, 50), limits = c(0, 500)) +
      theme_fivethirtyeight() +       
      theme(axis.title = element_text(),
            axis.title.y = element_text(),
            panel.grid.major.y = element_line(linetype = "dashed"),
            axis.text.x = element_text(size = 8, angle = 30))
</code></pre>

![Aid Station Time-1](/images/WSER-EDA/Aid%20Station%20Time-1.png)

It's pretty obvious that in 2010, they changed the method of recording. Since the race starts at 5 AM, we can adjust all aid station times from years prior to be 5 hours less than its current value.

<pre><code class="r">

    ## Change pre-2010 split times to "Time Elapsed"
    wserLong$YearInt <- as.numeric(levels(wserLong$Year))[wserLong$Year]
    wserLong <- wserLong %>% 
      mutate(AS_Time_NUM = ifelse(YearInt < 2010,
                                  AS_Time_NUM - 300,
                                  AS_Time_NUM))

    # Replot of information
    wsertime <- wserLong %>% 
      filter(AS_distance == 16 | AS_distance == 15.2) %>% 
      select(YearInt, Place, timeFinish, timeElapsed, AS_distance:timeElapsed_MIN)

    ggplot(wsertime, aes(x = YearInt, y = AS_Time_NUM)) +
      geom_point(position = position_jitter(width = 1), alpha = 0.2, col = '#b0bcbf') +
      geom_point(stat = 'summary', 
                 fun.y = mean, 
                 size =2) +
      labs(x = "Year of Race",
           y = "Time into Aid Station [minutes]",
           title = "Aid Station Timing - 2nd AS (Time Elapsed)") +
      scale_y_continuous(breaks = seq(0, 500, 50), limits = c(0, 500)) +
      scale_x_continuous(limits = c(1986, 2016), breaks = seq(1986, 2016, 1)) +
      theme_fivethirtyeight() +
      theme(axis.title = element_text(),
            axis.title.y = element_text(),
            panel.grid.major.y = element_line(linetype = "dashed"),
            axis.text.x = element_text(size = 8, angle = 40))

</code></pre>

![Adjusted Plot of 2nd aid station](/images/WSER-EDA/Adjusted%20Plot-1.png)

<DIV id="feature-adding">

## Adding Features

</DIV>

With these values adjusted, I can now look into categorizing our data a little more. First we'll add in a binary variable for whether or not a runner DNF ('Did Not Finish'), and then another one for their age group.

<pre><code class="r">

    # Setup for Age Group bins
    ageBin <- c(0,20,30,40,50,60,70,200)

    # Transform for Age Groups

    # Initialize Grade Rank Column
    wserLong$ageGroup <- 0

    # Step through list of grades and categorize
    for (i in 1:(length(ageBin) - 1)) {
      wserLong <- wserLong %>% 
        mutate(ageGroup =
                 ifelse((ageGroup == 0 & Age >= ageBin[i] & Age < ageBin[i + 1]),
                        i,
                        ageGroup))
    }

    # Transform for DNF
    wserLong <- wserLong %>% 
      mutate(DNF = ifelse(is.na(timeElapsed), 1, 0))

    # DNF as factor and a label
    wserLong$DNF <- factor(wserLong$DNF, levels = c("1", "0"), labels = c("DNF", "Finished"))

</code></pre>

Now that the timing is looking fairly consistent, what does the timing look like for each aid station throughout the years?

<pre><code class="r">

    ggplot(wserLong, aes(x = AS_distance, AS_Time_NUM)) +
      geom_point(position = position_jitter(width = 0.5), 
        alpha = 0.2, col = '#b0bcbf') +
      geom_point(stat = 'summary', fun.y = mean, size = 2) +
      labs(x = "Distance from Start [miles]",
           y = "Time since Start [minutes]",
           title = "Time to Aid Stations") +
      scale_x_continuous(breaks = seq(0, 100, 5), limits = c(0, 100)) +
      scale_y_continuous(breaks = seq(0, 2500, 250), limits = c(0, 2500)) +
      theme_fivethirtyeight() +
      theme(axis.title = element_text(),
            axis.title.y = element_text(),
            panel.grid.major.y = element_line(linetype = "dashed"),
            axis.text.x = element_text(size = 9),
            plot.title = element_text(size = 12))

</code></pre>

![Aid station timing](/images/WSER-EDA/as%20times-1.png)

<DIV id="troubleshooting">

### Troubleshooting

</DIV>

When I first created the above plot, something looked funky... Instead of finding some way to plot all these in a comprehensible way onto one plot, I plotted each year with the Distance from Start (`AS_distance`) and Time to Reach Distance (`AS_Time_NUM`) to see if the issue is with one year, or multiple years.

Instead of using `ggplot`, I instead just used `plot` since it's easier to output those files in a `for` loop.

After looking through the output files, it's clear the problem is with 2016. Here's a cool plot showing all the plots in aggregate, and isolating 2016 with color. I know it doesn't definitively show where the problem is, but trust me, I looked. I also wanted to use a plot function I saw on [FlowingData](flowingdata.com).

***
\*\*\*

After investigating, it turns out there was an issue with time formatting on some of the 2016 values. I had to change the source file and reload the CSV. That being the case, the issue should not show up below, so I have replaced it with a static image of the plot when I had the issue.

The fixed plot is displayed second, but when I initially addressed this issue, I was seeing the plot directly below.

![WSER Time to read AS By Year](/images/WSER-EDA/TimeToReachAS_byYear.png)

![Aggregate plot with Issues](/images/WSER-EDA/Aggregate%20Plot%20with%20Issue-1.png)

<DIV id="wide-format">

## Reformatting to Wide

</DIV>

Now that the data is fixed, the next goal is to have two forms of the data - long and wide. I already have the long form, and that will be helpful with displaying some of the plots, but for the model that I want to create, it will need to be back in wide format.

The long format is better suited for comparing aggregate times into each aid station but almost irrespective of the runners. The wide format puts everything back together for each individual runner. Our ultimate goal is still to categorize by place, gender, age, along with figuring if the time between aid stations tells us anything.

<DIV id="long-to-wide">

### Long ---&gt; Wide

</DIV>

<pre><code class="r">

    ## Only interested in "Time In"
    wserLong <- wserLong %>% filter(AS_InOut != "out") %>% select(-AS_InOut)

    ## Re-Rank All AS Splits
    wserLong <- wserLong %>% group_by(Year, AS_distance) %>%
      mutate(AS_Place = rank(AS_Time_NUM,
                             na.last = "keep",
                             ties.method = "random")) %>% 
      ungroup()

    ## Set up key column and identifiers
    wserWide <- wserLong %>% 
      mutate(key = paste(Year, Place, sep = "-"),
             AS_P = paste(AS_distance, "P", sep = "_"), 
             AS_T = paste(AS_distance, "T", sep = "_")) %>% 
      select(-AS_distance, -AS_Time)

    ## Reorganize DF
    wserTemp <- wserWide[c(18,1:11,14:17)] %>% distinct(key, .keep_all = TRUE)

    ## Create 2 temp DF for converting to wide
    widePlace <- wserWide %>% select(key, AS_Place, AS_P)
    wideTime <- wserWide %>% select(key, AS_Time_NUM, AS_T)

    ## Spread varying data
    widePlace <- widePlace %>% spread(AS_P, AS_Place)
    wideTime <- wideTime %>% spread(AS_T, AS_Time_NUM)

    ## Merge back into single dataframe
    wserP <- merge(wserTemp, widePlace, by = "key")
    wserT <- merge(wserTemp, wideTime, by = "key")

    ## Tidy up our workspace
    rm(wserTemp)
    rm(widePlace)
    rm(wideTime)
    rm(wsertime)
    rm(wserWide)
    rm(yearDF)

</code></pre>

There is now 3 sets of tidied data, one being long and 2 wide forms that capture both time and place independently. The wide type of data with `time` can easily be merged with `place` if needed, but it is a bit easier to compare by keeping them separate.

<DIV id="finish-times">

## Exploring Finish Times

</DIV>

Because the data is nicely configured into the wide format, that can now be used to compare finish times. To further the exploration, the data can also be used to look at finish times amongst different groups - gender, age, and year. Additionally, the ability to compare times between aid stations exists and potentially revealing more information from what that shows too.

<DIV id="overall-times">

### Finish Times Overall

</DIV>

<pre><code class="r">

    ggplot(subset(wserT, DNF == "Finished"), aes(x = Place, y = timeElapsed_MIN, col = Year)) +
      geom_point(alpha = 0.75) +
      labs(x = "Finishing Place",
           y = "Elapsed Time from the Start [min]",
           title = "Total Time to Finish") +
      scale_color_manual(values = colPal) +
      theme_minimal() +
      theme(axis.title = element_text(),
            axis.title.y = element_text(size = 9, face = "bold"),
            panel.grid.major.y = element_line(linetype = "dashed"),
            axis.text.x = element_text(size = 9),
            plot.title = element_text(size = rel(1.5)),
            plot.background = element_rect(fill = '#a8a5a6', color = "#a8a5a6"))

</code></pre>

![Total Finish Times - 1](/images/WSER-EDA/Total%20Finish%20Times-1.png)

Already, it has an interesting pattern. There seems to be some sort of plateau around 1400 minutes. My guess is that is at 1440 minutes, or 24 hours. There is a chance that something with the data is funky, but it may be simpler than that... The goal of making it under 24 hours is a huge driving factor to finish, especially when getting very close to that cutoff point.

![Over-Under 24 hours](/images/WSER-EDA/Under%2024%20Hours-1.png)

What about coloring by gender to visually show how finishing times may differ by gender?

<pre><code class="r">

    ggplot(subset(wserT, DNF == "Finished"), 
           aes(x = Place, y = timeElapsed_MIN, col = Year)) +
      geom_point() +
      facet_grid(.~Sex) +
      scale_color_manual(values = colPal, guide = FALSE) +
      labs(x = "Finishing Place",
           y = "Elapsed Time from the Start [min]",
           title = "Finishing Time - by Gender") +
      theme_fivethirtyeight() +
      theme(axis.title = element_text(),
            axis.title.y = element_text(size = 9, face = "bold"),
            axis.text.x = element_text(size = 9),
            panel.background = element_rect(fill = '#a8a5a6', color = "#a8a5a6"))

</code></pre>

![Finish by gender](/images/WSER-EDA/finish%20by%20gender-1.png)

There is a big difference between the amount of males that compete versus the number of females that compete. How has that changed over the years?

<pre><code class="r">

    ggplot(wserT, aes(x = YearInt, fill = Sex)) + 
      geom_histogram(binwidth = 1) +
      scale_fill_manual(values = c("#d8b365", "#5ab4ac")) +
      labs(x = "Year",
           y = "No. of Participants",
           title = "Entrants per Year - by Gender") +
      scale_x_continuous(breaks = seq(1985, 2020, 2), limits = c(1985, 2017)) +
      theme_fivethirtyeight()
  
</code></pre>

![Gender over the years](/images/WSER-EDA/gender%20over%20the%20years-1.png)

Not much...

Futher, the data can be categorized by the finish times and by age category.

<pre><code class="r">

    ggplot(subset(wserT, !is.na(ageGroup) & DNF == "Finished"), 
           aes(x = factor(ageGroup), y = timeElapsed_MIN, fill = factor(ageGroup))) +
      geom_boxplot() +
      scale_fill_manual(values = col2Pal,
                        guide = FALSE) +
      scale_x_discrete(breaks = seq(1,7,1), labels = ageLabels) +
      labs(x = "Age Range",
           y = "Time to Finish [min]",
           title = "Finishing Times by Age Grouping") +
      theme_fivethirtyeight() +
      theme(axis.title = element_text(face = "bold", size = 9),
            axis.title.y = element_text(),
            panel.grid.major.y = element_line(linetype = "dashed"),
            axis.text.x = element_text(size = 9),
            panel.background = element_rect(fill = '#a8a5a6', color = "#a8a5a6"))

</code></pre>

![Age Group Finishers](/images/WSER-EDA/unnamed-chunk-1-1.png)

> Death Before DNF

A slogan amongst many ultra-runners is *Death Before DNF* meaning that they'd rather perish than throw in the towel and not finish a race. This, however, is unrealistic, as a 100 mile race will always see DNFs. Perhaps, when faced with death or needing to abandon the race, runners make the tough call and decide to live?

![DNF Histogram](/images/WSER-EDA/DNF%20Occurence-1.png)

<DIV id="ranking-deciles">

### Finishing Deciles

</DIV>

<pre><code class="r">

    # Create Long DF containing decile rank of for pace grouped by:
    #   1.  Years
    #   2.  Aid Stations (distance)
    #   3.  Overall

    wser.long <- wserLong %>% 
      mutate(qRank.overall = ntile(timeElapsed_MIN, 10),
             pace.overall = timeElapsed_MIN/100) %>% 
      group_by(distFact = factor(AS_distance)) %>% 
      mutate(qRank.as = ntile(AS_Time_NUM, 10), 
             pace.as = AS_Time_NUM/AS_distance) %>% 
      ungroup() %>% 
      group_by(Year) %>% 
      mutate(qRank.year = ntile(timeElapsed_MIN, 10)) %>% 
      ungroup()

    ## Add Index for which Aid Station number
    iAS <- unique(wser.long$AS_distance)
    iAS <- sort(iAS)
    re.ind <- function(x) {
      which(iAS == x, arr.ind = TRUE)
    }

    wser.long$AS.Num <- sapply(wser.long$AS_distance, re.ind)

    ## Add in key for an individual runner per year
    wser.long <- wser.long %>% 
      mutate(key = paste(Year, Place, sep = "-"))

    ## Determine distance, time, and pace between current and previous aid station
    ### First we'll distinguish between the aid station number overall (AS.Num), and aid station they hit for that year (AS.count)

    wser.long$AS.count <- as.integer(ave(wser.long$key, wser.long$key, FUN = seq_along))

    ## Some additional dataframes as reference for the Aid Stations identifiers
    aidStation.dict <- data.frame("AS.Distance" = iAS, "Index.Number" = seq(1:22))
    aidStation.byYear <- wser.long %>% 
      group_by(Year) %>% 
      distinct(AS.Num, AS_distance) %>% 
      select(Year, AS.Num, AS_distance) %>% 
      ungroup()

    ### Set up the differential times and distances between each aid station
    ### Use of IFELSE to set the 1st aid station reached as the differential as well since the differential distance is subtracted from 0 (the start).

    ### Use of data.table::shift function to return previous value in a row's value to get differential
    wser.long <- wser.long %>% group_by(key) %>% 
      mutate(diff.dist = ifelse(AS.count == 1,
                                AS_distance,
                                AS_distance - shift(AS_distance, 1L, type = "lag")),
             diff.time = ifelse(AS.count == 1,
                                AS_Time_NUM,
                                AS_Time_NUM - shift(AS_Time_NUM, 1L, type = "lag")),
             diff.pace = diff.time/diff.dist) %>% 
      ungroup()

    ## Format DNF as a factor

    # Create Wide DF containing decile rank and pace for:
    #   1.  Years (Rank only)
    #   2.  Overall

    wser.wide <- wserT %>% 
      mutate(pace.overall = timeElapsed_MIN/100,
             qRank.overall = ntile(timeElapsed_MIN, 10)) %>% 
      group_by(Year) %>% 
      mutate(qRank.year = ntile(timeElapsed_MIN,10)) %>% 
      ungroup()

    ## Tidy up our Workspace
    rm(wserLong)
    rm(wserT)
    rm(wser)

</code></pre>

Another important piece of information to explore before doing our final analysis and prediction model is ranking finishers by decile. There are few different thoughts here on how to categorize this.

1.  Use finishing times from all years to calculate decile times
2.  Calculate deciles per year and see how those values changed
3.  Rank deciles of each runner into each aid station, whether or not they DNF'd

A note on the variable naming convention used: 

-    `qRank.overall` is the rank within the decile a runner had when compared to runners at any year. 
-    `qRank.as` refers to the decile rank a runner had into an individual aid station 
-    `qRank.year` is rank for a runner only compared to other runners within the same year -    `pace.overall` is the pace (minute per mile) a runner had over the entire race 
-    `pace.as` is the pace a runner held between the distance from the previous aid station to the one at which this measurement is being calculated

These different features can tell a little bit more on the race and how each runner performed.

A basic plot to explore would be how a runner's overall time throughout the race contributed to their final decile ranking. That is shown below.

<pre><code class="r">

    ggplot(wser.wide, aes(x = qRank.overall, y = timeElapsed_MIN)) +
      geom_bar(stat ='summary', fun.y = mean) +
      geom_line(stat = 'summary', fun.y = mean, col = "red", lwd = 1.3) +
      labs(x = "Decile",
           y = "Time to Finish [hrs]",
           title = "Mean Finishing Times") +
      scale_y_continuous(breaks = seq(0, 2000, 150), 
                         labels = seq(0, 2000, 150)/60) +
      scale_x_continuous(breaks = seq(0,10,1), labels = seq(0,10,1)) +
      theme_fivethirtyeight(base_size = 10) +
      theme(axis.title = element_text(face = "bold", size = 9),
            axis.title.y = element_text(),
            panel.grid.major.y = element_line(linetype = "dashed"),
            axis.text.x = element_text(size = 9))

</code></pre>

![Overall Decile Ranking](/images/WSER-EDA/overall%20decile%20rank-1.png)

The above plot raised a few questions and while a runner may have fared well in their own year, did that hold true when compared against all years? As in, were there faster years, or years where a lot of top notch talent showed up?

<pre><code class="r">

    pal2 <- colorRampPalette(c("#e5f849", "#f0111c", "#470d03"), bias = 0.5)
    color2 <- pal2(30)

    ggplot(wser.wide, aes(x = factor(qRank.year), y = factor(qRank.overall), col = Year)) +
         geom_point(position = "jitter", alpha = 0.9) +scale_x_discrete(breaks = seq(0, 10, 1)) +
      labs(x = "Decile Rank per Year",
           y = "Decile Rank Overall",
           title = "Comparing Finishing Time Deciles") +
      scale_color_manual(values = color2) +
      theme_fivethirtyeight() +
      theme(axis.title = element_text(face = "bold", size = 9),
            rect = element_rect(fill = "#586e75"),
            axis.title.y = element_text(),
            panel.grid.major.y = element_line(linetype = "dashed"),
            axis.text.x = element_text(size = 9),
            panel.background = element_rect(fill = '#657b83', color = "#657b83"),
            legend.background = element_rect(fill = '#657b83', color = "#657b83"),
            legend.key = element_rect(fill = '#657b83', color = "#657b83"),
            legend.position = "right",
            legend.direction = "vertical")

</code></pre>

![Comparing Deciles - scatter plot](/images/WSER-EDA/comparing%20deciles-1.png)

To interpret this plot, the 1:1 ratio means a runner was ranked the same in their year of running the race versus compared to all years. Data points below the 1:1 line show a runner who was maybe faced with stiff opposition their year and finished lower in their decile for their respective year, but would have been ranked better (lower) when looking at all years. The opposite is true for being above the line - ranked well for current year, but were slower overall.

I had trouble figuring out the proper colors to accurately represent the data. However, what can be gleaned from the coloring of the years is the yellow-ish colors representing the early years seems to have had faster slower runners when compared with all runners. Actually, during the mid-90s (orange) seemed to have the slowest runners compared to all years.

Additionally, there are a lot of darker colors at the bottom, in deciles 1 and 2 for the rank per year.

A few thoughts on the observations above - the initial years (pre-1990s) may have drawn in top talent throughout the entire competition; however, as the race began to attract **normal** people, along with much faster **elites**, there started to be more of a divide between the top deciles (1 and 2), and the lower deciles.

<DIV id="aid-station-specific">

### Aid Station Specific

</DIV>

Last part to look at is the how the time and speed changed between aid stations, and between decile finishing groups. This should help visualize how the runners sustain themselves through the race and if different groups maybe keep a specific pace throughout, or not.

The Western States course has had to change many times due to adverse conditions (snow, fires, etc.) and thus do not have a completely consistent aid station setup throughout all 30 years the race has been going on. With increased manpower helping the race function, more data
was able to be collected as well.

With that, we can see which distances the aid stations are most used, and some of which were rarely used.

![AS Histogram](/images/WSER-EDA/aid%20station%20histogram-1.png)

By comparing the different aid stations and associated paces for each runner into each, a nice graph can be generated to see how each runner moved across each. To go a step further and splitting the graphs into their own decile groups, a nice representation of the trend for each group can be established. This graph below alone is worthy of further in-depth analysis. Something further that would be worthwhile investigating is how the slopes of each decile group varies, and why.

<pre><code class="r">

    ggplot(wser.long, aes(x = AS_distance, y = pace.as, group = qRank.as)) +
      geom_point(stat = "summary", fun.y = mean) +
      facet_grid(.~qRank.as) +
      labs(x = "Distance from Start",
           y = "Pace [min/mile]",
           title = "Pace into each Aid Station") +
      geom_smooth(method = "lm") +
      theme_fivethirtyeight() +
      theme(axis.title = element_text(face = "bold", size = 9),
            rect = element_rect(fill = "#586e75"),
            axis.title.y = element_text(),
            panel.grid.major.y = element_line(linetype = "dashed"),
            axis.text.x = element_text(size = 7, angle = 30),
            panel.background = element_rect(fill = '#657b83', color = "#657b83"),
            legend.background = element_rect(fill = '#657b83', color = "#657b83"),
            legend.key = element_rect(fill = '#657b83', color = "#657b83"),
            legend.position = "right",
            legend.direction = "vertical")

</code></pre>

![AS Decile Pace](/images/WSER-EDA/aid%20station%20pace%20by%20decile-1.png)

Some very interesting results. As to be expected, each decile rank is slightly moved up on the pace scale, and the overall trend of the group seems to be trending steeper, as in their pace is slower with each aid station. This is unfounded, just a thought on looking at the graph.

There seems to be a few points that really stick out at the bottom of each group - I'm not sure why someone would have such a faster time, while being in the same decile group of much slower persons. My guess is that must have been from a specific race year where the course changed, so there weren't many groups to fit into bins?

Another way to look at the data is to take the pace over each section of the race for all years and all runners. What the graph below is showing is the pace for each person's entire race (whether they DNF'd or not), along with an average pace for each overall placing.

For example - the line at 125 along the x-axis has the data points for each year where a runner finished at place 125, and the grey dots represent the pace for every recorded point along the way (aid station recordings). While the black dot at 125 is the average pace, for all years, through all aid stations.

![Pace vs Place](/images/WSER-EDA/Pace%20vs%20Place-1.png)

An interesting point here is where the overall place is greater than 350, but a runner's pace will have been 7.5 min/mile into an aid station, as that doesn't make sense. However, the fast paces in high overall places are either runners who wanted to go really fast at one point, and then barely made it, or just didn't finish.

We can do a similar plot to above, but this time color by their decile rank into each aid station, along with separating those that finished and those that received a DNF.

I also added a default `geom_smooth` line.

![Ranked pace vs place](/images/WSER-EDA/Pace%20vs%20Place%20by%20Rank-1.png)

As would be expected, runners finishing near the top of the pack (overall place is near 1), have a more solid yellow vertically, meaning they were consistently in the top top decile throughout the whole race. Towards the back of the pack, in those finishing 250 or greater, there are a few runners that tried to go fast (yellow dots) but ultimately finished towards the end.

The same plot as above can be created, but this instance only using finishing times instead of aid station time.

![Overall Place vs Pace](/images/WSER-EDA/Pace%20vs%20Place%20Overall-1.png)

Because the rank was determined by a runners overall performance, it should come as no surprise that the rank corresponds to a typical pace with little to no abnormalities. Since `pace` is `Overall Time`/`100 miles`, the `y-axis` is really a transformation of the same plot shown farther up.

<DIV id="correlation">

### CorrPlot

</DIV>

It is important to take into account which variables are highly correlated, and which are not. Based on the feature set created, a corrplot should help to visualize the variables that closely related.

To use `corrplot`, there needs to be a dataframe that contains only numbers of the variables we want to compare. Currently there are two dataframes - `wser.wide` and `wser.long`. `wser.wide` tells information on total finishing times (and thus includes only those that finished), while `wser.long` is better suited for aid station timing and placing.

<pre><code class="r">

    wser.cor.AS <- wser.long %>% 
      mutate(sex.male = ifelse(Sex == "M",1,0),
             DNF = ifelse(DNF == "Finished", 0, 1)) %>% 
      select(YearInt, Place, sex.male, Age, AS_distance, 
             AS_Place, AS_Time_NUM, ageGroup, DNF, qRank.as, pace.as,
             AS.count, diff.pace) %>% 
      filter(!is.na(Age) & !is.na(ageGroup))

    wser.cor.finishers <- wser.wide %>% 
      filter(DNF == "Finished") %>% 
      mutate(sex.male = ifelse(Sex == "M",1,0),
             DNF = ifelse(DNF == "Finished", 0, 1)) %>% 
      select(YearInt, Place, sex.male, Age,
             timeElapsed_MIN, ageGroup, 
             pace.overall, qRank.overall, qRank.year) %>% 
      filter(!is.na(Age) & !is.na(ageGroup))

    # First, the Correlation plot of Aid Station timing
    corrplot(cor(wser.cor.AS), method = "shade",
             type = "lower", order = "hclust",
             title = "Correlation Plot - Aid Station Timing")

</code></pre>

![Corrplot-1](/images/WSER-EDA/corrplot-1.png)

<pre><code class="r">

    corrplot(cor(wser.cor.finishers), method = "shade",
             type = "lower", order = "hclust",
             title = "Correlation Plot - Finishers", mar = c(1,1,1,1))

</code></pre>

![Corrplot-2](/images/WSER-EDA/corrplot-2.png)

The two plots above will now be used to help determine codependency between variables along with figuring out which variables may be best to use in the final model.

<DIV id="final-thoughts">

Final Thoughts

</DIV>

A lot of the visualizations shown above tell a lot about the race and how racers finish or don't finish, and some of the different measures we can look at. Truly, the data has a lot to tell, and many of the graphs could warrant their own in-depth analysis individually. Unfortunately, I have to focus my time and will be going into a final analysis of what determines if someone will finish and how their placing throughout the race will determine their final outcome.

If you have any thoughts to share or other ways to look at the data, email me (via link on site) or leave a comment below. Otherwise, look for my next post that will go into a more in-depth analsyis and predictive model.