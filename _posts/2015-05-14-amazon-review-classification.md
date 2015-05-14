---
layout: post
title: Classifying Amazon reviews Part II
excerpt: "Classifying Amazon reviews as helpful or not"
modified: 2015-05-08
tags: [nlp, natural language processing, python, statistics, metis]
comments: true
image:
  feature: sample-image-5.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---
Pivot.

We had a talk recently about lean startups and one of the themes was the ability to pivot.  This week I had to do exactly that!

In my last post I detailed how to read in Amazon reviews data in Python and how to engineer some basic features.  I had planned to use features about the review length and sentence structure in order to classify whether other users would find your Amazon review helpful or not.  This was working OK, but I knew that there was a wealth of data in the actual review that I was not capturing in my analysis.  Not having any natural language processing training, I didn't really know where to start, so I did what any sane person would do; GOOGLE it.

### Transforming our data
I came across quite a few different tutorials across the web, but ultimately decided to implement what is called a 'Bag-of-Words' approach.  What this approach does is looks through all of my review text and it creates a sparse matrix of all the words.  Then your classification algorithm of choice can treat each of these words as a feature.  Let's look at an example.

### Example

<figure>
  <a href="/images/matrix.png"><img style="display:block; margin: 0 auto;" src="/images/matrix.png"></a>
</figure>

Ok, so we can see that we have two sentences above.  The first one contains six words, all of them unique.  The second sentence contains seven words, but there are three words that we saw in the first sentence so no additional columns need to be created for those three words.  Now that we have reviewed all our of reviews and put together a unique word list, we now have to go back and start filling in our matrix.  The next part of the process just involves going through each review and seeing if it contains the word in the first position of our matrix.  If yes, then we put a 1, if not, then we put a 0.  Then move on to the next word in our matrix and do the same process.  After this has been done for one review, then we move to the next review and do the same thing.  Sounds daunting right??  Thanks to Python, it isn't!


{% highlight python %}
"""As a good data scientist, let's first split our data into a training set and test set.  I've chosen an 80/20 split"""

from sklearn.cross_validation import train_test_split
df_train, df_test = train_test_split(df_reviews, test_size=0.2, random_state=0)

"""Ok, now that we have our data split into training and test, let's create our word matrix!  We do this by importing the CountVectorizer function"""

from sklearn.feature_extraction.text import CountVectorizer
X_train = vectorizer.fit_transform(df_train.reviewText)

"""What this just did is used our training data and created the word matrix.  Now the next step is to apply this matrix to our test data set.  If the word wasn't seen in the training data set, then it won't be in our word matrix we create for our test data set"""

X_test = vectorizer.transform(df_test.reviewText)
{% endhighlight %}

### Testing the models
Alright!  Using the above code we were able to create a word matrix from our training data, and then apply that same word matrix to our test data set.  Now that we have our data in order, it is time to start fitting some models!  For this project I tested all of the typical classification algorithms such as Naive Bayes, Logistic Regression, Support Vector Machimes, and Random Forests.  What we want to do is use our training data to fit our model.  Then once we have a model that has been trained, we test that on our test data!  Remember, our model has not seen our test data at all so this is the moment of truth to see how effective it is at classifying data that it has not seen before.  Feel free to skip the code below and just move on to the next section.

{% highlight python %}
"""standard imports"""
from sklearn.naive_bayes import MultinomialNB

def naive_bayes():
    nb = MultinomialNB()
    nb.fit(X_train, df_train.reviews_helpful)
    nb_pred = nb.predict(X_test)
    nb_score = nb.score(X_test, y_test)
    return "Naive Bayes accuracy: " + str(nb_score)

naive_bayes()
{% endhighlight %}




~ Trevor

[1]: http://cseweb.ucsd.edu/~jmcauley/
[2]: http://jmcauley.ucsd.edu/data/amazon/
[3]: http://textblob.readthedocs.org/en/dev/
