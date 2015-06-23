---
layout: post
title: Drug Labeling Anomalies
excerpt: "Finding reactions not found on drug labels"
modified: 2015-05-23
tags: [topic modeling, svm, mongodb, nlp, natural language processing, python, statistics, metis]
comments: true
image:
  feature: banner1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---
### Post in progress

I've been quite busy since my last post about the FDA open data api!  I downloaded over 50 gb of FDA data and explored all three 'Drug' options (Labeling, Adverse Events, Recalls).  There are over 4 million adverse events, 70k drug labels, and 7k recalls to analyze so it's kept me quite busy!  In this post, I'm going to speak about how I analyzed the FDA approved drug labels to look for labels that need to be strengthened / improved based on the reactions being reported for the drug.  Getting this data into a DB and cleaning it was a process in itself and I'll speak more about that towards the bottom of the post after the results section.

### Why FDA drug labels?
When I was in university, I woke up one day with serious neck pain and had to see the Doctor immediately because the pain was unbearable.  The Doctor prescribed me some pain killers, told me to take one right now, and then as needed.  About 15 minutes later, I started to sweat, the room was spinning, and next thing I know my roommate is repeatedly asking me "Are you OK??".  I had passed out from this drug...an unpleasant adverse reaction to this painkiller.  When I reviewed the warning label later that day, there was no mention of this as a side effect.

When looking over the drug label data, this story came to my mind and I thought "I wonder how many drug labels out there don't truly show the potential side effects?"  This post explains how I went about finding these drug labels.

### How to identify anamolies?
I wanted to analyze the actual warning label text of an FDA drug label, so how was I going to identify anamolies based on the text alone?

<figure>
  <a href="/images/fda_drug_label.png"><img style="display:block; margin: 0 auto;" src="/images/fda_drug_label.png"></a>
</figure>

I tried two techniques to analyze the warning label text.  The first was to transform all the warning labels into a matrix of term frequencies.  I tried both TF and TF-IDF representations of the data, but neither worked particularly well for the analysis I was conducting.  The TF-IDF works well when trying to find unique words in a block of text and I used it on my last project.  However, for warning label text, many of the same words were going to be repeated and I did not want to alienate those words.

Next, I had recently come across topic modeling using [Latent Dirichlet Allocation][1] and thought it would be more appropriate.  This technique works to extract all the various topics present in your text.  Applied to my data, each warning label would have a score for each topic ranging from 0 to 1.  A zero meaning that topic isn't present at all in the label, and a 1 meaning that topic is fully contained in the label.

To do this, there is an open source package called [Gensim][2].

<figure>
  <a href="/images/gensim.png"><img style="display:block; margin: 0 auto;" src="/images/gensim.png"></a>
</figure>



### Overview


### Option I'm Choosing

<figure>
  <a href="/images/competition2.png"><img style="display:block; margin: 0 auto;" src="/images/competition2.png"></a>
</figure>


As you can see above, I'll be taking on option number 2 which is related to looking for patterns in product labels and seeing if I can identify any inefficiencies.  To me, this screams both clustering and text analysis.  I'm excited to see if I can build a product that helps the FDA better understand their data.

### Conclusion

Well... I'm very VERY excited to be working on this project!  It is one of the richest data sets I have seen and the potential impact of great analysis can be massive!  I encourage all of you reading this blog to participate as well and see what insights you can come up with.  To follow my progress on this project, please visit my github [FDA OpenData Project][3].

### Misc.

On a more technical note, I've been working in MongoDB for this project because the data comes from the API in JSON format and MongoDB is perfect for this sort of data.  I had some issues with the API at first with rate limits and pagination, but now seem to have it all sorted out.  If you want to get some boiler plate code to get your data into MongoDB, feel free to check out the code snippet below.  I'm using PyMongo which is a nice python wrapper for MongoDB.

{% highlight python %}
"""Note: you need to have MongoDB running for any of this to work.
  Also, you need to have your own api key so that you don't exceed
  the rate limit parameters I'm looping with."""

### IMPORTS ###
import requests
import json
import cnfg
import pprint
from pymongo import MongoClient
import time

### EXTRACTING DRUG LABELS DATA ###
# there's around 70k drug labels
client = MongoClient()
labels = client.drugs.drug_labeling

print "starting the drug labels extraction..."
for i in range(0,1000):
    j = int(i*100)
    if i % 240 == 0:
        time.sleep(59)
        print "on iteration: " + str(i)
    response = requests.get("https://api.fda.gov/drug/label.json?api_key=GETYOUROWN&search=&limit=100&skip=" + str(i) + "\"")
    labels.insert(response.json())


### EXTRACTING ADVERSE EVENTS DATA ###
# there's over 4mm events from 2004

client = MongoClient()
events = client.drugs.adverse_events

print "starting the adverse events extraction..."
for i in range(0,45870):
    j = int(i*100)
    if i % 240 == 0:
        time.sleep(59)
        print "on iteration: " + str(i)
    response = requests.get("https://api.fda.gov/drug/event.json?api_key=GETYOUROWN=&limit=100&skip=" + str(i) + "\"")
    events.insert(response.json())

## EXTRACTING RECALLS DATA ###
# there's around 4,000 events

client = MongoClient()
enforcement = client.drugs.enforcement

print "starting the enforecement extraction..."
for i in range(0,38):
    j = int(i*100)
    response = requests.get("https://api.fda.gov/drug/enforcement.json?api_key=GETYOUROWN&search=&limit=100&skip=" + str(i) + "\"")
    enforcement.insert(response.json())
    print "finished iteration: " + str(i)
print "all done!"
{% endhighlight %}


~ Trevor

[1]: https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation
[2]: https://radimrehurek.com/gensim/
[3]: https://github.com/trevor-smith/FDA_OpenData
