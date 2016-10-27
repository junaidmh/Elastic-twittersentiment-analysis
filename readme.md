## Twitter Sentiment - Python,Elasticsearch, Kibana

Twitter Sentiment Analysis - Using Python,Elasticsearch & Kibana



# Getting Started - Installation

We would have to install Elasticsearch & Kibana from the Elastic Stack for this applicatiokn. 

Python would be used for coding the application. I have used python 2.7, you can use 3.5 also but some code changes would have to be made.


Find below the link for installing Elasticsearch & Kibana. 

Install Elasticsearch :            [Elasticsearch](https://docs.docker.com/installation/) and 
Install Kibana :    [Kibana](https://docs.docker.com/compose/install/)

After installing these two applications we would now get stated by cloning this project onto your host machine. 


# Creating Twitter Dev account and obtaining the access tokens.

In order to get started be sure to clone this project onto your host machine. Create a directory on your host. Please note that this application would use the twitter dev API's for getting access to twitter feed. You need to create a twitter dev account and get the following information form there. 





git clone https://github.com/Jay-Wani/dockercompose-sampleapp.git



# Analyze the python code 

We will be using the below `twittersent.py` file to stream the tweets. 

```yaml
#Imporyt the various libraries, these would contain the modules that we would be using

import json
from tweepy.streaming import StreamListener
from tweepy import OAuthHandler
from tweepy import Stream
from textblob import TextBlob
from elasticsearch import Elasticsearch

# import twitter keys and tokens
from config import *

# create instance of elasticsearch
es = Elasticsearch()


class TweetStreamListener(StreamListener):

    # on success
    def on_data(self, data):

        # decode json
        dict_data = json.loads(data)

        # pass tweet into TextBlob
        tweet = TextBlob(dict_data["text"])

        # output sentiment polarity
        print tweet.sentiment.polarity

        # determine if sentiment is positive, negative, or neutral
        if tweet.sentiment.polarity < 0:
            sentiment = "negative"
        elif tweet.sentiment.polarity == 0:
            sentiment = "neutral"
        else:
            sentiment = "positive"

        # output sentiment
        print sentiment

        # add text and sentiment info to elasticsearch
        es.index(index="sentiment",
                 doc_type="test-type",
                 body={"author": dict_data["user"]["screen_name"],
                       "date": dict_data["created_at"],
                       "message": dict_data["text"],
                       "polarity": tweet.sentiment.polarity,
                       "subjectivity": tweet.sentiment.subjectivity,
                       "sentiment": sentiment})
        return True

    # on failure
    def on_error(self, status):
        print status

if __name__ == '__main__':

    # create instance of the tweepy tweet stream listener
    listener = TweetStreamListener()

    # set twitter keys/tokens
    auth = OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_token, access_token_secret)

    # create instance of the tweepy stream
    stream = Stream(auth, listener)

    # search twitter for "congress" keyword
    stream.filter(track=['cloud'])
```



Import Modules : We would be importing the following modules for this application
      1. Tweepy (for accessing twitter API for streaming tweets) 
      2. TextBlob (for doing sentiment analysis)
      3. Json (for loading data)
      4. elasticsearch api (for moving data to elasticserch)



# Run the python file
```yaml
docker-compose up -d
```

