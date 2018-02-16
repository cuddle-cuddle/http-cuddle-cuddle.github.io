---
layout: post
title: "NLP: Short Sentence comparison techniques OR: Tell me a joke! (Part 1: scraping twitter)"
---
![KidsWriteJokes]({{ site.baseurl }}/images/posts/2018-02-13/kidswritejokes.png)

Okay, so, Start of the problem is simple. I want an AI such that when I say, "Tell me a joke about nails!" I'd like to have some reasonable answers back about either Jesus or manicures.

It's a problem that I would have never visited myself --- I mean, if I were to implement one, I'd just make a google API call and get the result, instead of crawling through some corpus that I have to scrape and crawl my self.

But otherwise, the underlying problem is very interesting:
We're comparing sentences, not documents, not words. Hum.

We're going to visit a few Gensim implementations, and a few home-brews to see how things compare.
Quantification and comparison of methods will be... qualitative. Some of the methods, word mover's distance for example, are unapologetically long and GPU consuming. As a hobo, I'll have to resort to using my verbal skills and pragmatism to settle scores.

But first, crawling. Where the heck do we get jokes?

The first joke source that I got my hands on is interesting: [Kids Write Jokes](https://twitter.com/KidsWriteJokes) Twitter handle.

Twitter has well implemented, well used, well supported API that make things very very simple.

TL;DR: `pip install tweepy`

[A good Tweepy introduction: ](https://marcobonzanini.com/2015/03/02/mining-twitter-data-with-python-part-1/).

 Given all tweets, we load them with pandas.
 {% highlight python linenos %}
 df = pd.read_json("./kidswritejokes/kidswritejokes_tweets.json")
 df.sample(10)
 {% endhighlight %}

A random sample of the json file reveals this:

![raw jokes dataframe](/images/posts/2018-02-13/sample10.png)

You can see features and problems within the data immediately:
<ul>
<li>we only need the text column</li>
<li>The text is a mix of tags, comments, references that are not jokes. </li>
<li>The jokes are in the format of: `Q:...? \n\n A:...`. For example, "Where's batman? \n\n Some silly answer here." </li>
</ul>

So immeidately, one can start stripping away useless information and things that are not jokes:

{% highlight python linenos %}
# get all potential jokes
joke_list = list(df['text'])
# filter all non-jokes
jl = list(filter(lambda x: (x.count('\n')>1 & x.count('@')==0 & x.count('//')==0 ), joke_list))
# split potential jokes into Q&A format
joke_list_list = list(map(lambda jk: list(filter(lambda x: len(x) >0, jk.split('\n')))[-2:], jl))
# Strip all jokes with hashtags, links, etc.
joke_list_list = list(filter(lambda jk:
            jk[0].find('http') ==-1 &
            jk[0].find('.com') ==-1 &
            jk[0].find('@') ==-1 &
            jk[0].find('\\') ==-1 &
            jk[1].find('http') ==-1 &
            jk[1].find('.com') ==-1 &
            jk[1].find('@') ==-1 &
            jk[1].find('\\') ==-1 , joke_list_list))

{% endhighlight %}

The result is as desired:

![clean jokes](/images/posts/2018-02-13/jokelistlist.png)

Which reduces about 3k+ lines of jokes, comments etc. to ~250 Q&A style jokes.

A bigger list (about 200k+) of short jokes are actually available at Kaggle:
[Kaggle Short Jokes](https://www.kaggle.com/abhinavmoudgil95/short-jokes)
![kaggle short jokes](/images/posts/2018-02-13/kaggleshortjokes.png)
