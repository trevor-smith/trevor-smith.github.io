---
layout: post
title: Using moving averages to improve fanduel predictions
excerpt: "Improving performance to 94%"
modified: 2015-07-16
tags: [random forest, svm, tf-idf, nlp, natural language processing, python, statistics, metis]
comments: true
image:
  feature: banner1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---
### Intro
[Project link][1]


Ok, last night was my first post around fanduel and it had me very excited!  I was able to create a model that explained 76% of the variation in the fanduel scores I was seeing on a nightly basis!  But there were so many other things I wanted to try that would improve predictive accuracy; one of them beingt moving averages!  This post details how I used moving averages to improve predictions up to 94%

<figure>
  <a href="/images/fanduel_post2_1.gif"><img style="display:block; margin: 0 auto;" src="/images/fanduel_post2_1.gif"></a>
</figure>



### What's a moving average?
A moving average is just an average that keeps updating as new data is available.  For example, a 3 day moving average takes your data from the last 3 days and computes and average.  The next day, the previous 3 days of data are used to compute and updated moving average.  Moving averages can help to identify trends.

Why do we care? Because basketball, just like any sport, is about hot and cold streaks.  Sure, Russell Westbrook is phenomenal all year long, but there's definitely periods where he is doing better and worse.  If Kevin Durant is playing, Westbrooks moving averages are going to likely be lower than when Kevin Durant was injured, and the moving average calculations are going to capture that!

In the context of this analysis, I am going to start by creating a moving average based on just the player's fanduel scores.  So in Westbrook's case, I will be saying "what were his average fanduel points for the last 3 games, 5 games, 10 games?"

<figure>
  <a href="/images/fanduel_post2_2.jpg"><img style="display:block; margin: 0 auto;" src="/images/fanduel_post2_2.jpg"></a>
</figure>


### Calculating our moving average
In order to calculate a moving average, pandas makes it very easy.  There is a 'rolling_mean' function that you can call which calculates a moving average for you.  You just need to specify the column and the number of periods.

In order to do this, you must first sort your data by player name, and then by date, so that performances are ordered properly.  The code to sort and calculate a moving average is below:


{% highlight python %}
# data must be grouped first
# sorting the data
df_sorted = df.sort(['PLAYER', 'DATE'], ascending=[True, True])

# creating the moving averages
df_sorted['past_1'] = pd.rolling_mean(df_sorted['fanduel_score'], 1)
df_sorted['past_3'] = pd.rolling_mean(df_sorted['fanduel_score'], 3)
df_sorted['past_5'] = pd.rolling_mean(df_sorted['fanduel_score'], 5)
df_sorted['past_10'] = pd.rolling_mean(df_sorted['fanduel_score'], 10)

{% endhighlight %}

So this is great, but you'll notice now that there are NaNs for some of the moving average values.  For example, on our first very data point for a player, we see that none of the moving averages compute because there isn't any previous data to compute on.  So if we are in the first 5 games of a season for a player, we aren't going to be able to compuate a 10 game moving average.  In these cases, I think imputing a season mean or the moving average from the next most recent moving average would be appropriate.

### Predicting
Ok, now assuming we have already split our data into training and test or are going to do cross validation, it's time to predict!  I'm using all the same features as yesterday, but will now add the moving averages in today.  I wasn't able to get the log transform to work on this data set, so I will be using the regular fanduel score as the dependent variable.  Another note is that I did not have time to impute the season averages or any other missing values from the moving average computation, so I have just dropped rows that contain NaN.  Not ideal, but I will fix this in the long run.  Still, there are many players and games that can still be predicted and the results are great.

As yesterday, I'm just using the SVM regressor with the RBF kernel.  Not tuning the C or Gamma parameters yet.

**Results:** 94% r-squared! Very great score for again somewhat minimal work.  I'll be pretty pleased if on average I can explain 94% of variations in fanduel scores with this model.  By the time we are two or so weeks into the season, my model should be outputting great scores!

{% highlight python %}
from sklearn.svm import SVR

model_SVR_rbf = SVR(kernel='rbf', C=.5)
model_SVR_rbf.fit(x_train, y_train)
model_SVR_rbf.score(x_test, y_test)

{% endhighlight %}


### Next steps
One input that seriously helps our prediction is the number of minutes that a player plays on a given night.  Very predictive.  But this isn't known beforehand and minutes can often be a factor of how well a player is playing that night.  Some work will need to be done to look at the distribution of minutes for players who start.  If this is fairly small, then I would like to introduce a 'Starter' variable which indicates whether a player is starting or not.

Another way to predict minutes is to look for inuries.  If a starting PG goes down with an injury, the backup PG is almost guaranteed to see his minutes jump from around 15-20 to 35-40, thus increasing fanduel production.  This is a very important strategy when crafting your lineups, and I'll be looking to automate this step in the future.

~ Trevor

[1]: https://github.com/trevor-smith/fanduel_nba
[2]: https://radimrehurek.com/gensim/
[3]: http://scikit-learn.org/stable/modules/generated/sklearn.manifold.TSNE.html
[4]: http://scikit-learn.org/stable/modules/generated/sklearn.metrics.silhouette_score.html#sklearn.metrics.silhouette_score
[5]: https://twitter.com/bo_p
[6]: https://twitter.com/planarrowspace
