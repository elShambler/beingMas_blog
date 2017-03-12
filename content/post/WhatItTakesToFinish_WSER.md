+++
date = "2017-03-09T12:54:11-05:00"
title = "What It Takes To Finish Western States"
slug = "FinishingModel_WSER"
topics = ["Running"]
tags = ["data", "trail races"]
draft = true

+++

## Overview of the Project and Western States

For the past year or so, whatever spare time I had outside of work was spent towards learning a new set of skills in the field of Data Science, and working on a capstone project that I am now proud to show below.

I wanted to focus on a project in an area that I have a passion for, along with one that could lead to further applications. I knew I needed a good set of data to begin with, and then be able to glean information that would be worthwhile predicting some outcome or variable. I ultimately decided to go for data surrounding ultramarathons, specifically 100 mile races.

There are a few different sites that I thought to pull data from (UltraSignup, Strava, RealEndurance, or Trail Runner Project), but I wasn't able to properly scrape the data. Instead, I was able to take data from race websites' results page, and focused on what is the gold standard for 100-mile races - the Western States Endurance Run (WSER).

The race has some interesting lore behind it, but it is consistently one of the most competitive races year after year ([it was ranked #1 for 2016](https://www.ultrarunning.com/featured/2016-most-competitive-fields/), [and again in 2015](https://www.ultrarunning.com/featured/2015-most-competitive-fields/) by [ultrarunning.com](ultrarunning.com)), and was also the very first 100-mile ultramathon. It is most often a metric to compare other 100-mile races to. Therefore, I figured it would be a good race to evaluate and analyze for my project.

### Ultramarathon 101

While most may be familiar with a traditional marathon of 26.2 miles, an ultramarathon is any foot race over 26.2 miles. More often than not, these races take place on the trails of state and federal lands (sometimes looped, other times point-to-point), and often involve lots of vertical climbing and descending to go with the horizontal distance (known as "vert" for vertical gain).

In terms of physiology, for a 100-mile race, the body does not contain enough readily available resources for the mind and muscles to last the duration of these events. Depending on the terrain, these races can take anywhere from 14 hours for the superhumans, to 24 and 36 hours and beyond. Aid stations are setup along the course to fuel and hydrate runners, and serve as a chance for rest if needed. The aid stations can also double as check points for timing and to verify a runner is on course, or offer them a chance to drop if they deem necessary.

Lastly, while the point of a race is to go as fast as possible, 100 miles is a long distance and requires smart planning and execution, otherwise finishing may not even be an option. For some runners that may mean to run up the hills and spend little time at aid stations, while for others it is wiser to power-hike up a hill and conserve energy for flatter and downhill sections.

One disclaimer is this project is attempting to break down the different factors available and use those to best predict whether or not a runner will reach the finish line. I personally have experience running ultras and understand there is a lot more that can determine whether a runner will finish aside from what I outline below. Keep in mind, I am working with a limited data set.

## Origin and Wrangling of Data

The WSER site, where I pulled the data from,  not only includes the final times and overall places for each race, but also has a detailed results page that includes all splits into the aid stations for each year. 

![WSER Splits Page](img/wserSplitsPage.png "WSER Page for Splits Data")

The format of the file was not always consistent and required a fair amount of wrangling.

![Example of Splits Data](img/wserRaw_sample.png "Examples of Different Years Splits Data")

One of the first steps was determining which aid stations were used in a given year, since they varied. I was able to normalize each aid station based on its name, distance from the start, and whether the measurement was for a runner coming into an aid station, or leaving. Below is an example of what the final data looked in excel before I exported it to a `*.csv` and imported it to R - the programming language I used to analyze and create a predictive model with.


![Final XLSX Output](img/WSER_full.xlsx.png)


### Formatting in R

Once in R, it was necessary to wrangle the data into a format that would be most conducive to analysis. Information pertaining to the runner's overall stats (name, age, gender, overall time, place, etc.) were formatted in a style that was easy to work with. Information detailing the aid stations was not always in a consistent layout, and required a method to normalize the data. Below is an example of my methodology for how I categorized the aid stations and the information extracted.


AS Name        | Variable Name  | Distance (miles)  |  In or Out   | Type of Measurement
---------------|----------------|-------------------|--------------|---------------------
Red Star Ridge | `t16_in`       | 16                | In           | Time
Red Star Ridge | `p16_in`       | 16                | In           | Place
Duncan Canyon  | `t23.8_in`     | 23.8              | In           | Time
Duncan Canyon  | `p23.8_in`     | 23.8              | In           | Place
Miller's Defeat | `t34.4_in`    | 34.4              | In           | Time
Miller's Defeat | `p34.4_in`    | 34.4              | In           | Place
Miller's Defeat | `t34.4_out`   | 34.4              | Out          | Time
Miller's Defeat | `p34.4_out`   | 34.4              | Out          | Place


Once the dataset was in a somewhat manageable wide format (one runner per year per line), there were still lots of steps to go through. Different challenges included accounting for the way the time was recorded at aid stations on different years (i.e. time of day vs. elapsed time since the start of the race), removing missing data, writing some fun `REGEX`, and doing a bit of feature engineering (adding variables or different measures).

Ultimately, I was able to get data back into a format where each line represents a single racer per year with all relevant information for the racer for that particular race year. The final data frame only contains the time a racer came into an aid station due to the fact that the exit times weren't as consistent as entry times.



### Code for Wrangling

A lot of the legwork involved with this project required writing code in R to wrangle and ultimately perform all the analysis. I have left it out of here, but you can view all the [code for it here](https://github.com/elShambler/springboard/tree/master/CapstoneProject).

\pagebreak

### Final Data Set

Below is an outline of the variables I was able to use for analysis. I am excluding the list of all the aid stations, but have that list created that I'll link to in the future.


Variable Name     | type  |   Explanation
------------------|-------|---------------------------------
`key`             | chr   | Unique ID of year and overall place (e.g. 1995-3)
`YearInt`         | num   | Year of race (1986-2016)
`Year`            | factor| Year of race as a factor
`Place`           | int   | Overall finish place
`Last.Name`       | chr   | Last name of runner
`First.Name`      | chr   | First name of runner
`Bib`             | chr   | Bib number of runner (mostly numbers, some alpha numeric)
`Sex`             | chr   | Male or Female (M or F)
`Age`             | int   | Age of runner for given year
`ageGroup`        | num   | Index number for 7 bins - (0,20,30,40,50,60,70,200)
`timeElapsed_MIN` | num   | Overall time, in minutes, for race (NA if DNF)
`DNF`             | num   | Finish status - "Finished" = 0, "DNF" = 1
`qRank.overall`   | int   | Decile rank based on overall time vs all years
`pace.overall`    | num   | Overall pace for duration of race (assumes distance of 100 miles)
`qRank.year`      | int   | Decile rank based on overall time vs given year
** AID STATIONS **| num   | Time elapsed (in minutes) to reach given aid station
`pace.avg`        | num   | Average pace between each aid station
`pace.sd`         | num   | Standard deviation of the paces between each aid station
`as.max`          | num   | Maximum number of aid stations given runner checked in to
`bibNo`           | num   | Reformatted `Bib` removing the ALPHA characters
`sex.male`        | num   | Reformatted `Sex` - "Female" = 0, "Male" = 1


## Exploratory Data Analysis

It is always important to evaluate the data to ensure, first and foremost, none of the data are abnormal outliers. After that, I wanted to explore the different variables and what they showed, and how they are related. I have a [separate post](http://beingmas.com/2016/11/08/western-states-eda/) on the exploration of the data, but have a included it here as well. In an effort to conserve space, I have supressed the code output so all that is shown are the graphs.


### Timing into Aid Stations

Below are a few of the different graphs that explore what sorts of times runners were hitting going into the second aid station for different years.

```{r Aid Station 2 Timing, echo=FALSE, message=FALSE, warning=FALSE, fig.cap = "Time elapsed for all runners reaching the 2nd aid station"}

# Plot of Times Entering 2nd Aid Station
as2 <- wser.long %>% 
  filter(AS_distance == 16 | AS_distance == 15.2) %>% 
  select(YearInt, Place, timeFinish, timeElapsed, AS_distance:timeElapsed_MIN)

ggplot(as2, aes(x = YearInt, y = AS_Time_NUM)) +
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
        plot.title = element_text(size = 12),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 8, angle = 40))
```

Something to note on figure 04 is how a few years seem to have abnormally fast 2nd aid stations. This is due to snow on the course and the race altering the traditional path, and the 2nd aid station being a bit closer to the start.

Figure 05 builds on the above and plots each aid station for all the years, with the black dot representing the mean for the aid station at the given distance. 

```{r as times, echo = FALSE, message = FALSE, warning=FALSE, fig.cap = "Time elapsed into each aid station"}
ggplot(wser.long, aes(x = AS_distance, AS_Time_NUM)) +
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

```

### Finishing Times

At the outset of this project, the goal was to capture finish times and evaluate that. The project has since evolved, but there are still good data with some interesting trends with regard to total finishing times. Below are a few of the explorations into the different times, categorized a few different ways.

Figure 06 shows a base plot of the total time it took to finish.

```{r Total Finish Times, echo=FALSE, message=FALSE,fig.width = 7, fig.asp = 0.6, warning=FALSE, fig.cap = "Total finishing times for all years"}
pal2 <- colorRampPalette(c("#e5f849", "#f0111c", "#470d03"), bias = 0.5)
color2 <- pal2(30)

ggplot(subset(wser.wide, DNF == 0), aes(x = Place, y = timeElapsed_MIN, col = Year)) +
  geom_point(alpha = 0.75) +
  labs(x = "Finishing Place",
       y = "Elapsed Time from the Start [min]",
       title = "Total Time to Finish") +
  scale_color_manual(values = color2) +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        rect = element_rect(fill = "#586e75"),
        axis.title.y = element_text(),
        plot.title = element_text(size = 11),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 9),
        panel.background = element_rect(fill = '#657b83', color = "#657b83"),
        legend.background = element_rect(fill = '#657b83', color = "#657b83"),
        legend.key = element_rect(fill = '#657b83', color = "#657b83"),
        legend.position = "right",
        legend.direction = "vertical",
        legend.text = element_text(size = 8))

```

The first thing to notice is how there is a change in the shape of the curve about halfway up. The change in shape is the effect of runners trying to hit the 24-hour cutoff (1440 minutes).

By ranking each runner per year based on their overall time compared to all times recorded, a set of 10 quantile groups (deciles) can be established. By categorizing by this factor, the categories are more or less what would be expected (figure 07).

```{r Pace vs Place Overall, echo = FALSE, fig.width = 7, fig.asp = 0.6, warning = FALSE, message = FALSE, fig.cap = "All finishers colored by their ranking"}
color3 <- pal2(10)

ggplot(wser.wide, aes(x = Place, y = pace.overall)) + 
  geom_point(aes(col = factor(qRank.overall)), alpha = 0.6) +
  scale_color_manual("Rank", values = color3) +
  geom_smooth(colour = "black") +
  labs(x = "Place",
       y = "Pace [min/mile]",
       title = "Pace for Finishing Place by Decile Rank") +
  scale_y_continuous(breaks = seq(0,35,2.5)) +
  scale_x_continuous(breaks = seq(0, 350, 25)) +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        rect = element_rect(fill = "#586e75"),
        axis.title.y = element_text(),
        plot.title = element_text(size = 11),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 8, angle = 30),
        panel.background = element_rect(fill = '#657b83', color = "#657b83"),
        legend.background = element_rect(fill = '#657b83', color = "#657b83"),
        legend.key = element_rect(fill = '#657b83', color = "#657b83"),
        legend.position = "right",
        legend.direction = "vertical",
        legend.text = element_text(size = 9))
```

To understand what type of pace is required to get into each of the decile rankings, a separate plot for mean finishing times of each group will be useful. Figure 08 is an example of a take-away that a potential entrant can evaluate and perhaps apply to their training if they wish to make a certain group (without taking anything else into consideration, that is).

```{r overall pace for decile rank, warning = FALSE, echo = FALSE, message = FALSE, fig.cap = "Finishing times for each decile ranking"}
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
```

One last graph (figure 09) that I found rather interesting was comparing a runner's decile rank for the specific year (e.g. in 1995, the 45th place runner ranked in the 2nd decile) vs what their time would have them ranked for all years (e.g. the same runner is now ranked in the 3rd decile).

```{r comparing deciles, echo = FALSE, message = FALSE, warning = FALSE, fig.cap = "Rank in given year vs. overall rank"}
ggplot(wser.wide, aes(x = factor(qRank.year), y = factor(qRank.overall), col = Year)) +
     geom_point(position = "jitter", alpha = 0.9) +scale_x_discrete(breaks = seq(0, 10, 1)) +
  labs(x = "Decile Rank per Year",
       y = "Decile Rank Overall",
       title = "Comparing Finishing Time Rankings") +
  scale_color_manual(values = color2) +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        rect = element_rect(fill = "#586e75"),
        axis.title.y = element_text(),
        plot.title = element_text(size = 12),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 9),
        panel.background = element_rect(fill = '#657b83', color = "#657b83"),
        legend.background = element_rect(fill = '#657b83', color = "#657b83"),
        legend.key = element_rect(fill = '#657b83', color = "#657b83"),
        legend.position = "right",
        legend.direction = "vertical")
```

### Further Categorization by Age and Gender

To tell the whole story, it will be necessary to understand how different categories may change the overall time, or allow a better look at how certain runners finish. I broke out both age and gender to help understand this a bit better.

```{r gender over the years, echo = FALSE, message = FALSE, warning=FALSE, fig.cap = "Histogram of gender split"}
ggplot(wser.wide, aes(x = YearInt, fill = Sex)) + 
  geom_histogram(binwidth = 1) +
  scale_fill_manual(values = c("#d8b365", "#5ab4ac")) +
  labs(x = "Year",
       y = "No. of Participants",
       title = "Entrants per Year - by Gender") +
  scale_x_continuous(breaks = seq(1985, 2020, 2), limits = c(1985, 2017)) +
  theme_fivethirtyeight()
```

As is evident from figure 10, the split between Men and Women is quite one-sided (in aggregate, 18% of entrants are female). Because of this, I did not include gender into building my final model due to a heavy bias towards males. Figure 11 shows the times for the two compared as well.

```{r finish by gender, echo = FALSE, warning = FALSE, message = FALSE, fig.cap = "Splitting finish times by gender"}
ggplot(subset(wser.wide, DNF == 0), 
       aes(x = Place, y = timeElapsed_MIN, col = Year)) +
  geom_point() +
  facet_grid(.~Sex) +
  scale_color_manual(values = color2, guide = FALSE) +
  labs(x = "Finishing Place",
       y = "Elapsed Time from the Start [min]",
       title = "Finishing Time - by Gender") +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(),
        plot.title = element_text(size = 12),
        axis.title.y = element_text(size = 9, face = "bold"),
        axis.text.x = element_text(size = 9),
        panel.background = element_rect(fill = '#c1c1c1', color = "#c1c1c1"))
```

Another variable to consider with each runner is age. I broke out each age into age groups usually used in races in figure 12 (box plot seemed the most apt here). What is interesting is the number of runners in each category (figure 13). The cause for a small number in the younger (and faster) age groups, I suspect, has to do with the average age of ultra runners, and lottery process to get into the race.

```{r age grouping plot, echo = FALSE, warning = FALSE, message = FALSE, fig.cap = "Box plot of finish times"}
## Setup custom color brackets for age groups
col2Pal <- c('#ffffcc','#c7e9b4','#7fcdbb','#41b6c4','#1d91c0','#225ea8','#0c2c84')
ageLabels <- sapply(seq(1,7,1), function(x) {
  paste(ageBin[x], ageBin[x+1], sep = "-")
})

ggplot(subset(wser.wide, !is.na(ageGroup) & DNF == 0), 
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
        plot.title = element_text(size = 12),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 9),
        panel.background = element_rect(fill = '#c1c1c1', color = "#c1c1c1"))

```


```{r age group histogram, echo = FALSE, message = FALSE, warning = FALSE, fig.cap = "Age group histogram"}
ggplot(subset(wser.wide, !is.na(ageGroup)), 
       aes(x = factor(ageGroup), fill = factor(ageGroup))) +
  geom_bar(stat = "count", col = "black") +
  scale_fill_manual(values = col2Pal,
                    guide = FALSE) +
  scale_x_discrete(breaks = seq(1,7,1), labels = ageLabels) +
  labs(x = "Age Range",
       y = "No. of Entrants",
       title = "Count of Entrants per Age Group") +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        axis.title.y = element_text(),
        plot.title = element_text(size = 12),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 9),
        panel.background = element_rect(fill = '#c1c1c1', color = "#c1c1c1"))

```

### Overall vs. DNF

All the plots above are essentially an exploration of those that finished the race. While there is still valuable information in the plots and associated data above, what really interested me was what distinguished a person from DNF'ing (Did Not Finish) and finishing.

First, figure 14 shows the histogram grouped by those that finished and DNF'd.

```{r DNF Occurence, echo = FALSE, warning = FALSE, message = FALSE, fig.cap = "DNF Histogram"}
ggplot(wser.wide, aes(YearInt, fill = factor(DNF, levels = c("1","0"), labels = c("DNF", "Finished")))) +
  geom_histogram(binwidth = 1) +
  scale_fill_manual(values = c("#ef8a62", "#999999"),
                    name = "DNF Status") +
  labs(x = "Year",
       y = "No. of Participants",
       title = "Entrants per Year - Finish or DNF") +
  scale_x_continuous(breaks = seq(1985, 2020, 2), limits = c(1985, 2017)) +
  theme_fivethirtyeight() +
  theme(plot.title = element_text(size = 12))
```

Overall - the DNF rate is 32%. To continue exploring this further, it was necessary to look at how fast runners went into the aid stations and how their rank may have changed over time.

### Aid Stations and the Story Within

I was able to break out each runner's times into aid stations and extract some valuable information from that. I could evaluate a runner's decile rank at each aid station (`qRank.as`), overall pace at each aid station (`total distance to aid station`/`time elapsed` = `pace.as`), and the differential pace to reach one aid station from the previous (`diff.pace`). From there, I was able to produce summary statistics showing the average pace into each aid station (`pace.avg`), the maximum number of aid stations a runner entered (`as.max`), and the standard deviation of a runner's pace through all the aid stations (`pace.sd`).

By plotting each runner's (x-axis) pace into each aid station, figure 15 gives a good idea of the speed with which each place (finisher or not) ran with. Keep in mind, for every tick on the x-axis, there are multiple paces for each aid station that person hit, along with multiple runners at each place for the different year.

By then breaking out each runner into their respective decile ranking, it becomes easier to see how the pace over time changes with each group in figure 16.

```{r Pace vs Place, echo = FALSE, message = FALSE, warning = FALSE, fig.cap = "Mean pace per runner for every aid stations"}

ggplot(wser.long, aes(x = Place, y = pace.as)) + 
  geom_point(alpha = 0.3, col = "dark grey") +
  geom_point(stat = "summary", fun.y = mean, size = 2, col = "black") +
  scale_y_continuous(breaks = seq(0,35,2.5)) +
  scale_x_continuous(breaks = seq(0, 450, 25)) +
  labs(x = "Overall Place",
       y = "Overall Pace [min/mile]",
       title = "Pace to AS per Place \nMean Pace in Black") +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        #rect = element_rect(fill = "#586e75"),
        axis.title.y = element_text(),
        plot.title = element_text(size = 12),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 9),
        #panel.background = element_rect(fill = '#657b83', color = "#657b83"),
        #legend.background = element_rect(fill = '#657b83', color = "#657b83"),
        #legend.key = element_rect(fill = '#657b83', color = "#657b83"),
        legend.position = "right",
        legend.direction = "vertical")

```


```{r aid station pace by decile, echo = FALSE, warning = FALSE, message = FALSE, fig.cap = "Aggregate of pace into aid stations by decile rank"}
ggplot(wser.long, aes(x = AS_distance, y = pace.as, group = qRank.as)) +
  geom_point(stat = "summary", fun.y = mean) +
  facet_grid(.~qRank.as) +
  labs(x = "Distance from Start",
       y = "Pace [min/mile]",
       title = "Pace into each Aid Station") +
  geom_smooth(method = "lm") +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        #rect = element_rect(fill = "#586e75"),
        axis.title.y = element_text(),
        plot.title = element_text(size = 12),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 7, angle = 30),
        panel.background = element_rect(fill = '#c1c1c1', color = "#c1c1c1"),
        #legend.background = element_rect(fill = '#657b83', color = "#657b83"),
        #legend.key = element_rect(fill = '#657b83', color = "#657b83"),
        legend.position = "right",
        legend.direction = "vertical")
```

Adding a color group to the data for a runner's rank into each aid station helps to show in figure 17 how top runners stayed in the top rankings throughout the race (same color along a vertical section), while those in the DNF section seem to have more color change throughout their grouping. Keep in mind, again, that the x-axis represents the finishing place, while the color represents a runner's decile rank into multiple aid stations for a single year' race.

```{r pace v place by as rank, echo = FALSE, warning = FALSE, message = FALSE, fig.cap = "Individual paces for all years grouped and colored by rank into aid stations"}
ggplot(wser.long, aes(x = Place, y = pace.as)) + 
  geom_point(aes(col = factor(qRank.as)), alpha = 0.6) +
  scale_color_manual("Rank", values = color3) +
  geom_smooth(colour = "black") +
  labs(x = "Place",
       y = "Pace [min/mile]",
       title = "Pace for Finishing Place by Decile Rank") +
  scale_y_continuous(breaks = seq(0,35,2.5)) +
  scale_x_continuous(breaks = seq(0, 450, 25)) +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        rect = element_rect(fill = "#586e75"),
        axis.title.y = element_text(),
        plot.title = element_text(size = 12),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 8, angle = 30),
        panel.background = element_rect(fill = '#657b83', color = "#657b83"),
        legend.background = element_rect(fill = '#657b83', color = "#657b83"),
        legend.key = element_rect(fill = '#657b83', color = "#657b83"),
        legend.position = "right",
        legend.direction = "vertical") +
  facet_grid(.~factor(DNF, levels = c("0","1"), labels = c("Finished", "DNF")))
```

When I extracted out the average pace between aid stations, I was then able to compare how the pace of those that finished differed from those that did not (figure 18).

```{r average pace based on finish status, echo = FALSE, message = FALSE, warning = FALSE, fig.cap = "Average pace per runner into aid stations"}
ggplot(wser.wide, aes(x = Place, y = pace.avg)) + 
  geom_point(aes(col = factor(DNF, 
                              levels = c("0","1"), 
                              labels = c("Finished", "DNF")))) +
  scale_color_manual(values = c("#ef8a62", "#999999"),
                  name = "DNF Status") +
  scale_y_continuous("Average Pace into Aid Stations [min/mile]",
                     breaks = seq(0,35,5)) +
  scale_x_continuous("Overall Place",
                     breaks = seq(0, 450, 25)) +
  ggtitle("Average Pace into Aid Stations by Finish Status") +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        plot.title = element_text(size = 12),
        #rect = element_rect(fill = "#586e75"),
        axis.title.y = element_text(),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 9),
        #panel.background = element_rect(fill = '#657b83', color = "#657b83"),
        #legend.background = element_rect(fill = '#657b83', color = "#657b83"),
        #legend.key = element_rect(fill = '#657b83', color = "#657b83"),
        legend.position = "right",
        legend.direction = "vertical")
  
```

The reason there are so many different pace ranges for those that DNF'd is the number of aid stations that they may have gone through. Sometimes a runner will only go through the 1st or 2nd aid station before callling it quits, and may have gone too fast. Whereas, those that finished seem to have a fairly defined curve by making it through each aid station.

The standard deviation became of more interest to me as that feature may help reveal how consistent they were and whether or not a runner was about to DNF or able to make it through - shown in figure 19.

```{r standard deviation based on finish status, echo = FALSE, message = FALSE, warning = FALSE, fig.cap = "Standard Deviation for Single Runner's Pace to Aid Stations"}
ggplot(wser.wide, aes(x = Place, y = pace.sd)) + 
  geom_point(aes(col = factor(DNF, 
                              levels = c("0","1"), 
                              labels = c("Finished", "DNF")))) +
  scale_color_manual(values = c("#0392cf", "#999999"),
                  name = "DNF Status") +
  scale_y_continuous("Standard Deviation into Aid Stations [min/mile]",
                     breaks = seq(0,40,5), limits = c(0,40)) +
  scale_x_continuous("Overall Place",
                     breaks = seq(0, 450, 25)) +
  ggtitle("Standard Deviation of Pace into Aid Stations by Finish Status") +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        plot.title = element_text(size = 11),
        #rect = element_rect(fill = "#586e75"),
        axis.title.y = element_text(),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 9),
        #panel.background = element_rect(fill = '#657b83', color = "#657b83"),
        #legend.background = element_rect(fill = '#657b83', color = "#657b83"),
        #legend.key = element_rect(fill = '#657b83', color = "#657b83"),
        legend.position = "right",
        legend.direction = "vertical")
```

The standard deviation of a runners pace maintains a similar curve to it, while those that DNF'd have no pattern to them.

The last few graphs on aid station timing and pacing became the focus for my final project. I wanted to be able to find the pattern and factors that are most significant, and to know what was the major cause for a person finishing vs not finishing the race.

## WSER Finishing Model

By establishing a baseline for what the data can do, there are a few different things that can be predicted.

1.  Predict whether or not a racer will finish.
2.  Predict the different times a racer will show up to an aid station along the course based on their previous times.
3.  Predict the final finish times and overall place.

The main goal with my project is to predict whether or not a runer will finish. We'll make the model based on the pre-race information (age, sex, bib) along with information from along the course to help determine if the runner will make it or not. While the other two aspects for prediction in the race would be extremely helpful, they will need to be addressed at a later time.

Taking in the different variables and engineered features added and shown above, it will be important to select the most impactful features and ones that minimize multicolinearity.

Below is a graph of variable importance using the function `boruta` from the package `Boruta` (more information can be [found here](https://m2.icm.edu.pl/boruta/)). I won't get into the details, but it goes through a Random Forest model to parse out the importance of the different factors and affect they have on the output variable.

```{r variable importance, message=FALSE, echo = FALSE, fig.cap = "Variable Importance", cache = TRUE}
# Setup final Data Frame
wser.cor <- wser.wide %>% 
  select(YearInt, Place, Age, ageGroup,
         DNF, pace.avg, pace.sd, as.max, bibNo, sex.male) %>%
  filter(!is.na(Age) & !is.na(pace.sd))

# Run Model to determine importance
var.imp <- Boruta(DNF ~ ., data = na.omit(wser.cor), doTrace = 2)

# Variables of Importance
boruta_signif <- names(var.imp$finalDecision[var.imp$finalDecision %in% c("Confirmed", "Tentative")])

print(boruta_signif)

# Create Plot
plot(var.imp, cex.axis = 0.7, las = 2, 
     xlab = "Features", main = "Variable Importance")
```

Now, a correlation plot to show how those features are related, so as to better select the proper variables.

```{r corrplot of final df, echo = FALSE, fig.cap = "Correlation Plot"}
corrplot(cor(wser.cor), method = "shade",
         type = "lower", order = "hclust",
         title = "Correlation Plot - Aid Station Timing", mar = c(1,1,1,1))

## Split Data
split <- sample.split(wser.cor$DNF, SplitRatio = 3/4)
dnfTrain <- subset(wser.cor, split == T)
dnfTest <- subset(wser.cor, split == F)
```


### Selecting Features 

From the `corrplot` and `variable importance` plot above, the two most impactful features are `Place` and `as.max`. It can be easy to just pick those to use in the model, but we'll need to understand what they mean and if we should actually use them.

Because `as.max` represents the total number of aid stations a runner hits, it makes sense that if a runner made it to more aid stations, they are likely to finish. So this variable, while highly correlated with DNF, can show that basically if a runner makes it to, let's say, aid station #8, they have a more likely chance of finishing the race.

While useful and interesting, the number of aid stations varied between the years due to various reasons, figure 17 shows the different aid stations over the years. Because of how the number of aid stations changes, the variable `as.max` would not accurately model whether or not a runner will finish. This also sheds light on why `as.max` and `YearInt` are highly correlated and why `YearInt` (or `Year`) will be excluded as well.

```{r aid stations over years, echo = FALSE, message = FALSE, fig.cap="Aid Stations by Year"}
aidStations <- wser.cor %>% group_by(YearInt) %>% summarise(AS = max(as.max))
ggplot(aidStations, aes(x = YearInt, y = AS)) + geom_bar(stat = "identity", fill = "#c3cb71") +
  scale_x_continuous(labels = seq(1986, 2016, 1), breaks = seq(1986, 2016, 1)) +
  labs(x = "Year", y = "No. Of Aid Stations", title = "Historical Number of Aid Stations") +
  theme_fivethirtyeight() +
  theme(axis.title = element_text(face = "bold", size = 9),
        axis.title.y = element_text(),
        panel.grid.major.y = element_line(linetype = "dashed"),
        axis.text.x = element_text(size = 7, angle = 30),
        panel.background = element_rect(fill = '#a19c9c', color = "#a19c9c"),
        legend.position = "right",
        legend.direction = "vertical")
```

`Place` is the runner's final finishing placement, and therefore is similar to a runner's DNF status since a DNFers `Place` is given after all finisher's `Place` is assigned.

`ageGroup` is just a larger sub-category of `Age` and only one of those variables will be included. Races typically categorize based on age group, but since `Age` seems to have more importance, that will be the variable we use there.

`bibNo` is a runner's bibNo. While there really isn't much of a correlation here, there are certain bib numbers that are reserved for elites or those that get into the race a certain way. Therefore, `bibNo` offers a chance of helping to add in some predictive power to those with the _fast_ bibs.

The last variable to be used here has to do with pace. Both `pace.avg` and `pace.sd` are taken from a runner's time between each of the aid stations they reached. Calculating `pace.avg` just takes the distance between each of the aid stations and gets a pace for each section, then takes the mean for each runner. `pace.sd` is the standard deviation of those results different paces as well.

From here, the goal will be to come up with a model that uses the data above, and best predicts whether or not a runner will finish the Western States Endurance Run.

### Base Prediction To Beat

To start, a baseline prediction for anyone starting the race. This way, any predictive models should be able to improve upon the accuracy of this basline.

Taking all the results and tabulating those that finished (`0`) and did not (`1`).

```{r base prediction}
table(wser.cor$DNF)
```

That means the base model will predict a runner will finish with an accuracy of _67.8%_.

### Linear Regression

The goal will see how much better we can predict a finish. First we'll start with a basic linear model.

```{r dnf linear modeling, warning = FALSE, cache = TRUE}
## Linear Model
dnf.lm <- lm(DNF ~ bibNo + Age + pace.avg + pace.sd, data = dnfTrain)
summary(dnf.lm)
par(mfrow = c(2,2))
plot(dnf.lm)
dnf.lm.train <-stats::predict(dnf.lm, type = 'response')

rocrPred <- ROCR::prediction(dnf.lm.train, dnfTrain$DNF)
rocrPerf <- performance(rocrPred, 'tpr', 'fpr')
par(mfrow = c(1,1))
plot(rocrPerf, colorize = TRUE,
     print.cutoffs.at = seq(0, 1, 0.1),
     text.adj = c(-0.2, 1.7))

dnf.lm.test <- stats::predict(dnf.lm, dnfTest)

table(dnfTest$DNF, dnf.lm.test > 0.41)

#lm.rmse <- sqrt(mean((dnfTest$DNF - dnf.lm.test)^2))
acc <- round(sum((dnf.lm.test > 0.41) == dnfTest$DNF)/length(dnfTest$DNF),3)
#print(paste("RMSE for Linear Model:", round(lm.rmse, 3)))
print(paste("Accuracy for Linear Model:", acc))
```

The linear model is a first shot at determining a decent regression model for the data here. There are limitations to a linear model, and it is not the best at a binary classification. The accuracy on the test data is what will be used to compare all the different models.

### Logistic Regression

Logistic regression may offer a better fit than linear regression as it has the capability of classifying an output into 1 of 2 groups.

```{r DNF GLM Model, warning = FALSE, cache = TRUE}

#####
# GLM Model k-Fold Cross Validation
#####

ctrl <- trainControl(method = "repeatedcv", number = 10,
                                            repeats = 3)

set.seed(352)
dnf.glm <- train(factor(DNF) ~ bibNo + Age + pace.avg + pace.sd,
                    data = dnfTrain,
                    method = "glm",
                    family = "binomial",
                    trControl = ctrl)

## Coefficients
exp(coef(dnf.glm$finalModel))

## Apply Model to prediction
dnf.glm.pred <- predict(dnf.glm, newdata = dnfTest)

## Summary of Results
cm <- confusionMatrix(dnf.glm.pred, dnfTest$DNF)
cm
```

That seems to give us pretty good results! The model predicts with an accuracy of `r round(as.numeric(cm$overall[1]),4)` But can we do better?

### Support Vector Machine

For the Support Vector Machine, we'll use the base parameters to give an idea to how SVM performs. There are a few different kernel methods to choose when creating the SVM, but here we stuck with a fairly basic Radial Basis Function put through 10-fold cross validation to help with overfitting (`svmRadial`).

```{r SVM Model, message = FALSE, warning = FALSE, cache = TRUE}
## Setup model

## NOT USED ##
#set.seed(352)

#dnf.svm <- ksvm(factor(DNF) ~ bibNo + Age + pace.avg + pace.sd,
#                data = dnfTrain)

###############

## Base SVM Model

set.seed(352)
dnf.svm <- train(factor(DNF) ~ bibNo + Age + pace.avg + pace.sd,
                   data = dnfTrain,
                   method = "svmRadial",
                   trControl = trainControl(method = "repeatedcv", number = 10,
                                            repeats = 3))


dnf.svm.pred <- predict(dnf.svm, dnfTest)

dnf.svm

table(dnfTest$DNF, dnf.svm.pred)

acc <- round(sum(dnf.svm.pred == dnfTest$DNF)/length(dnfTest$DNF),3)
names(acc) <- "accuracy"
acc
```

With the SVM model, we're able to see much different results. Potentially due to the fact that the final classification we're seeking (Finish or DNF), has some heavy outliers that a linear model won't necessarily capture, and SVM has the ability to classify a bit better.

While there are additional modifications that can be made towards the model in terms of tuning, for now, we will stick with the base model with only minor modifications to the model. 

### Regression Trees

There are two separate classification tree algorithms we can use. The first is the standard CART method (Classification And Regression Tree). The second is the Random Forest. We'll utilize both approaches.

#### CART

For the CART method, I ended up using two different libraries to compute the model for CART. The reason is that while both are using `rpart`, `dnf.tree` is called directly through `rpart` and saved as an `rpart` object, therefore giving access to the `rattle` library and having a nice plot output.

The other `rpart` model is created using `caret` with 10-fold cross validation. While the method starts the same, the two must go through some sort of selection process or automatic methods that the other is not, as the two models are not equal. Quite honestly, I'm not sure where they differ and that will take further exploration and reading to understand. I'll show the comparison below, but despite them not being equal, I use the `rpart` to plot the decision tree, and `caret::rpart` to compare to other models.

```{r CART Method, message=FALSE, cache = TRUE}
# Classification Tree
set.seed(352)
dnf.tree <- rpart(factor(DNF) ~ bibNo + Age + pace.avg + pace.sd,
                  data = dnfTrain)

set.seed(352)
dnf.tree.t <- train(factor(DNF) ~ bibNo + Age + pace.avg + pace.sd,
                    data = dnfTrain,
                    method = "rpart",
                    trControl = 
                    trainControl(method = 'repeatedcv', number = 10, repeats = 3))

plotcp(dnf.tree)
#summary(dnf.tree.p)
fancyRpartPlot(dnf.tree, main = "Classification Tree Model", palettes = "PiYG")

### Applying Model to test data
dnf.tree.pred <- predict(dnf.tree, dnfTest)

#tree.rmse <- sqrt(mean((dnfTest$DNF - dnf.tree.pred)^2))
#print(paste("RMSE of Classification Tree: ", round(tree.rmse,3)))

## Accuracy of Model
## Split Point for Probability is centered around 0.5
dnf.tree.pred <- as.data.frame(dnf.tree.pred)
dnf.tree.pred$DNF <- factor(ifelse(dnf.tree.pred$`0` > 0.5, 0, 1))
ref <- factor(dnfTest$DNF)

cartTree.cm <- confusionMatrix(data = dnf.tree.pred$DNF, reference = ref)
cartTree.cm

### Comparison to CARET rpart
dnf.predCaret <- predict(dnf.tree.t, dnfTest)
caretTree.cm <- confusionMatrix(data = dnf.predCaret, ref)
```

Comparison of the accuracy of the two models:

`rpart`      |    `caret::rpart`
-------------|--------------------
`r round(as.numeric(cartTree.cm$overall[1]),4)` | `r round(as.numeric(caretTree.cm$overall[1]),4)`


#### Random Forest

```{r random forest, warning=FALSE, cache=TRUE}
######################
## Random Forest
######################

set.seed(352)
dnf.randForest <- train(factor(DNF) ~ bibNo + Age + pace.avg + pace.sd,
                  data = dnfTrain,
                  method = "rf", trControl = 
                    trainControl(method = 'repeatedcv', number = 10, repeats = 3), 
                  prox = TRUE, allowParallel = TRUE)

dnf.randForest

getTrainPerf(dnf.randForest)

## Applying Model to test data
dnf.randForest.pred <- predict(dnf.randForest, dnfTest)

## Calculating Accuracy
confusionMatrix(dnf.randForest.pred, reference = ref)

```

### Neural Network

It may not be completely applicable to use a neural network for this model with the limited variables and binary classification, but nonetheless, it is worthwhile to go through this model and see what results we can find. We'll just use the base setup for a neural net from the package `nnet::nnet`.

The model will is using `caret` to make the predictions and we'll set it up using 2 hidden layers.

```{r neural net, message=FALSE, warning=FALSE, results='hide', cache = TRUE}
#######################
## Neural Net Model
#######################

set.seed(352)
dnf.net <- train(factor(DNF) ~ bibNo + Age + pace.avg + pace.sd,
                 data = dnfTrain,
                 method = "nnet", hidden = 2,
                 trControl = trainControl(method = "repeatedcv", number = 10, repeats =3))
```
```{r neural net output, warning = FALSE}
## Summary and Accuracy of Trained Model
print(dnf.net$results)

# Applying Model to Test Data
dnf.net.pred <- predict(dnf.net, newdata = dnfTest)

## Confusion Matrix of Prediction
confusionMatrix(dnf.net.pred, reference = ref)
```

### Comparing Results

Because I was able to build the models using `caret`, each model is stored similarly across the different methods and therefore I am able to compare them using a few built in functions. Below is a representation of those different models, and their accuracies compared.

```{r comparing results, fig.cap="Comparison of Model Results"}
results <- resamples(list(Forest = dnf.randForest, nnet = dnf.net, SVM = dnf.svm, GLM = dnf.glm, CART = dnf.tree.t))
summary(results)

## Dot Plot of Accuracy
scales <- list(x = list(relation="free"), y = list(relation="free"))
dotplot(results, scales = scales)
```

### Summary of the Model

By looking at the results table and comparison graphs, the final results show us that, based on pure numbers, the random forest model gets us the best results of predicting a finish at the Western States. While the difference in accuracy is not all that different, it is still noticeable, especially from the GLM model or the baseline of 68%.

The only downside is I was not able to replicate the decision tree properly using `caret` and `rpart`, however, the `rpart` model has very similar accuracy to the `randomForest`. The reason I find this a downside is the decision tree is easy to follow with _YES_ or _NO_ questions.

Based on the ease of reading the decision tree, and only a 1% difference in accuracy, I would recommend using the decision tree for atheletes curious about what their training might lead to and how to plan a race.

For instance, if a runner were to have a training program put together where by the end of it, they knew they could maintain a 15 min/mile, while fluctuating 4 min/mile depending on the vert or descent of a particular section, they could accurately predict they would have an 80% of a solid finish.

However, if the training has been slacking and maintaining even a 19 min/mile is tough, it is better to stay as consistent with the slow pace as possible (57%).

## Future Enhancements and Final Thoughts

Going forward, I think there is still a lot of work that can be done with this data and lead to further insights.

1.  While this excercise above goes through a few different machine learning methods, I pretty much used the base tuning parameters, aside from putting each one through a 10-fold cross validation. Further exploration of the data would include analzying and understanding all the relationships between the features even more to get a better understanding of their geometry and dimensionality. This would allow for finer tuning, and even an enhancment of the feature space too.

2.  A few other things can be done with the data scraping aspect of the project. While I was able to extract what I deemed the most useful data, more feature engineering can take place with some more of the fields (e.g., `% of total aid stations`, `time spent at aid station`), along with capturing a few more of the fields I had to exclude due to length of time to data mine (e.g. `state` and `country`).

3. Another way to split and perform regression on the data would be to use clustering and understand the difference between the elites from the normal runners, or if they differ at all aside from their shear speed.

4.  Further data to gather, not present in this dataset, would be the addition of elevation data along the course and training data. Both of these would be extremely helpful, and would take this model to a new step.  
By adding in elevation data, it can help to compare how pace changes with grade, and provide further insight. This would open the door even more by allowing a comparison of not only this single race, but being able to compare this race with others based on relative pace per grade. While this information is available, it would require a significant more amount of time to integrate, but I do hope to do so in the future.

5.  Training data is probably the biggest determining factor for whether or not a runner will finish. Training data would give the ability to create a model and determine what sort of training is most apt for this race, and could even allow for a better understanding of ultra marathons as a whole. By creating a model that is inclusive of elevation gain and miles per week, coaches and runners could use training tools such as Strava and Training Peaks to really hone in their approach for a specific race.

6.  Predict finishing time and place and anticipated arrival at aid stations.

## Application in the Real World

What this model shows is the ability to focus in on specific variables and help a runner determine whether or not they have the tools to finish. But this information is sometimes only applicable during race day.

By combining this model with training information, the model would have much greater power as it now has a better piece of information that could be applied prior to the run, instead of during it. Online platforms such as [Training Peaks](http://trainingpeaks.com/) and [Strava](https://www.strava.com) offer training plans along with helping athletes track workouts and give metrics.

A few ways this project could integrate with their system:

1. Through their training plans they offer, a user could enter in their goal of finishing the Western States Endurance Run. This model would then be able to take their Age information, and then bsaed on speed and different splits and standard deviations over time, they would accumulate more and more data points to give indicators to how they would do in the race.

2. The trainng platforms could retroactively ask users that have participated in WSER to share their training data so as to incorporate that into building a better model.

3. Coaches (or those self-coached) could take and analyze their own data through whatever program works best for them, and compare their average paces over distance, along with standard deviation of different splits from different races, and get a gauge for how they might finish at WSER.

I hope to be able to continue the development of this project along with the accuracy and usefulness of this model.

## Acknowledgements

My sincerest thanks goes out to Matt Fornito for helping me to refine this project and turn it into something meaninful and fun, and hopefully insightful to some. I know I have enjoyed it, and couldn't have gotten this far without his mentorship.

Secondly, I want to thank YOU reading this now. If you made this far (or just skipped to the bottom), you are reading this, and that is 1 more person than I thought would read this. Thank you.

Please don't hesitate to add commentary or thoughts on future improvements.