---
layout: post
title: Stepping up our game
excerpt: "A forward stepwise linear regression using statsmodels"
modified: 2015-04-23
tags: [intro, beginner, jekyll, tutorial]
comments: true
image:
  feature: banner1.jpg
<!--
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
 -->
---

One of the great things about R is that everything, I mean EVERYTHING relevant to statistics has been implemented.  Being a former R user, I was aware of it's power, but up until now hadn't found any features missing in Python.  Statsmodels was filling all my linear regression needs and then some.  Dare I say, I even liked the summary output more??

Anyways, for my recent project, I found myself having to optimize ten
different linear models...one for each country!  I found it time consuming enough to even optimize one model, let alone ten...but then I remembered that one time in statistics class when I learned about forward stepwise regression.  Basically, it adds features in to your model one at a time until it finds an optimal score for your feature set, and this process happens almost instantly on a reasonable amount
 of data.  This seemed like a great place to start for each of my ten countries
so I quickly googled "stepwise regression statsmodels" and to my surprise I found NOTHING!  What could one do in this sort of situation?  Code it ourselves, DUH!

I reached out to the very knowledgeable [Aaron Schumacher][1] (seriously, I'm blown away every day by how much he knows regarding python), and he decided to sit down with me and help me find a solution.  He suggested we optimize adjusted R-squared and I agreed and BOOM, we were off implementing the solution.  I will admit he was the  brains behind the clean code below, but I'd like to think that I helped him with a typo or two ;)

Now for the code!  Feel free to try this yourself!  Very flexible and easy to use.


{% highlight python %}
import statsmodels.formula.api as smf

def forward_selected(data, response):
    """Linear model designed by forward selection.

    Parameters:
    -----------
    data : pandas DataFrame with all possible predictors and response

    response: string, name of response column in data

    Returns:
    --------
    model: an "optimal" fitted statsmodels linear model
           with an intercept
           selected by forward selection
           evaluated by adjusted R-squared
    """


    remaining = set(data.columns)
    remaining.remove(response)
    selected = []
    current_score, best_new_score = 0.0, 0.0
    while remaining and current_score == best_new_score:
        scores_with_candidates = []
        for candidate in remaining:
            formula = "{} ~ {} + 1".format(response,
                                           ' + '.join(selected + [candidate]))
            score = smf.ols(formula, data).fit().rsquared_adj
            scores_with_candidates.append((score, candidate))
        scores_with_candidates.sort()
        best_new_score, best_candidate = scores_with_candidates.pop()
        if current_score < best_new_score:
            remaining.remove(best_candidate)
            selected.append(best_candidate)
            current_score = best_new_score
    formula = "{} ~ {} + 1".format(response,
                                   ' + '.join(selected))
    model = smf.ols(formula, data).fit()
    return model
{% endhighlight %}

~ Trevor

[1]: http://http://planspace.org/
