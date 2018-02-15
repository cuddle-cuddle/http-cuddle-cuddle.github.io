
# Scraping /r/jokes from reddit
I do feel there's a need for explanation: 
Why reddit jokes? 
The answer is simple: the ease of scraping and the format of the jokes. 
/r/jokes reddit posts follow similar format: Short question or first line in the title, and punch line in the body of the post. 
Added goodies are: score to seperate good jokes from the bad ones, and time of the post. 
This leaves room plenty for exploration: 
-- what kind of jokes are made the most often when? 
-- are people posting more jokes during which month? 

etc. etc. 

## set up PRAW
praw is the go to API for reddit scraping, so this is not an exception. 


```python
import string
import praw

redditBot = praw.Reddit(user_agent='XXX',
                  client_id='XXX',
                  client_secret='XXX',
                  username='XXX',
                  password='XXX')
```

View the first few submissions to see what we're dealing with: 


```python
jokesSub = redditBot.subreddit('Jokes')
from datetime import datetime
for submission in jokesSub.hot(limit=3):
    print(submission.title, submission.selftext)
    print("time:", datetime.utcfromtimestamp(submission.created_utc).strftime('%Y-%m-%d, %H::%M'))
    print("author: ",submission.author.name)
    print("score: ", submission.score,)
    print("id:", submission.id)
    #print("ups & ratio:", submission.ups,',', submission.upvote_ratio)
    print("----------------")
```

    By popular demand, we now have a discord server. Join this **Guaranteed reposts.** 
    
    https://discord.gg/66qyTgJ  or https://discord.gg/jokes
    time: 2017-11-10, 19::30
    author:  love_the_heat
    score:  2099
    id: 7c3dev
    ----------------
    A 90-year-old man goes for a physical and all of his tests come back normal. The doctor says, “Larry, everything looks great. How are you doing mentally and emotionally? Are you at peace with God?” 
    
    Larry replies, “God and I are tight. He knows I have poor eyesight, so He’s fixed it so when I get up in the middle of the night to go to the bathroom, poof! The light goes on. When I’m done, poof! The light goes off.”
    
    “Wow, that’s incredible,” the doctor says.
    
    A little later in the day, the doctor calls Larry’s wife.
    
    “Bonnie,” he says, “Larry is doing fine! But I had to call you because I’m in awe of his relationship with God. Is it true that he gets up during the night, and poof, the light goes on in the bathroom, and when he’s done, poof, the light goes off?”
    
    “Oh sweet Jesus”, exclaims Bonnie. “He’s peeing in the refrigerator again!”
    time: 2018-02-15, 10::30
    author:  madazzahatter
    score:  6532
    id: 7xpiv9
    ----------------
    My wife just called me and said, "Three of the girls in the office have just received some flowers for Valentines Day. They are absolutely gorgeous!" 
    I repied, "That's probably why they've received flowers then."
    time: 2018-02-14, 22::58
    author:  madazzahatter
    score:  16671
    id: 7xm4ar
    ----------------


From this, a few things are apparent: 
-- reddit no longer displays downvote counts, only upvote counts and upvote ratio
-- not all posts are jokes, some are mod posts
-- upvote ratio is very slow to get, for some reason. 

## Getting the jokes, get it? get it? 

Previously, I have tried to mine reddit from a large range of dates, or stream. One thing usually happens: notebook crashes, all data or lost, or both. 
So I am downloading jokes piece wise, month per month, from 2010. 

Second thing is that I'm only getting jokes with score>5, to weed out the really bad ones. 


```python
from datetime import datetime
years = range(2010, 2018)
months = range(1, 13)
timestamp_list = []
for y in years:
    for m in months: 
        timestamp_list.append(datetime(y, m, 1).timestamp())

timestamp_list.append(datetime(2018, 1, 1).timestamp())
timestamp_list.append(datetime(2018, 2, 1).timestamp())
for d in timestamp_list:
    value = datetime.fromtimestamp(d)
    #print(value.strftime('%Y-%m-%d'))
```

Finally, getting file --- I do have to say, this scraping took 3 hours, without getting the upvote_ratio. 
I'm not sure how long a complete data set will take. 
The resulting CSV file is about 50mb.


```python
start_timestamp = timestamp_list[0]
jokelist = []
counter = 0
import csv
with open('all_jokes_plus.csv', 'w') as csvfile:
    spamwriter = csv.writer(csvfile, quoting=csv.QUOTE_ALL, quotechar="|", delimiter=",")

    for i in range(1, len(timestamp_list)):
        end_timestamp = timestamp_list[i]
        s = datetime.fromtimestamp(start_timestamp)
        e = datetime.fromtimestamp(end_timestamp)
        print("getting top jokes from ", s.strftime('%Y-%m-%d'), " to ", e.strftime('%Y-%m-%d'))
        print(start_timestamp, end_timestamp)
        currJokes = jokesSub.submissions(start_timestamp, end_timestamp)
        for submission in currJokes:
            #print(submission.title, submission.author)
            if (submission.author in jokes_mods): 
                continue
            if (submission.score <5):
                continue
            q = submission.title
            a = submission.selftext
            if (len(q) == 0 or len(a) == 0):
                continue
            #jokelist = jokelist + [[submission.id,submission.created_utc, q, a]]
            spamwriter.writerow([submission.id, submission.score,q, a,
                submission.created_utc,submission.author.name, submission.ups, submission.upvote_ratio])
            counter = counter + 1
            if (counter %500 ==0):
                print("\t processed ", counter, " jokes")
        start_timestamp = end_timestamp
        #for j in jokelist: 
        #    spamwriter.writerow(j)
        #jokelist=[]
        print("processed ", counter, " jokes so far")
```

... sample output:....

getting top jokes from  2010-01-01  to  2010-02-01
1262322000.0 1265000400.0
processed  4  jokes so far
getting top jokes from  2010-02-01  to  2010-03-01
1265000400.0 1267419600.0
processed  9  jokes so far
getting top jokes from  2010-03-01  to  2010-04-01
1267419600.0 1270094400.0
processed  10  jokes so far
getting top jokes from  2010-04-01  to  2010-05-01
1270094400.0 1272686400.0
... ...
which goes on for hours. 