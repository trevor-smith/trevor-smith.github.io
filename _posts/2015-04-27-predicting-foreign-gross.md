---
layout: post
title: Predicting Foreign Revenue for American Movies
excerpt: "Exploring movie tastes for countries around the world"
modified: 2015-04-27
tags: [intro, beginner, jekyll, tutorial]
comments: true
image:
  feature: sample-image-5.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---
As you know from my last post, I had scraped A LOT of movie data from BoxOfficeMojo...but I was unsure what to do with it.  After reviewing the data I had scraped, the problems that seemed the most interesting to me were related to predicting revenue, but predicting how a movie would do in the United States did not seem as interesting to me.  I wanted to solve a prediction problem that had global implications.  I finally settled on trying to predict how much money an American movie would gross in every single country it was released.  This excited me, although I knew it would be challenging!

To start this problem, I did an initial review of my data to see what I was working with.  Overall, the data was pretty clean because my web scraping was robust, but I did have to link two data sets together.  Luckily, in pandas this is an easy one line implementation that joined the two data sets based on their urls.  One thing I did notice during this process was that my revenue data was not normally distributed.  It had a left skew, but I was able to normalize it by doing a log transform (sample code below):

{% highlight python %}
df.Revenue.hist()
# it has a left skew, so let's transform
df.Revenue = df.Revenue.apply(np.log)
df.Revenue.hist()
# rerun histogram and now data looks normally distributed
{% endhighlight %}

<figure class="half">
    <a href="/images/revenue.png"><img src="/images/revenue.png"></a>
    <a href="/images/revenue_log.png"><img src="/images/revenue_log.png"></a>
    <figcaption>Left: no log transform Right: log transform</figcaption>
</figure>

This transformation greatly helped the predictive power of my linear regrssion.  The features that I was working with initially were standard movie revenue features such as movie budget, genre, MPAA rating, and release date.  These were great at predicting domestic movie revenue, but there were still many missing pieces for country by country prediction so I brainstormed and came up with a few features I thought would be predictive:

- GDP per capita (worldbank.org)
- Population (worldbank.org)
- U.S. Favorability index (pew research center)

The reason I focused on these three features above was because I felt like they were widely available data that would impact the movie revenue a particular country would generate.  GDP per capita is important because countries with low GDP per capita will not be able to spend as much money on non-essential items.  As that GDP per capita rises, discretionary spending on other items such as movie tickets can increase.  Next, population is important because even if a country has a high GDP per capita (Luxembourg), they still do not have the population to generate high movie revenue.  Lastly, I hypothesized that how a country views the U.S. will impact whether or not they will see U.S movies.  To estimate this, I used the U.S. favorabiity index which measure the percent of a country's population that approves of the United States.




~ Trevor
