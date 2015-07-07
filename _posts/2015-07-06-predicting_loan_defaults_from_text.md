---
layout: post
title: Predicting Loan Defaults from Text Data
excerpt: "Cleaning and using Vectorizers for Classification"
modified: 2015-07-06
tags: [random forest, svm, tf-idf, nlp, natural language processing, python, statistics, metis]
comments: true
image:
  feature: banner1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---
### Intro

I've been interviewing with a number of financial companies and one recently had me use the Lending Club data set to predict loan default.  There are a number of numerical features that I plan to explore, and I could spend A LOT of time cleaning and engineering features (there are almost 100 to start with).

However, one way to predict loan default was to use the 'desc' field.  This field contains the reason why the person needs the loan.  An example of a few are shown below:

<figure>
  <a href="/images/sample_desc.png"><img style="display:block; margin: 0 auto;" src="/images/sample_desc.png"></a>
</figure>



### Data Cleaning
As you can see, there are a number of issues with this text data.  Firstly, they all start off with 'Borrower added on 10/04/13 >', maybe some other dates (which won't be useful in this initial classification), and a 'br' tag.

When classifying using text data, we also want to remove punctuation and lower all the text so that 'Loan', and 'loan' are treated the same.  My code for cleaning this is below.  I use regular expressions.

Lastly, there are some missing descriptions.  For these I just fill with 'no description'.


{% highlight python %}
### IMPORTS ###
import pandas as pd
import numpy as np
import re

# read in data
df = pd.read_csv('LCdataset.csv', header=1)

# view a few examples
for i in df.desc[:10]:
  print i

# view blanks and fill in
print df.desc.isnull().sum()
df.desc = df.desc.fillna('no description')

# now to actually clean the data
# we will put the results to a new column called 'desc_clean'

# lower case everything
df['desc_clean'] = df.desc.apply(lambda x: x.lower())

# remove beginning string 'borrower added on...'
df['desc_clean'] = df.desc_clean.apply(lambda x: re.sub('borrower added on ', '', x))

# remove the dates
df['desc_clean'] = df.desc_clean.apply(lambda x: re.sub(r'(\d+/\d+/\d+)', '', x))

# remove br tag
df['desc_clean'] = df.desc_clean.apply(lambda x: re.sub('<br>', '', x))

# remove whitespace and punctuation
df['desc_clean'] = df.desc_clean.apply(lambda x: re.sub(r'[^\w\s]','', x))

{% endhighlight %}



### Representing our text data in matrix form
Now that our text data was clean, it was now time to start classifying!  There are a few common ways to do this...the two I generally use are python's count vectorizer and python's tf-idf vectorizer.  The first one uses counts of the words and the second one uses counts of the words divided by the number of times that word appears amongst all the samples.  The second method emphasizes phrases that don't appear a lot amongst all the samples, but appear in one specific sample.

The method I generally default to is the TF-IDF representation of the data because it finds words or phrases that are rare amongst the document corpus.  However, intuition can be towards just using the raw counts representation of the words for this problem.  Perhaps there are some phrases that do appear frequently, but are actually VERY useful in classification.  It is hard to know before hand, so I will do some testing with both.

The code below splits the data into training and test sets, then creates both count vectorizer and tf-idf vectorizer representations of the data.  A random forest will be run on both to see which produces higher accuracy, precision, and recall.

{% highlight python %}
### IMPORTS ###
from sklearn.cross_validation import train_test_split
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.feature_extraction.text import CountVectorizer

# split our data into training and test sets
df_train, df_test = train_test_split(df, test_size=.15, random_state=0)

# count vect
countvect = CountVectorizer(min_df=1, stop_words='english',ngram_range=(1,4))

# tf-idf
tfidfvect = TfidfVectorizer(min_df=1, stop_words='english',ngram_range=(1,4))

# transforming count data
X_train_count = countvect.fit_transform(df_train.desc_clean)
X_test_count = countvect.transform(df_test.desc_clean)

# transforming tfidf data
X_train_tfidf = tfidfvect.fit_transform(df_train.desc_clean)
X_test_tfidf = tfidfvect.transform(df_test.desc_clean)

{% endhighlight %}

### What is a bad loan?
Up until now this post has talked exclusively about the explanatory variables.  But what is it that we're actually trying to predict??

This data set contains a column labeled 'loan_status'.  It has 7 different categories which describe the current status of the loan, but the goal of this post is to just tell whether or not a loan will default or not.  In this case that equals a loan status of 'Default' or 'Charged Off'.  We can take a look at the distribution of these loan statuses.

<figure>
  <a href="/images/loan_statuses.png"><img style="display:block; margin: 0 auto;" src="/images/loan_statuses.png"></a>
</figure>

We can see from the above figure that most loans are current, then some are fully paid, and finally some are charged off.  Overall, we see that bad loans make up around 5% of the data set.

### Evaluating the better word representation
Now that I have defined what a bad loan is, it was time to test which matrix representation was more effective in predicting loan default.

I would just use a logistic regression classifier because it is quick to train.

{% highlight python %}
### IMPORTS ###
from sklearn.naive_bayes import GaussianNB
from sklearn.svm import SVC
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import precision_recall_fscore_support

# count representation
model = LogisticRegression(class_weight='auto', C=0.5)
model.fit(X_train_count, y_train)
pred_values = (model.predict(X_test_count))
true_values = y_test

recall = str(precision_recall_fscore_support(true_values == 1, pred_values == 1, average='binary')[1])
precision = str(precision_recall_fscore_support(true_values == 1, pred_values == 1, average='binary')[0])
accuracy = str(model.score(X_test_count, y_test))
print "count vectorizer: "
print "recall: " + str(recall) + " precision: " + str(precision) + " accuracy: " + str(accuracy)

# tfidf representation
model = LogisticRegression(class_weight='auto', C=0.5)
model.fit(X_train_tfidf, y_train)
pred_values = (model.predict(X_test_tfidf))
true_values = y_test

recall = str(precision_recall_fscore_support(true_values == 1, pred_values == 1, average='binary')[1])
precision = str(precision_recall_fscore_support(true_values == 1, pred_values == 1, average='binary')[0])
accuracy = str(model.score(X_test_tfidf, y_test))
print "tfidf vectorizer: "
print "recall: " + str(recall) + " precision: " + str(precision) + " accuracy: " + str(accuracy)

{% endhighlight %}

Count Accuracy: 91%
TFIDF Accuracy: 81%

We can see that the count representation of the data actually has better accuracy.  Moving forward we will use this representation.

### Modeling the data
91% accuracy is quite good for a first run using logistic regression.  I will be updating this section as results start coming in from my modeling.

~ Trevor

[1]: https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation
[2]: https://radimrehurek.com/gensim/
[3]: http://scikit-learn.org/stable/modules/generated/sklearn.manifold.TSNE.html
[4]: http://scikit-learn.org/stable/modules/generated/sklearn.metrics.silhouette_score.html#sklearn.metrics.silhouette_score
[5]: https://twitter.com/bo_p
[6]: https://twitter.com/planarrowspace
